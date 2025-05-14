# Terraform Vms Syntax

## Create a Vm with Cloud-init scripts for Initialization

Inside the module needs to have a cloud-init dir with a cloud-config yaml file in it:

```yaml
#cloud-config
packages:
  - nginx

write_files:
  - path: /etc/nginx/conf.d/default.conf
    content: |
      server {
          listen 80;
          server_name jenkins.azurewebsites.net;
          location / {
              proxy_pass http://${jenkins_ip}:8080;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
          }
      }
      server {
          listen 80;
          server_name nextcloud.azurewebsites.net;
          location / {
              proxy_pass http://${nextcloud_ip};
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
          }
      }

runcmd:
  - systemctl restart nginx
```

```hcl
resource "azurerm_linux_virtual_machine" "nginx" {
  name                = "nginx-vm"
  resource_group_name = var.resource_group_name
  location            = var.location
  size                = "Standard_B1s"
  admin_username      = "azureuser"
  network_interface_ids = [
    azurerm_network_interface.nginx.id
  ]
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
    name                 = "nginx-disk"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/azure_key.pub")
  }

  custom_data = base64encode(templatefile("${path.module}/cloud-init/nginx.yaml", {
    jenkins_ip   = "10.0.2.4"
    nextcloud_ip = "10.0.2.5"
  }))


  tags = {
    environment = "development"
  }
}
```