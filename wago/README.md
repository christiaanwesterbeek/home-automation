## WAGO 750-8203

### Doel

Het doel is niet om in de PLC te programmeren, maar de DI porten uitlezen en DO porten aansturen via Modbus TCP. Dat uitlezen aansturen wil ik met een Raspberry PI doen waar ik de logica in programmeer met node-red software. In dat geval is het niet nodig om Codesys te gebruiken (in andere worden de Codesys runtime te draaien op de PLC).

Het is mogelijk om DOCKER containers op de WAGO PFC te draaien waaronder ook een MODBUS container. Hoe je dit kunt doen, wordt in de onderstaande Youtube film uitgelegd. [WAGO PFC200 G2 Modbus Slave KBUS Docker Container](https://www.youtube.com/watch?v=TCV_BqcwM30). Maar dit lijkt niet mogelijk zou later blijken, omdat tijdens het uploaden van de docker.ipk file een error optreedt.

### Installatie

Dit stappenplan gaat uit van een uitgevoerde factory reset middels de volgende handeling beschreven in de [handleiding](https://asset.conrad.com/media10/add/160267/c1/-/en/001369150ML02/mode-demploi-1369150-api-controleur-wago-pfc200-2eth-can-750-8203-24-vdc-1-pcs.pdf).

> Set the mode selector switch to “RESET” and press the Reset button (RST) for 1 … 8 seconds. Briefly release the Reset button (RST) (< 1 second) and press it again until the “CAN” LED lights up red. If the “CAN” LED lights up red, release the mode selector switch and the Reset button. After the first 1 … 8 seconds, the controller reboots (all LEDs light up orange) and after another 3 seconds, the “Factory reset” process begins. The process is indicated by all LEDs lighting up red in succession.

Als dat goed is gegaan, branden alle 12 leds (SYS, RUN, I/O, etc) ononderbroken rood. Daarna zet je de PLC aan en uit. Dan zijn de SYS en I/O led's ononderbroken groen, en de RUN led knippert groen, en de andere leds zijn uit. Als de RUN-led groen knippert, betekent dit dat het PLC-programma in een debug is. Dit is de uitgangssituatie voor de verdere installatie.

#### Firmware en configuratie

Hierbij is de download link van de juiste Firmware. Voor de WAGO 750-8203 heb je de 8x0x variant nodig.

https://wago.sharepoint.com/:u:/s/S_SupportCenterDownloads/EeDbOLXUsSNMnqZ1j8TRixMBjAxXk35erjkR4I2rLKXqPw?e=pJUJj7

1. Schrijf image op de SD kaart met "Win32DiskImager" (Windows) of "balenaEtcher" (Mac)
2. De PLC is nog uit, steek nu de WAGO SD kaart erin, en zet de stroom op de PLC aan.
3. Wacht tot het booten van de PLC klaar is. SYS, RUN, I/O branden nu groen, rood, groen. Door de DHCP instelling heeft de PLC nu een IP-adres gekregen.
4. Middels het volgende commando, kom je erachter wat het IP-adres is geworden:

```
pi@raspberrypi:~ $ sudo arp-scan --interface=eth0 --localnet
Interface: eth0, type: EN10MB, MAC: e4:5f:01:82:d6:31, IPv4: 192.168.1.2
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.1.1	24:5a:4c:7a:d1:8c	(Unknown)
192.168.1.10	42:4c:99:c4:fa:96	(Unknown: locally administered)
192.168.1.40	00:30:de:44:19:4c	WAGO Kontakttechnik GmbH
```

5. Ga naar https://192.168.1.40/ in de browser en klik op negeren bij het ssl-certifaact-waarschuwing. Noteer nog even de hostname en description die je ziet staan in het inlogvenster. In dit geval zijn die:

Hostname:	PFC200-44194C
Description:	WAGO 750-8203 PFC200 2ETH CAN

6. Log in met het fabriekgebruikersnaam en wachtwoord admin/wago en wijzig het wachtwoord naar prototype.
7. Stel het statische IP-adres 192.168.1.17 in met "Wago Ethernet Settings".

Login|Verander ethernet instellingen
---|---
![image](https://user-images.githubusercontent.com/465989/157834401-8b54a2fa-387c-40e1-9b62-847f686c2c19.png)|![image](https://user-images.githubusercontent.com/465989/157834753-f473c3f3-455e-450a-a5e2-f24da2e1459b.png)

8. Klik op submit en open daarna in de browser https://192.168.1.17/ en login met admin/prototype
9. Nu kun je de bootable image op de SD card kopieren naar het interne geheugen (flash) van de PLC. Ga daarvoor naar wbm / administration / Create Image. Bij "Create bootable image from active partition (SD)".

Klik op "Start Copy"|aan het kopieeren
---|---
![image](https://user-images.githubusercontent.com/465989/157837772-341c2a15-1d3c-4888-88ea-d428d3e2ee55.png)|![image](https://user-images.githubusercontent.com/465989/157838123-b8e26877-6019-412b-899e-223e3b910261.png)

10. Schakel de voeding uit om de controller uit te schakelen.
11. Verwijder de SD-kaart
12. En schakel de voeding aan. Nu boot de PLC op zijn eigen flash geheugen en is na een minuut bereikbaar op 192.168.1.17.

Overigens branden de SYS, RUN, I/O LEDs nog steeds groen, rood, groen. Dat is normaal. Er draait nog geen programma, althans zo begrijp ik dat nu.

13. Probeer eens via ssh in te loggen op de PLC met root/wago en stel ook hier het nieuwe wachwoord prototype in.

```
pi@raspberrypi:~ $ ssh root@192.168.1.17
root@192.168.1.17's password: 


WAGO Linux Terminal on PFC200-44194C.


Security message: please change your password!

New password: 
Retype new password: 
passwd: password updated successfully
root@PFC200-44194C:~ exit
logout
Connection to 192.168.1.17 closed.
pi@raspberrypi:~ $ 
```

#### Modbus activeren

Sinds de factory reset en de installatie staat de switch op de PLC nog op de STOP stand en draait dus geen programma.

1. Verander de runtime naar "none"

Standaard draait CODESYS 2|Dit passen we aan naar none (en submit)
---|---
![image](https://user-images.githubusercontent.com/465989/157842285-103621d1-7083-4820-b3fe-8a830478e7c5.png)|![image](https://user-images.githubusercontent.com/465989/157844007-ccdfe86e-83d8-44e8-9541-cc9eb4ab7065.png)

De SYS, RUN, I/O LEDs branden nu groen, knipperend groen, groen

We gaan nu de KbusModbusPFCSlave installeren en draaien en zorgt voor directe toegang tot KBus-IO-Modules voor Node.js/node-red door modbus requests te sturen. De downloads en beschrijving van KbusModbusPFCSlave staan [hier](https://github.com/WAGO/pfc-howtos/tree/master/HowTo_AddKbusModbusSlave), maar schrijf ik alsnog hier in stappen uit.

2. Download de gehele pfc howto repo en pak deze uit.

Downloaden|Uitpakken
---|---
![image](https://user-images.githubusercontent.com/465989/157847774-017f8f3a-2d8a-474d-9887-338c450d5b22.png)|![image](https://user-images.githubusercontent.com/465989/157848141-41a48beb-f547-43f2-a459-ae7fb283dbc9.png)

3. Gebruik wbm en ga naar Configuration > Software Uploads en selecteer het bestand /home/pi/Downloads/pfc-howtos-master/HowTo_AddKbusModbusSlave/packages/kbusmodbusslave_1.5.1_armhf.ipk

ipk file installeren|uploading...
---|---
![image](https://user-images.githubusercontent.com/465989/157848678-06366ff1-2023-4c1c-b80b-a2d8bcf77931.png)|![image](https://user-images.githubusercontent.com/465989/157848752-992e526e-9e8b-4855-8901-06ea23b55cb6.png)

Na een paar seconden zie je het bericht dat het installeren is gelukt.

Automatisch starten van modbus activeren.

4. Copy the start-up script to PFC via ssh.
```
pi@raspberrypi:~ $ scp /home/pi/Downloads/pfc-howtos-master/HowTo_AddKbusModbusSlave/pfc/etc_init.d/kbusmodbusslave root@192.168.1.17:/etc/init.d/kbusmodbusslave
root@192.168.1.17's password: 
kbusmodbusslave                                                                               100% 1606   397.0KB/s   00:00    
```

You need to enter the root password

Create symbolic link in /etc/rc.d/ for start-up kbusmodbusslave on power-on.
Open a terminal session to PFC

```
pi@raspberrypi:~ $ ssh root@192.168.1.17
root@192.168.1.17's password: 


WAGO Linux Terminal on PFC200-44194C.

```
You need to enter the root password

Now on PFC side, create the sym-link

```
root@PFC200-44194C:~ ln -s /etc/init.d/kbusmodbusslave /etc/rc.d/S98_kbusmodbusslave
```

Make /etc/init.d/kbusmodbusslave executable
```
root@PFC200-44194C:~ chmod a+x /etc/init.d/kbusmodbusslave
```

Check for any ADI/DAL-Application(like PLC-Runtime) to be not started during start-up
Check for link to start up PLC-Runtime
If link S98_runtime is present delete it

```
root@PFC200-44194C:~ cd /etc/rc.d/
root@PFC200-44194C:/etc/rc.d ll
drwxr-xr-x    3 root     root          4032 Mar 11 11:27 .
drwxr-xr-x   38 root     root          7176 Mar 11 11:23 ..
drwxr-xr-x    2 root     root           312 Dec 14 17:08 disabled
lrwxrwxrwx    1 root     root            18 Dec 14 17:03 S00_boot_wdg -> ../init.d/boot_wdg
lrwxrwxrwx    1 root     root            23 Dec 14 16:09 S00_cgroupsconfig -> ../init.d/cgroupsconfig
lrwxrwxrwx    1 root     root            19 Dec 14 17:04 S00_check_rtc -> ../init.d/check_rtc
lrwxrwxrwx    1 root     root            14 Dec 14 16:02 S00_udev -> ../init.d/udev
lrwxrwxrwx    1 root     root            22 Dec 14 17:03 S01_device-setup -> ../init.d/device-setup
lrwxrwxrwx    1 root     root            18 Dec 14 16:03 S01_firewall -> ../init.d/firewall
lrwxrwxrwx    1 root     root            22 Dec 14 17:04 S01_link_devices -> ../init.d/link_devices
lrwxrwxrwx    1 root     root            19 Dec 14 17:03 S01_mountrw -> ../init.d/remountrw
lrwxrwxrwx    1 root     root            19 Dec 14 17:03 S01_omsdaemon -> ../init.d/omsdaemon
lrwxrwxrwx    1 root     root            27 Dec 14 17:03 S02a_config_usb_gadget -> ../init.d/config_usb_gadget
lrwxrwxrwx    1 root     root            28 Dec 14 17:04 S02_determine_hostname -> ../init.d/determine_hostname
lrwxrwxrwx    1 root     root            20 Dec 14 16:14 S02_networking -> ../init.d/networking
lrwxrwxrwx    1 root     root            14 Dec 14 15:52 S03_dbus -> ../init.d/dbus
lrwxrwxrwx    1 root     root            17 Dec 14 15:52 S03_rc-once -> ../init.d/rc-once
lrwxrwxrwx    1 root     root            31 Dec 14 17:03 S04_auto_firmware_restore -> ../init.d/auto_firmware_restore
lrwxrwxrwx    1 root     root            15 Dec 14 16:14 S10_crond -> ../init.d/crond
lrwxrwxrwx    1 root     root            19 Dec 14 16:44 S10_syslog-ng -> ../init.d/syslog-ng
lrwxrwxrwx    1 root     root            15 Dec 14 16:14 S11_inetd -> ../init.d/inetd
lrwxrwxrwx    1 root     root            33 Dec 14 16:03 S12_validate_gateway_config -> ../init.d/validate_gateway_config
lrwxrwxrwx    1 root     root            18 Dec 14 16:03 S13_netconfd -> ../init.d/netconfd
lrwxrwxrwx    1 root     root            33 Dec 14 16:04 S14a_create_factory_settings -> ../init.d/create_factory_settings
lrwxrwxrwx    1 root     root            18 Dec 14 17:03 S14_mounthd2 -> ../init.d/mounthd2
lrwxrwxrwx    1 root     root            13 Dec 14 16:19 S15_drm -> ../init.d/drm
lrwxrwxrwx    1 root     root            14 Dec 14 17:03 S16_rauc -> ../init.d/rauc
lrwxrwxrwx    1 root     root            16 Dec 14 16:14 S17_sysctl -> ../init.d/sysctl
lrwxrwxrwx    1 root     root            19 Dec 14 15:52 S20_ledserver -> ../init.d/ledserver
lrwxrwxrwx    1 root     root            30 Dec 14 16:25 S20_writeSnmpDefaultConf -> ../init.d/writeSnmpDefaultConf
lrwxrwxrwx    1 root     root            20 Dec 14 15:52 S21_logforward -> ../init.d/logforward
lrwxrwxrwx    1 root     root            27 Dec 14 16:14 S21_networking-finish -> ../init.d/networking-finish
lrwxrwxrwx    1 root     root            18 Dec 14 17:02 S22_ipwatchd -> ../init.d/ipwatchd
lrwxrwxrwx    1 root     root            17 Dec 14 16:41 S24_dnsmasq -> ../init.d/dnsmasq
lrwxrwxrwx    1 root     root            18 Dec 14 16:50 S25_dropbear -> ../init.d/dropbear
lrwxrwxrwx    1 root     root            14 Dec 14 17:03 S60_mdmd -> ../init.d/mdmd
lrwxrwxrwx    1 root     root            22 Dec 14 16:44 S70_logbootevent -> ../init.d/logbootevent
lrwxrwxrwx    1 root     root            22 Dec 14 16:33 S86_opcua-server -> ../init.d/opcua-server
lrwxrwxrwx    1 root     root            24 Dec 14 17:03 S90_cbm_set_config -> ../init.d/cbm_set_config
lrwxrwxrwx    1 root     root            23 Dec 14 17:03 S90_config_serial -> ../init.d/config_serial
lrwxrwxrwx    1 root     root            20 Dec 14 16:03 S90_mdmd_check -> ../init.d/mdmd_check
lrwxrwxrwx    1 root     root            17 Dec 14 16:14 S90_modules -> ../init.d/modules
lrwxrwxrwx    1 root     root            29 Dec 14 16:55 S90_set_thread_priority -> ../init.d/set_thread_priority
lrwxrwxrwx    1 root     root            21 Dec 14 16:03 S90_telecontrol -> ../init.d/telecontrol
lrwxrwxrwx    1 root     root            26 Dec 14 16:14 S92_rt-set-bandwidth -> ../init.d/rt-set-bandwidth
lrwxrwxrwx    1 root     root            18 Dec 14 16:19 S95_progexec -> ../init.d/progexec
lrwxrwxrwx    1 root     root            18 Dec 14 16:49 S96_lighttpd -> ../init.d/lighttpd
lrwxrwxrwx    1 root     root            27 Dec 14 16:34 S97_serial_dispatcher -> ../init.d/serial_dispatcher
lrwxrwxrwx    1 root     root            27 Mar 11 11:27 S98_kbusmodbusslave -> /etc/init.d/kbusmodbusslave
lrwxrwxrwx    1 root     root            30 Dec 14 15:55 S99_featuredetect_switch -> ../init.d/featuredetect_switch
lrwxrwxrwx    1 root     root            23 Dec 14 17:03 S99_finalize_root -> ../init.d/finalize_root
lrwxrwxrwx    1 root     root            24 Dec 14 17:03 S99_logsystemstart -> ../init.d/logsystemstart
lrwxrwxrwx    1 root     root            18 Dec 14 17:03 S99_ssl_post -> ../init.d/ssl_post
```
Belangrijk: zet nu de schakelaar op de PLC op RUN ipv STOP. Het kbusmodbusslave programma zal daardoor automatisch starten na reboot.

```
root@PFC200-44194C:/etc/rc.d reboot
root@PFC200-44194C:/etc/rc.d Connection to 192.168.1.17 closed by remote host.
Connection to 192.168.1.17 closed.
pi@raspberrypi:~ $ 
```

#### Modbus adres-configuratie bekijken

1. Zet de schakelaar op de PLC van RUN naar STOP.
2. Login op de PLC en draai een commando
3. Kopy-paste die informatie hieronder.
4. Zet de schakelaar op de PLC van STOP naar RUN.


```
pi@raspberrypi:~ $ ssh root@192.168.1.17
root@192.168.1.17's password: 


WAGO Linux Terminal on PFC200-44194C.


root@PFC200-44194C:~ /etc/init.d/runtime stop
Terminate Runtime...done
root@PFC200-44194C:~ kbusmodbusslave --nodaemon -v8

======= CONFIGURATION =======
PORT: 502
MAX CONNECTIONS: 5
OPERATION MODE: 0
MODBUS DELAY MS: 0
KBUS CYCLE TIME MS: 50
KBUS PRIORITY: 60
==============================
verbosity level is 8
kbusmodbusslave running...
ADI Device[0]: libcanopenm
Modbus: Wait for KBUS to be initialized
ADI Device[1]: libcanopens
ADI Device[2]: libdps
ADI Device[3]: libmbs
ADI Device[4]: libpbdpm
ADI Device[5]: libpackbus
Found kbus device on: 5
KBUS device open OK
KBUS set to application state: 1

        .KbusBitCount: 288 
        .TerminalCount: 6 
        .ErrorCode: 0 
        .ErrorArg: 0 
        .ErrorPos: 0 
        .BitCountAnalogInput: 320 
        .BitCountAnalogOutput: 256 
        .BitCountDigitalInput: 16 
        .BitCountDigitalOutput: 16 
Offset: IN: 40 - OUT: 32

 Pos:1:	 Type: 750-4XX / 16DI	 BitOffsetOut:0;	 BitSizeOut:0;	 BitOffsetIn:320;	 BitSizeIn:16;	 Channels:0;	 PiFormat:0;
 Pos:2:	 Type: 750-5XX / 16DO	 BitOffsetOut:256;	 BitSizeOut:16;	 BitOffsetIn:0;	 BitSizeIn:0;	 Channels:0;	 PiFormat:0;
 Pos:3:	 Type: 750-450 / 0-0	 BitOffsetOut:0;	 BitSizeOut:0;	 BitOffsetIn:0;	 BitSizeIn:64;	 Channels:4;	 PiFormat:0;
 Pos:4:	 Type: 750-455 / 0-0	 BitOffsetOut:0;	 BitSizeOut:0;	 BitOffsetIn:64;	 BitSizeIn:64;	 Channels:4;	 PiFormat:0;
 Pos:5:	 Type: 750-555 / 0-0	 BitOffsetOut:0;	 BitSizeOut:64;	 BitOffsetIn:0;	 BitSizeIn:0;	 Channels:4;	 PiFormat:0;
 Pos:6:	 Type: 750-652 / 0-0	 BitOffsetOut:64;	 BitSizeOut:192;	 BitOffsetIn:128;	 BitSizeIn:192;	 Channels:1;	 PiFormat:0;
MKDIR /tmp/KBUS/ failed: File exists
Set Priority: 60 successfully
-->Init
Modbus Debug enabled
Watchdog Init
Modbus config Init
Modbus Config MAC Init
Modbus KBUS Info Init
Modbus const Init
Modbus ShortDesctiption Init
Modbus-Init complete - Ready for take off
--> STOP
--> Application in STOP mode
KBUS set to application state: 2
New Modbus connection from 192.168.1.2:27362 on socket 14
Waiting for a indication...
<00><01><00><00><00><06><01><03><00><14><00><01>
[00][01][00][00][00][03][01][83][06]
Waiting for a indication...
<00><02><00><00><00><06><01><03><00><14><00><01>
[00][02][00][00][00][03][01][83][06]
Waiting for a indication...
<00><03><00><00><00><06><01><03><00><14><00><01>
[00][03][00][00][00][03][01][83][06]
Waiting for a indication...
<00><04><00><00><00><06><01><03><00><14><00><01>
[00][04][00][00][00][03][01][83][06]
Waiting for a indication...
<00><05><00><00><00><06><01><03><00><14><00><01>
[00][05][00][00][00][03][01][83][06]
Waiting for a indication...
<00><06><00><00><00><06><01><03><00><14><00><01>
[00][06][00][00][00][03][01][83][06]
^CReceived Signal (Interrupt)
KBUS_CLOSE
KBUS_STOP
MODBUS STOP
Waiting for a indication...
<00><07><00><00><00><06><01><03><00><14><00><01>
[00][07][00][00][00][03][01][83][06]
Modbus loop exit
Watchdog stop
root@PFC200-44194C:~
```
Zet op dit moment die schakelaar weer naar RUN.

```
root@PFC200-44194C:~ reboot
root@PFC200-44194C:~ Connection to 192.168.1.17 closed by remote host.
Connection to 192.168.1.17 closed.
pi@raspberrypi:~ $ 
```
