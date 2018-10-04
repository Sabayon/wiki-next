On Sabayon systems the firewall is controlled by firewalld.

Starting or stopping the firewall service on a Sabayon can be done with these commands:

To stop the firewall service:
systemctl stop firewalld

To start the firewall service:
systemctl start firewalld

To system wide disable the firewall at system boot:
systemctl disable firewalld
