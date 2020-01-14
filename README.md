# ansible-viptela

An Ansible Role for automating a Viptela iniital wokr.

This role can perform the following functions:
- Configure the Vipltela device
- Set CA root-chain for signature 
- Install certificated key 
- Add vBond with GUI
- Reboot the Viptela device to  makesure the certificated key is ok  

###
inventory file is used to define the device that you would like to build the environment.
the content include:
- CA_host
- vManage
- vBond 
- vSmart
- ZTP
- vEdge
- Vipltela 
the user need to makesure the host-name and management IP of the device in here
```
[CA_host]
CA ansible_host=10.124.112.56

[CA_host:vars]
ansible_ssh_user=root
ansible_ssh_pass=admin

[vManage_DC1]
SGCORE_vManage_01 ansible_host=10.124.112.110

[vManage_DC2]

[vManage:children]
vManage_DC1
vManage_DC2

[vBond]
SGCORE_vBond_01 ansible_host=10.124.112.214

[vSmart]
SGCORE_vSmart_01 ansible_host=10.124.112.215

[ZTP]
SGCORE_ZTP_SERVER_01 ansible_host=10.124.112.111

[vEdge_DC]
SGCORE_DC_SITE_01 ansible_host=10.124.112.10
SGCORE_DC_SITE_02 ansible_host=10.124.112.40

[vEdge_01]

[vEdge_02]

[vEdge_03]

[vEdge_04]

[vEdge:children]
vEdge_DC
vEdge_01
vEdge_02
vEdge_03
vEdge_04

[Viptela:children]
vManage
vBond
vSmart
ZTP
vEdge

[Viptela:vars]
ansible_ssh_user=admin
ansible_ssh_pass=admin

```


### Set all the default paramter 

group_vars/all.yml
```
FIRST_LOGIN: True     # When FIRST_LOGIN is True, finishing all the tasks, the Vipleta devie will reboot ,then the flag will change to False 
CLIENT_CSR_DIR: '/CA/client/csr'         #viptela csr_key will store in the directory in CA server
CLIENT_SIGNED_DIR: '/CA/client/signed'   #viptela certificated key  will store in the directory in CA server
```

### Set Viptela Setting
group_vars/Viptela.yml for the common setting 
```
ansible_ssh_user: admin
ansible_ssh_pass: admin
ansible_network_os: ios
ansible_connection: network_cli
ansible_user: admin    #cli function username
ansible_password: admin #cli function password
SP_ORGANIZATION_NAME: "Cisco Sy1 - 19968"
ORGANIZATION_NAME: "Cisco Sy1 - 19968"
CLOCK_TIMEZONE: "Asia/Shanghai"
VBOND_ADDRESS: "dc2-vbond1.com"
VBOND_PORT: 12346
NTP_SERVER: 10.0.4.1
DNS_SERVER: 10.0.4.1
CA_SERVER: 10.124.112.56
CA_SERVER_PASSWORD: 'admin'

  
```

#### Set vManage Setting:
group_vars/vManage_DC1.yml  for the DC1_vManage common setting
```
VBOND_FLAG: False  #all of the device except vbond require setting False 
SITE_ID: 1
```

### Set particular device Setting
host_vars/Test_vManage_02.yml #every device need a single paramter ,please makesure the file name is the same as your device name 
makesure the management interface is eth0, because ansible will get the IP and host from the variable
```
HOST_NAME: "Test_vManage_02"
SYSTEM_IP: "10.255.255.2"
VPN0_INTERFACES:
  eth1:
    TUNNEL_FLAG: True            #if you want to start tunnel function,set the tunnel_flag is True 
    IPSEC: False                 #vBond && vEdge need set the flag is True 
	IP_ADDR: 10.1.1.1
    PREFIX: 24
    PORT_STATE: "no shutdown"
    COLOR: "mpls"
    SERVICE: "all dhcp dns icmp sshd netconf ntp https"  #set thetunnel interface allow  service 
    NEXT_HOP: 10.1.1.254         #default route next_hop setting 
  eth2:
    TUNNEL_FLAG: False
    IP_ADDR: 10.1.2.1
    PREFIX: 24
    PORT_STATE: "no shutdown"
    NEXT_HOP: 10.1.2.254
VPN512_INTERFACES:
  eth0:                        # you must confirm that the management interface is the same as eth0 for vpn 512
    IP_ADDR: 10.124.112.119
    PREFIX: 24
    PORT_STATE: "no shutdown"
    NEXT_HOP: 10.124.112.254

```

#### CA paramter setting:
group_vars/CA_host.yml
```
A_PRIVATE_KEY_DIR: '/CA/private_key' 
CA_CSR_DIR: "/CA/csr"
CA_PUBLIC_KEY_DIR: '/CA/public_key'
#CLIENT_CSR_DIR: '/CA/client/csr'
#CLIENT_SIGNED_DIR: '/CA/client/signed'
COUNTRY_NAME: "AU"                      
STATE_NAME: "AU"
LOCALITY_NAME: "Perth"
ORGANIZATION_NAME: "BHP"
ORGANIZATION_UNIT_NAME: "BHP"
COMMON_NAME: "XCA"
EMAIL_ADDRESS: "admin@bhp.com"

```

#### Running:
step:
1.  cd Vipltela
2.  ansible-playbook site.yml -i inventory

