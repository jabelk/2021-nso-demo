## Setting up NSO 

ncs-setup --package nso/packages/neds/cisco-ios-cli-6.44 \
--package nso/packages/neds/cisco-nx-cli-5.15 \
--package nso/packages/neds/cisco-iosxr-cli-7.20 \
--package nso/packages/neds/cisco-asa-cli-6.8 \
--dest nso-instance

cd ~/nso-instance
ncs

(wait 60-90 seconds)


## Populating your NSO instance 

ncs_cli -C -u admin

conf
devices authgroups group labadmin
default-map remote-name cisco
default-map remote-password cisco
default-map remote-secondary-password cisco
commit
top


devices device edge-sw01
address 10.10.20.172
authgroup labadmin
device-type cli ned-id cisco-ios-cli-6.44
device-type cli protocol telnet
ssh host-key-verification none
trace raw
devices device core-rtr01
address   10.10.20.173
ssh host-key-verification none
authgroup labadmin
device-type cli ned-id cisco-iosxr-cli-7.20
device-type cli protocol telnet
state admin-state unlocked
trace raw
!
devices device core-rtr02
address   10.10.20.174
ssh host-key-verification none
authgroup labadmin
device-type cli ned-id cisco-iosxr-cli-7.20
device-type cli protocol telnet
state admin-state unlocked
trace raw
!
devices device dist-rtr01
address   10.10.20.175
ssh host-key-verification none
authgroup labadmin
device-type cli ned-id cisco-ios-cli-6.44
device-type cli protocol telnet
state admin-state unlocked
trace raw
!
devices device dist-rtr02
address   10.10.20.176
ssh host-key-verification none
authgroup labadmin
device-type cli ned-id cisco-ios-cli-6.44
device-type cli protocol telnet
state admin-state unlocked
trace raw
!
! removed nexus 
!
devices device edge-firewall01
address   10.10.20.171
ssh host-key-verification none
authgroup labadmin
device-type cli ned-id cisco-asa-cli-6.8
device-type cli protocol telnet
state admin-state unlocked
trace raw
!
devices device edge-sw01
address   10.10.20.172
ssh host-key-verification none
authgroup labadmin
device-type cli ned-id cisco-ios-cli-6.44
device-type cli protocol telnet
state admin-state unlocked
trace raw
!
devices device internet-rtr01
address   10.10.20.181
ssh host-key-verification none
authgroup labadmin
device-type cli ned-id cisco-ios-cli-6.44
device-type cli protocol telnet
state admin-state unlocked
trace raw
!
commit

end


devices connect
show devices list

show running-config devices device dist-rtr01 config

devices sync-from

show running-config devices device dist-rtr01 config

check trace - what other automation considers important, NSO considers a log file
/home/developer/nso-instance/logs/ned-cisco-ios-cli-6.44-internet-rtr01.trace


config
devices device-group IOS-DEVICES
device-name internet-rtr01
device-name dist-rtr01
device-name dist-rtr02
devices device-group XR-DEVICES
device-name core-rtr01
device-name core-rtr02
devices device-group ASA-DEVICES
device-name edge-firewall01
devices device-group ALL
device-group ASA-DEVICES
device-group IOS-DEVICES
device-group XR-DEVICES



devices device-group IOS-DEVICES check-sync 

exit

telnet 10.10.20.181

cisco/cisco

conf t
interface Loopback 100
description Out of Band Change for NSO to find
end
exit

ncs_cli -C -u admin
devices device-group IOS-DEVICES check-sync

## error 

devices device-group IOS-DEVICES sync-from dry-run

## find the error 

devices device-group IOS-DEVICES sync-from

show running-config devices device internet-rtr01 config interface

## see the new loopback 

## Exploring the network with the NSO CLI 

show running-config devices device internet-rtr01 config

## explain each step of the hierarchy 

show running-config devices device internet-rtr01 config interface
show running-config devices device internet-rtr01 config interface | display json
show running-config devices device internet-rtr01 config interface | display xml
 
## explain NED parsing, not using NETCONF 



## Updating device configuration 

switch cli
show configuration devices device internet-rtr01 config interface
switch cli



  interface GigabitEthernet3
   description to enp0s2.internet-host01
   no switchport
   negotiation auto
   no mop enabled
   no mop sysid
   ip address 172.31.0.1 255.255.255.0
   no shutdown


look at trace
/home/developer/nso-instance/logs/ned-cisco-ios-cli-6.44-internet-rtr01.trace
internet-rtr01(config-if)

config
devices device internet-rtr01 config ?
interface GigabitEthernet ?
interface GigabitEthernet 3
ip address ? 

## existing one is there 

ip address 172.64.0.1 ?

## existing mask is there 

ip address 172.64.0.1 255.255.255.0
exit
interface Loopback 1337
description "Placeholder Loopback"



exit
top
show configuration
show configuration | display xml
commit dry-run outformat native

## adds in the no shutdown for me 
commit

show configuration rollback changes

rollback configuration

commit dry-run outformat native
commit
end


## Templates and live status 

devices template SET-DNS-SERVER
! IOS TEMPLATE
ned-id cisco-ios-cli-6.44
config
ip name-server name-server-list 208.67.222.222
ip name-server name-server-list 208.67.220.220
exit
exit
exit

! ASA TEMPLATE
! 3 exits to return back to the 'config-template-SET-DNS-SERVER' context
ned-id cisco-asa-cli-6.8
config
dns domain-lookup mgmt
dns server-group DefaultDNS
name-server 208.67.220.220
name-server 208.67.222.222

! IOS-XR TEMPLATE
ned-id cisco-iosxr-cli-7.20
config
domain name-server 208.67.222.222
exit
domain name-server 208.67.220.220

commit
top

devices device-group ALL apply-template template-name SET-DNS-SERVER
commit dry-run outformat native
commit


devices template SET-DNS-SERVER
ned-id cisco-ios-cli-6.44
config
no ip name-server name-server-list 208.67.222.222
ip name-server name-server-list 208.67.222.111
top
commit dry-run
commit
devices device-group ALL apply-template template-name SET-DNS-SERVER
commit dry-run outformat native

## only new server is present, doesn't remove old server 

## take out config  templates 
rollback configuration 10019
admin@ncs(config)# show configuration
no devices template SET-DNS-SERVER
devices device core-rtr01
 config


ncs-make-package --service-skeleton template  dns-servers



devices device internet-rtr01 config
snmp-server community VARIABLE-TO-BE RO
commit dry-run outformat xml

devices device edge-firewall01 config
snmp-server community secret
commit dry-run
snmp-server community VARIABLE-GOES-HERE
snmp-server enable

devices device core-rtr01 config
snmp-server community VARIABLE-TO-BE RO


<snmp-server xmlns="urn:ios">
  <community>
    <name>VARIABLE-TO-BE</name>
    <RO/>
  </community>
</snmp-server>

<snmp-server xmlns="http://cisco.com/ned/asa">
  <community>
    <secret>VARIABLE-GOES-HERE</secret>
  </community>
  <enable-conf>
    <enable>true</enable>
  </enable-conf>
</snmp-server>


<snmp-server xmlns="http://tail-f.com/ned/cisco-ios-xr">
  <community>
    <name>VARIABLE-TO-BE</name>
    <RW/>
  </community>
</snmp-server>


packages reload


dns-servers 1st-instance device core-rtr01
top
dns-servers 2nd-instance device core-rtr02
top
dns-servers 3rd-instance device dist-rtr01
top
dns-servers 4th-instance device edge-firewall01
top
dns-servers 5th-instance device edge-sw01
top
dns-servers 6th-instance device internet-rtr01
commit dry-run
commit dry-run outformat native
top


change template to have yang
make
packages reload

dns-servers 1st-instance device core-rtr01 community-string MY-NFD-COMM-STRING
top
dns-servers 2nd-instance device core-rtr02 community-string MY-NFD-COMM-STRING
top
dns-servers 3rd-instance device dist-rtr01 community-string MY-NFD-COMM-STRING
top
dns-servers 4th-instance device edge-firewall01 community-string MY-NFD-COMM-STRING
top
dns-servers 5th-instance device edge-sw01 community-string MY-NFD-COMM-STRING
top
dns-servers 6th-instance device internet-rtr01 community-string MY-NFD-COMM-STRING

## where are the variables stored? CDB 

show running-config dns-servers





## operational state 

show devices device internet-rtr01 platform
show devices device internet-rtr01 platform serial-number

show devices device * platform

Note: In this case the actual serial number for the IOS-XR platform is N/A


show devices device * platform version

show devices device internet-rtr01 live-status
show devices device dist-rtr01 live-status | display json

devices device internet-rtr01 live-status exec show ip route
devices device internet-rtr01 live-status exec show ip int br
devices device internet-rtr01 live-status exec any dir
devices device internet-rtr01 live-status exec any show ip int br

devices device dist* live-status exec show ip int br

devices device dist* live-status exec show ip int br | save /home/developer/sh-ip-int-br.txt






