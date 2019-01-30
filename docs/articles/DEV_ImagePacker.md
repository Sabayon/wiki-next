= Building VMware images =

== Setting up a build environment using VMware Player ==

While we'll be installing the VMware Workstation in these instructions which is not free software, we only need the player and VIX components which are free to use so there's no need to purchase a workstation license.

* Install app-emulation/vmware-workstation with USE="ovftool server vix" (currently without these use flags in entropy, built in SCR temporarily)
* If using bash, `env-update && source /etc/profile`
* Setup virtual networks
** Copy the following into `/etc/vmware/networking`
<pre class="clear">
VERSION=1,0
answer VNET_1_DHCP yes
answer VNET_1_DHCP_CFG_HASH BA0A2ADB438F4A390ADE7333633E4AA8C00B897A
answer VNET_1_HOSTONLY_NETMASK 255.255.255.0
answer VNET_1_HOSTONLY_SUBNET 192.168.93.0
answer VNET_1_VIRTUAL_ADAPTER yes
answer VNET_8_DHCP yes
answer VNET_8_DHCP_CFG_HASH F80579270B94AB020609BA2500257CA9B04050DD
answer VNET_8_HOSTONLY_NETMASK 255.255.255.0
answer VNET_8_HOSTONLY_SUBNET 192.168.94.0
answer VNET_8_NAT yes
answer VNET_8_VIRTUAL_ADAPTER yes
</pre>
** run `vmware-netcfg`. You need at least vmnet8, type NAT, with local DHCP and Connect virtual host adapter options ticked. Save the changes and verify the network config has been written to `/etc/vmware/networking` and `/etc/vmware/vmnet8/`
* Get the vmware prerequisites running (kernel modules loaded, networks enabled) <br />`# systemctl enable --now vmware.target`
* Clone a copy of https://github.com/Sabayon/packer-templates

== Building the image ==

* Run the packer command to complete the install <br /> `packer build -var "flavor=SpinBase" -var "vagrant=vagrant" -only vmware-iso images.json`
* Packer produces a .tar.gz of a vmx directory. In order to import this into ESX, it needs to be converted into an OVA.
** Extract the tar into a temporary directory, and cd into it
** Repair the disk images, packer sometimes produces disks that need repair <br /> `vmware-vdiskmanager -R disk.vmdk`
** Run the conversion <br /> `/opt/vmware/lib/vmware-ovftool/ovftool ./Sabayon\ VMware\ SpinBase.vmx ../Sabayon_Linux_DAILY_amd64_SpinBase.ova`
