## Modbus

### Introduction

Once you understand how the PLC's memory is structured and how the variable addresses work to store their values in a desired place (see previous article), you can try to use the MODBUS protocol.  It is an industry standard for a communication between various systems.  It can be used for connecting a HMI panel to your PLC, connecting a 1-wire module or connecting the PLC to your PC...

WARNING! All that you find here is an amateur attempt to cover a complex subject with a simple mind.  I ask the experts to have understanding.  Unfortunately none of the 'professional' descriptions of the protocol was understandable to me.  What you see below is a composition of bits an pieces found on random sites.

The MODBUS communication is done by executing 'functions'.  They can be split into two main categories: 1st for bit (BOOL) variables and the 2nd for WORD.  Then that distinction can be split again into variables, which are read-only and those which can be overwritten.

Here is a table which shows the split of the main MODBUS functions:

<table style="text-align: center; width: 80%;" border="2 solid white">
<tbody>
<tr>
<td width="20%">&nbsp;</td>
<td width="40%">Read/Write</td>
<td width="40%">Read only</td>
</tr>
<tr>
<td>BIT/BOOL</td>
<td>
<p>FC1 - Read Coil<br>FC5 - Write Coil<br>FC15 -&nbsp;Write Multiple Coils</p>
</td>
<td>&nbsp;FC2 -&nbsp;Read Discrete Input</td>
</tr>
<tr>
<td>WORD</td>
<td>
<p>FC3 - Read Holding Register<br>FC6 - Write Single Register<br>FC16 - Write Multiple Registers<br>FC23 - Read/Write multiple registers</p>
</td>
<td>FC4 -&nbsp;Read Input Registers</td>
</tr>
</tbody>
</table>
 

To be honest, I do not like those function names.  I do not know why a 'coil' or a 'discrete input'.  What matters is that in order to read a single bit I can use FC1 or FC2, to write it - FC5 or FC15; for WORD values the functions are: FC3, FC4 for reading and FC6, FC16 or FC23 to write.  That's all.

To clarify - MODBUS can let you read or modify some areas of your PLC's memory.  It does not accept commands like "turn on the lights" or "show me the temperature".  To do that you need to reach for a specific part of the PLC's memory and interpret it.  

Now it is time to understand how the addresses of MODBUS protocol fit the map of addresses in PLC's memory.  In the PLC manual one can find a whole chapter titled "MODBUS Register Mapping".  You can find there tables showing the link between MODBUS and WAGO controllers.  

In case of reading/writing single bits (functions FC1, FC2 and FC5) the following values need to be used.  In case of digital ins or outs (DI or DO) one need to know their consecutive number, not their address in PLC's memory:

<table style="text-align: center; width: 80%;" border="2 solid white">
<tbody>
<tr>
<td width="20%">Address MODBUS [dec]</td>
<td width="40%">Memory range</td>
<td width="40%">Description</td>
</tr>
<tr>
<td>0...511</td>
<td>
<p>Physical inputs</p>
</td>
<td>First 512 digital inputs</td>
</tr>
<tr>
<td>512...1023</td>
<td>
<p>Physical outputs</p>
</td>
<td><span style="text-align: center;">First&nbsp;</span>512 digital outputs</td>
</tr>
<tr>
<td>12288...32767</td>
<td>
<p>%MX0...%MX1279.15</p>
</td>
<td>
<p>NOVRAM <br>retain memory</p>
</td>
</tr>
</tbody>
</table>

For WORD variables (functions FC3 i FC4):

<table style="text-align: center; width: 80%;" border="2 solid white">
<tbody>
<tr>
<td width="20%">Adres MODBUS &nbsp;[dec]</td>
<td width="40%"><span style="text-align: center;">Memory range</span></td>
<td width="40%"><span style="text-align: center;">Description</span></td>
</tr>
<tr>
<td>0...255</td>
<td>
<p>%IW0...%IW255</p>
</td>
<td><span style="text-align: center;">Physical input area</span></td>
</tr>
<tr>
<td>512...767</td>
<td>
<p>%QW0...%QW255</p>
</td>
<td><span style="text-align: center;">Physical output area</span></td>
</tr>
<tr>
<td>12288...24575</td>
<td>
<p>%MW0...%MW12287</p>
</td>
<td>
<p>NOVRAM&nbsp;<br>retain memory</p>
</td>
</tr>
</tbody>
</table>

So, let's look at some examples:

in order to read the status on input 0 use function FC1 and address 0
in order to read the status of output 3 use function FC1 and address 514
in order to read the BOOL value stored at %MX0.3, use function FC1 and address 12 291
in order to read the first word of the table holding statuses of inputs, use function FC3 and address 0
in order to read the first word of the table holding statuses of outputs, use function FC3 and address 512
in order to read a WORD variable stored at %MW0, use function FC3 and address 12288,
Writing values is very similar.  One just has to remember that PLC's inputs are read-only.

bron: https://www.edom-plc.pl/index.php/en/more-about-plc/functions/198-wstep-do-modbusa

## node-red and modbus

https://stevesnoderedguide.com/node-red-modbus
