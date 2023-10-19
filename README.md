# HSRP

HSRP is a Cisco proprietary protocol that was developed to allow several multilayer switches or routers to appear as a single gateway IP address.

HSRP using Active and Standby term.

## HSRP Configuration

![Creating a VLAN](https://raw.githubusercontent.com/deliawolf/HSRP/main/HSRP.drawio.png)

note* : network link that provide redundancy need to have same ip subnet.
note* : virtual ip need to have same ip subnet.
note* : link between devices that form HSRP not in mandatory rule to have link to each other, but it is considered best practice to have it.
note* : there is EIGRP configured in this lab, the config will not be provided.

1. Server Configuration
```
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0
sudo route add -net 0.0.0.0 gw 192.168.1.1
sudo /etc/init.d/services/dhcp stop
```
2. Switch COnfiguration
note* : not mandatory
```
ip default-gateway 192.168.1.1
```
3. R1 Configuration
```
!
interface GigabitEthernet0/0
 ip address 192.168.1.2 255.255.255.0
 standby 1 ip 192.168.1.1
 standby 1 priority 200
 standby 1 preempt
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.3.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 10.10.1.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
```
only HSRP Configuration
```
interface GigabitEthernet0/0
 ip address 192.168.1.2 255.255.255.0
 standby 1 ip 192.168.1.1
 standby 1 priority 200
 standby 1 preempt
```
4. R2 Configuration
```
!
interface GigabitEthernet0/0
 ip address 192.168.1.3 255.255.255.0
 standby 1 ip 192.168.1.1
 standby 1 preempt
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.3.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 10.10.2.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
```
only HSRP Configuration
```
interface GigabitEthernet0/0
 ip address 192.168.1.3 255.255.255.0
 standby 1 ip 192.168.1.1
 standby 1 preempt
```
5. R-Core
```
!
interface GigabitEthernet0/0
 ip address 10.10.1.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.2.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
```
### HSRP Verification
```
R1#show standby 
GigabitEthernet0/0 - Group 1
  State is Active
    2 state changes, last state change 04:50:43
  Virtual IP address is 192.168.1.1
  Active virtual MAC address is 0000.0c07.ac01
    Local virtual MAC address is 0000.0c07.ac01 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.056 secs
  Preemption enabled
  Active router is local
  Standby router is 192.168.1.3, priority 100 (expires in 8.368 sec)
  Priority 200 (configured 200)
  Group name is "hsrp-Gi0/0-1" (default)

R1#show standby brief 
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Gi0/0       1    200 P Active  local           192.168.1.3     192.168.1.1

R2#show standby 
GigabitEthernet0/0 - Group 1
  State is Standby
    1 state change, last state change 00:04:48
  Virtual IP address is 192.168.1.1
  Active virtual MAC address is 0000.0c07.ac01
    Local virtual MAC address is 0000.0c07.ac01 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.696 secs
  Preemption enabled
  Active router is 192.168.1.2, priority 200 (expires in 9.664 sec)
  Standby router is local
  Priority 100 (default 100)
  Group name is "hsrp-Gi0/0-1" (default)

R2#show standby 
GigabitEthernet0/0 - Group 1
  State is Standby
    1 state change, last state change 04:50:47
  Virtual IP address is 192.168.1.1
  Active virtual MAC address is 0000.0c07.ac01
    Local virtual MAC address is 0000.0c07.ac01 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 2.432 secs
  Preemption enabled
  Active router is 192.168.1.2, priority 200 (expires in 9.776 sec)
  Standby router is local
  Priority 100 (default 100)
  Group name is "hsrp-Gi0/0-1" (default)

R2#show standby brief 
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Gi0/0       1    100 P Standby 192.168.1.2     local           192.168.1.1

```
Fot Testing we can ping the virtual ip from end user:
1. Ping the Virtual IP
2. shutdown the link / the devices(not recomended)
3. observe the ping also observe the commands in R2 router "show standby" "show standby brief"

#### Additional Configuration
Priority contribute to router decision to elect Active and Standby Routers.
more heigher priority the more likely it becomes Active router.
```
R2(config)#interface ethernet 0/1
R2(config-if)#standby 1 priority 110
```
preempt allow Down router to regain the Active status after it goes up from down.
for example R1 is Active and R2 is standby, R1 is down so R2 got Active status. without preempt when R1 become up again is not regaining the Active status but it become stanby with preempt it automaticly regaining the Active status and R2 become stanby again.
```
R2(config-if)#standby 1 preempt
```
On both R1 and R2, configure the Ethernet 0/1 HSRP group 1 hello timer for 200 milliseconds, a hold time of 750 milliseconds, and a pre-empt delay minimum of 300 seconds.
```
R1(config-if)#standby 1 timers msec 200 msec 750
R1(config-if)#standby 1 preempt delay minimum 300
```
security authentication
```
R1(config)# key chain MY_KEYCHAIN
R1(config-keychain)# key 1
R1(config-keychain-key)# key-string 7 MY_SECRET_PASSWORD
R1(config-keychain-key)# exit
R1(config)# interface GigabitEthernet0/0
R1(config-if)# standby 1 authentication md5 key-chain MY_KEYCHAIN
```
change to HSRP v2
```
R1(config-if)# standby 10 version 2
```

#### Additional Configuration not lab yet.

Object Tracking

HSRP offers a feature called interface tracking. We can select an interface to track and if it fails we will give it a penalty. This way your priority will decrease and another device can become the active router.

preemption is mandatory

the interface we use is is not downlink interface but uplink interface. it is used so the HSRP know if the uplink interface up or down.

let me explain, R1 have downlink to Server 1 and Uplink to Border-Router. HSRP is configured in downlink and if downlink interface down standby will take over and become active. this config is implemented in Uplink interface so HSRP will know if the uplink down or not.
```
R1(config)track 1 interface GigabitEthernet 0/2 line-protocol
```
We can choose to decrement the priority or you can decide to shut the entire HSRP group in case the interface is down. Let’s try decrementing the priority:
```
R1(config)# interface GigabitEthernet0/0
R1(config-if)#standby 1 track 1 decrement 60
```
that mean when something is triggered the priority 150 will decreased by 60 becoming 90.

#### other additioanl config no lab yet using IP SLA

Interface tracking is useful but it will only check the state of the interface. It’s possible that the interface remains in the up state but that we are unable to reach outside network. It might be a better idea to use IP SLA instead since it can check end-to-end connectivity.
```
R1(config)#ip sla 1
R1(config-ip-sla)#icmp-echo 192.168.23.3
R1(config-ip-sla-echo)#frequency 10

R1(config)#ip sla schedule 1 start-time now life forever
```
We can now combine IP SLA with object tracking:
```
R1(config)#no track 1 
R1(config)#track 1 ip sla 1
```


references : https://content.cisco.com/chapter.sjs?uri=/searchable/chapter/content/en/us/td/docs/ios-xml/ios/ipapp_fhrp/configuration/xe-16/fhp-xe-16-book/fhp-hsrp.html.xml
