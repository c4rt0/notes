RHCSA preparation
-----------------

Practice question 1:
* You have been provided a virtual box names serverX.example.com (X is your domain number)
* password for virtual machine should be `redhat123`
* serverX.example.com provided with ip: 127.25.X.11/255.255.255.0
* serverX.example.com provided with gateway 172.25.254.254 & example.com dns domain with IP: 172.25.254.254

Solution:
a) Verify hostname to check domain number
`hostname`
b) Set network settings:
`nmcli dev status`,
`nmcli con add con-name <connection_name> ifname <same> type ethernet ip4 127.25.X.11/24 gw4 172.25.254.254` #Replace the X with number from hostname

`man nmcli-examples | less`
Seek for `nmcli con`
`nmcli con show`
`nmcli`

Alternatively use:
`nmtui`

