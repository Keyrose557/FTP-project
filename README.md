
# FTP Server setup

## 1. Laptop IP settings

- [ ] Windows laptop ethernet adapter settings should match these below

`ip addr:10.1.1.1`
this is the laptop acting as the FTP server

`subnet: 255.255.255.0`


### 2. TFTPD app Config

- [ ] Open tftpd app
- [ ] select Browse directory
- [ ] select the folder that contains the CISCO IOS files



## 3. CISCO Switch config
This is for the switch your using for to daisy chain the CISCO devices.

!!! Note: Do NOT forget to upgrade this switch Last.

Credentials
user: `admin`
password `$up3r$ecr3t!`

```bash

enable # you should see a `#` in the command line
configure terminal

# Assign IP address to VLAN 10 interface

interface vlan 10
 ip address 10.1.1.2 255.255.255.0
 no shutdown
exit

# Assign interfaces to VLAN 10

interface range fastethernet 1/1-8
 switchport mode access
 switchport access vlan 10
exit

# Powering off will revert settings unless you write to memory

```

## 4. Routers / Switchs receiving the update

**Interface number will be dependent on Router or Switch**

interface means Ethernet port

```bash

en # only if you dont see a `#` in the command line
configure terminal
interface fastethernet 0/1 # router and switch interface number is different
 ip address 10.1.1.3 255.255.255.0
 no shutdown
exit

```
Us the IP scheme as follows for the remaing devices 

10.1.1.1 for laptop

10.1.1.2 for switch supporting LAN

10.1.1.3 for Cisco device

10.1.1.4 for Cisco device

10.1.1.5 for Cisco device

10.1.1.6 for Cisco device

10.1.1.7 for Cisco device

10.1.1.8 for Cisco device

Before moving on configure all these ip address for the remainder of the devices using the commands above changing the IP address go in order, take note of the switch and routers interface number they are different. 

- [ ] fe 0/1 for router
- [ ] fe 1/1 for switch

fe = Fastethernet

## 5. Verify LAN Connectivity for all devices 

```bash

# 10.1.1.1 should be able to ping all these ip address if you are successful

ping 10.1.1.2
ping 10.1.1.3
ping 10.1.1.4
ping 10.1.1.5
ping 10.1.1.6
ping 10.1.1.7
ping 10.1.1.8

# If you have issues your recheck your configuration for typos
# using command ' show ip int br ' in CISCO CLI will show you your ip addr for the cisco device

```

## 6. Initiating tftp connection from CISCO CLI 

```bash
en
copy tftp flash

# address to ftp server 10.1.1.1

10.1.1.1

# source file name dependent on CISCO device one or the other not both !!!
    #Router ios
c-enterprise.987-9.Z19.bin
    #Switch ios
c-universal.198-9.Z9.bin

#Destination Filename, leave as default press Enter

# should be the new versions as shown above
```

- [ ] Initiate the download on one and than repeat by disconnecting from the console port to the next device.

- [ ] Once all devices have download initiated go back to the first device to continue with the next code block


## TFTP Solutions (Skip if no issues)

!!! Note: Read Before actioning and apply only as needed (Try Simple first)

### 1.TFTP interface source
- [ ] Check for typos
- [ ] Use if you cant access TFTP

in case you get TFTP issue when loading use the commands below but check for typos


```bash

conf t
ip tftp source-interface fastEthernet 0/1

# use the one below

no tftp source-interface # use this

```

### 2. Storage Space

Do you have enough space for the download on your CISCO device

!!! Notes: When deleting old files SPECIFY the new BOOT file. Missing this will brick the Device (Kinda)

## 7. Select NEW IOS before deletion of the old

!!! note: if UNSURE request support user error could brick pacstar

### Skip this code block

```bash

conf t
no boot system
# (removes old boot settings)
boot system flash:your_new_image.bin

# Router ios c.bin 
#or 
#Switch ios c.bin
# Dependent on CISCO Device

end
write memory


reload

show version
```
## Use this

```bash

en
conf t
boot system flash:your_new_image.bin

end
write memory

reload

show version
# look for the version to match the new file 
```

## 8. Only when you have verified IOS is uploaded, if unsure get me

```bash

en
dir
delete flash #(Old IOS file)
dir

# verify the old file is not there and the new version boots up correctly

```

Your done if you have applied this update to all routers and switches. Yay!




### Ignore this note (For backups)  
```bash

copy startup-config tftp:

10.1.1.1

Router-confg router.cfg


```

```bash

copy startup-config flash:backup.cfg 

dir flash:

copy flash:backup.cfg tftp:

10.1.1.1

rename



```

## CISCO Router Recovery-Rommon

### Boot into Rommon

- Open and Launch Putty Prior to router boot up

- During the IOS image decompression process. Press the mode button (power button) ONE time, this will reboot. followed by sending the Break command (special command) using putty

- This will interupt the boot process and talk you to Rommon mode

### Rommon Commands

Using the command Below will tell the router to ignore the startup-config at next boot.

```bash

confreg 0x2142

```
If done correctly reboot into the router and make your changes.

Once recovery steps are made enable using the following command

```bash
en

conf t

config-register 0x2102

write memory

# or

copy running-config startup-config

reload

```

## CISCO Switch Recovery-Bootloader

### Booting into Switch bootloader

- reboot switch, During boot press ONCE the mode button (Power button)
    - Must be done with in 5 seconds of boot process

use the commands below to rename the configurations files

```bash

dir flash:

rename flash:YourFile.txt flash:new_name

```

## Python Http method - alternative process (possible scripting?)

```bash

python -m http.server 8000

```

cisco command

```bash

copy http://10.1.1.1:8000/PATH/WAY/TO/file.bin flash:

```

## Upgrade vis Rommon (future project?)
