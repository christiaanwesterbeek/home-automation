

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
