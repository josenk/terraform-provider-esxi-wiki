# Title:  How to configure a static IP address using remote-exec.

# Description:
  * This is a simple method to set an IP address to a esxi guest using the built-in remote-exec provisioner.

# OS Support:
  * This example will support any Linux-type OS that supports ssh.  It can probably be adapted to work with Windows also.

# Pros:
  * Very simple to implement.
  * Very high compatibility.
  * No special software or scripts is required on the guest.   (no chef, ansible, scripts...)
  * No third-party provisioner plugins are required.

# Cons:
  * The terraform state requires a refresh.

# Details:
  * Terraform uploads a stored copy of the network configuration file to the guest, then the guest is rebooted for those changes to take effect.
  * The guest will take the new IP address on reboot, however the terraform state is out-of-date.   You need to run `terraform refresh` after the guest has rebooted.

# Example:
  * The following example will work on RHEL-type OS's. (Centos, Fedora, etc...).  Modify path and file contents to fit your Linux distribution and requirements.
  *  In your terraform directory, create a copy of the network configuration file.   Name it <guest_name>-ifcfg-eth0.cfg.  This example" vmstatic01-ifcfg-eth0.cfg
```
BOOTPROTO=none
DEVICE=eth0
ONBOOT=yes
PREFIX=24
IPADDR=192.168.1.253
```
  * variables.tf
```
variable "esxi_hostname"  { default = "esxi" }
variable "esxi_hostport"  { default = "22" }
variable "esxi_username"  { default = "root" }
variable "esxi_password"  { } # Unspecified will prompt
variable "disk_store"     { default = "DataStore01" }
variable "guest_name"     { default = "vmstatic01" }
variable "guest_password" { default = "MyGuestPW" }

```
  * main.tf
```
#########################################
#  Provider configuration
#########################################
provider "esxi" {
  esxi_hostname = var.esxi_hostname
  esxi_hostport = var.esxi_hostport
  esxi_username = var.esxi_username
  esxi_password = var.esxi_password
}

#########################################
#  ESXI Guest resource
#########################################
resource "esxi_guest" "default" {
  guest_name    = var.guest_name
  disk_store    = var.disk_store
  clone_from_vm = "centos7"
  network_interfaces {
    virtual_network = "VM Network"
  }

  #########################################
  #  Provisioners to configure static IP
  #########################################
  provisioner "file" {
    source      = "${var.guest_name}-ifcfg-eth0.cfg"
    destination = "/etc/sysconfig/network-scripts/ifcfg-eth0"
    connection {
      type     = "ssh"
      user     = "root"
      password = var.guest_password
      host     = esxi_guest.default.ip_address
    }
  }

  provisioner "remote-exec" {
    inline = [
      "sleep 5",
      "shutdown -r 5"
    ]
  }
  connection {
    type     = "ssh"
    user     = "root"
    password = var.guest_password
    host     = esxi_guest.default.ip_address
  }
}
```
  * `$ terraform apply`
  * `$ sleep 120 ; terraform refresh`
