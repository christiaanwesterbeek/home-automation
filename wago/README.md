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
![image](https://user-images.githubusercontent.com/465989/157834401-8b54a2fa-387c-40e1-9b62-847f686c2c19.png)

6. Log in met het fabriekgebruikersnaam en wachtwoord admin/wago en wijzig het wachtwoord naar prototype.

7. Stel het statische IP-adres 192.168.1.17 in met "Wago Ethernet Settings".

![image](https://user-images.githubusercontent.com/465989/157834753-f473c3f3-455e-450a-a5e2-f24da2e1459b.png)

8. Klik op submit en open daarna in de browser https://192.168.1.17/ en login met admin/prototype

9. Nu kun je de bootable image op de SD card kopieren naar het interne geheugen (flash) van de PLC. Ga daarvoor naar wbm / administration / Create Image. Bij "Create bootable image from active partition (SD)", klik op "Start Copy".
.|.
---|---
![image](https://user-images.githubusercontent.com/465989/157837772-341c2a15-1d3c-4888-88ea-d428d3e2ee55.png)|![image](https://user-images.githubusercontent.com/465989/157838123-b8e26877-6019-412b-899e-223e3b910261.png)


## Adding KbusModbusPFCSlave to the PFC

Via https://github.com/WAGO/pfc-howtos gevonden.

Using Web-Based-Management(WBM) feature "Software-Upload" for upload and installing OPKG packages

```
root@PFC200-44194C:~ kbusmodbusslave --nodaemon -v7

======= CONFIGURATION =======
PORT: 502
MAX CONNECTIONS: 5
OPERATION MODE: 0
MODBUS DELAY MS: 0
KBUS CYCLE TIME MS: 50
KBUS PRIORITY: 60
==============================
verbosity level is 7
kbusmodbusslave running...
Modbus: Wait for KBUS to be initialized
ADI Device[0]: libcanopenm
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
--> RUN
--> Application in RUN mode
KBUS set to application state: 1
New Modbus connection from 192.168.1.2:39646 on socket 15
Waiting for a indication...
<00><01><00><00><00><06><01><03><9C><41><00><64>
Watchdog trigger
Function: 3
[00][01][00][00][00][03][01][83][02]
Waiting for a indication...
<00><02><00><00><00><06><01><03><9C><41><00><64>
Watchdog trigger
Function: 3
```
