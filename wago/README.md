## WAGO 750-8203

### Doel

Het doel is niet om in de PLC te programmeren, maar de DI porten uitlezen en DO porten aansturen via Modbus TCP. Dat uitlezen aansturen wil ik met een Raspberry PI doen waar ik de logica in programmeer met node-red software. In dat geval is het niet nodig om Codesys te gebruiken (in andere worden de Codesys runtime te draaien op de PLC).

Het is mogelijk om DOCKER containers op de WAGO PFC te draaien waaronder ook een MODBUS container. Hoe je dit kunt doen, wordt in de onderstaande Youtube film uitgelegd. [WAGO PFC200 G2 Modbus Slave KBUS Docker Container](https://www.youtube.com/watch?v=TCV_BqcwM30). Maar dit lijkt niet mogelijk zou later blijken, omdat tijdens het uploaden van de docker.ipk file een error optreedt.

### Installatie
Dit stappenplan gaat uit van een uitgevoerde factory reset middels de volgende handeling beschreven in de [handleiding](https://asset.conrad.com/media10/add/160267/c1/-/en/001369150ML02/mode-demploi-1369150-api-controleur-wago-pfc200-2eth-can-750-8203-24-vdc-1-pcs.pdf).

> Set the mode selector switch to “RESET” and press the Reset button (RST) for 1 … 8 seconds. Briefly release the Reset button (RST) (< 1 second) and press it again until the “CAN” LED lights up red. If the “CAN” LED lights up red, release the mode selector switch and the Reset button. After the first 1 … 8 seconds, the controller reboots (all LEDs light up orange) and after another 3 seconds, the “Factory reset” process begins. The process is indicated by all LEDs lighting up red in succession.

Als dat goed is gegaan, branden alle 12 leds (SYS, RUN, I/O, etc) ononderbroken rood. Daarna zet je de PLC aan en uit. Dan zijn de SYS en I/O led's ononderbroken groen, en de RUN led knippert groen, en de andere leds zijn uit. Als de RUN-led groen knippert, betekent dit dat het PLC-programma in een debug is. Dit is de uitgangssituatie voor de verdere installatie.

#### Firmware update

...

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
