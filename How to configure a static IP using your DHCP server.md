# Title:  How to configure a static IP address using your DHCP server.

# Description:
  * This is a simple method to set an IP address to a esxi guest using your DHCP server.  Most DHCP servers, including DHCP servers that are built-in to routers (Linksys, Dlink, Buffalo, OpenWRT) all support a fixed IP mapped to a MAC address.


# OS Support:
  * This method will support any OS that supports DHCP, probably close to 100% coverage.  Linux, Windows, Unix, Container OS's etc...

# Pros:
  * Very simple to implement.
  * Compatible to just about every recent OS.
  * The terraform state does NOT need to be refreshed.  When the OS boots, it will have your fixed IP.
  * No special software or scripts is required on the guest.   (no chef, ansible, scripts...)
  * No provisioner plugins are required.
  * Most DHCP servers will also configure additional network settings.  Like DNS servers, domain name, hostname...

# Cons:
  * Not really considered a "static IP".   It's more of a guaranteed IP when using DHCP.
  * Very slim chance that you may "loose" the fixed IP.  For example, you lost your DHCP server/configuration.


# Details:
  * Select/use a unique MAC address for your guest.  You cannot choose a random MAC!  It must be valid.
    * https://docs.vmware.com/en/VMware-vSphere/5.5/com.vmware.vsphere.networking.doc/GUID-ADFECCE5-19E7-4A81-B706-171E279ACBCD.html
  * On your DHCP server, map the MAC address to an IP address you assign.
  * Configure a static MAC address in the esxi provider.
    * For example:  (replace values required for your environment.)
```
network_interfaces {
  virtual_network = "VM Network"
  mac_address     = "00:50:56:a1:b1:c2"
  nic_type        = "e1000"
}
```
  * Run `terraform apply`
