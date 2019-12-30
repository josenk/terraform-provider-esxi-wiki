# Title:  How to configure a static IP address using cloud-init.

# Description:
  * This is a reasonably straightforward method to set an IP address to a esxi guest using the cloud-init package.

# OS Support:
  * This example should support any Linux-type OS that supports cloud-init.

# Pros:
  * Reasonably straightforward to implement.
  * No third-party provisioner plugins are required.

# Cons:
  * Cloud-init need to be pre-installed on the guest
  * Only tried on Ubuntu 16.04 so far

# Details:
  * Build a guest/OVA manually or using preseed (see below) with cloud-init and VMware datasource installed
  * Terraform provisions the machine and sets guestinfo.metadata using a config file which is embedded as gzip+base64 content
  * On boot cloud-init reads the configuration from the guestinfo.metadata key and configures the network accordingly

# Building guest/OVA
  * Install basic OS manually or using preseed (I used the Ubuntu 16.04 mini.iso image)
  * Install open-vm-tools, cloud-init, python3-pip, curl
  * Install the VMware Guestinfo datasource (see below)
  * Remove all clauses from /etc/network/interfaces, except for the line "source /etc/network/interfaces.d/*"
  * Shut down the guest and extract as an OVA or leave for cloning

# Installing VMware Guestinfo datasource

The default install of cloud-init on Ubuntu 16.04 does not have the ability to read the guestinfo.metadata and guestinfo.userdata properties set by the hypervisor. VMware has a datasource which can be installed (Python's PIP required) with their install script:

```
curl -sSL https://raw.githubusercontent.com/vmware/cloud-init-vmware-guestinfo/master/install.sh | sh -
```

More information at https://github.com/vmware/cloud-init-vmware-guestinfo

# Further customisation

  * The provisioned machine may be further customised with guestinfo.userdata 
  * The Terraform template_cloudinit_config datasource can be used to render file/templates into a multi-part cloud-init userdata config
  * This can setup users, SSH keys, install packages, run scripts, etc.

See: https://www.terraform.io/docs/providers/template/d/cloudinit_config.html

# Example:
  * In your terraform directory, create a copy of the network configuration file.   Name it <guest_name>-metadata.cfg.  This example "vmstatic01-metadata.cfg"
```
network:
  version: 1
  config:
    - type: physical
      name: ens32
      subnets:
        - type: static
          address: 192.168.0.253/24
          gateway: 192.168.0.1
    - type: nameserver
      address:
        - 8.8.8.8
        - 1.1.1.1
      search:
        - example.org
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
  ovf_source    = "ubuntu-1604.ova"
  network_interfaces {
    virtual_network = "VM Network"
  }

  #########################################
  #  ESXI Guestinfo metadata
  #########################################
  guestinfo = {
    "metadata" = base64gzip(file("${var.guest_name}-metadata.cfg"))
    "metadata.encoding" = "gzip+base64"
  }

}
```
  * `$ terraform apply`
