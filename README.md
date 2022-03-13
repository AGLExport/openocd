# Attention:  
This README is R-Car M3/H3 ulcb specific information.

# Focusing hardware

## JTAG adapters

ARM-USB-OCD-H

## Debug targets

Renesas R-Car M3/H3 ulcb Cortex-R7 core


# Building OpenOCD

## OpenOCD Dependencies

- make
- libtool
- pkg-config >= 0.23 (or compatible)
- autoconf >= 2.64
- automake >= 1.14
- texinfo
- libftdi
- libusb


## Compiling OpenOCD

To build OpenOCD, use the following sequence of commands:

	$ ./bootstrap
	$ ./configure --prefix=/usr
	$ make
	$ sudo make install

## Download cross GDB

	$ wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/10.3-2021.10/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2
	$ tar xvjf gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2

#  OpenOCD Debugging setup

## Setting for ulcb

The R-Car M3/H3 ulcb is booting from Coretex-A57-0 core in default.  
When you are debugging for Cortex-R7 firmware, need to change Coretex-R7 boot mode.  
The boot mode can change in u-boot console.

After booting by Cortex-A57,  you can use u-boot console.

#### Before
	=> cpld info
	CPLD version:                   0x20160923
	H3 Mode setting (MD0..28):      0x00218128
	Multiplexer settings:           0x0000000e
	DIPSW (SW6):                    0x0000000f

#### Boot mode change command
	=> cpld write 0 0x802181e8

#### After
	=> cpld info
	CPLD version:                   0x20160923
	H3 Mode setting (MD0..28):      0x802181e8
	Multiplexer settings:           0x0000000e
	DIPSW (SW6):                    0x0000000f


This setting is keeping for the duration of the power supply.

When you push reset button, your board boot from Cortex-R7.  
Typically, it boot SCIF download mode.

	Parameter Error!!
	0x00000000 0x002181E8 0x00000008 0x00000000
	
	 SCIF Download mode (w/o verification)
	 (C) Renesas Electronics Corp.
	
	-- Load Program to SystemRAM ---------------
	Work RAM(H'E6300000-H'E632E800) Clear....
	please send !

## Exec open OCD

After the board reset, you can connect M3/H3 ulcb using ARM-USB-OCD-H.

#### Linux host console No.1

	$ sudo openocd -f /usr/share/openocd/scripts/interface/ftdi/olimex-arm-usb-ocd-h.cfg -f /usr/share/openocd/scripts/board/renesas_m3ulcb-cr7.cfg

	Open On-Chip Debugger 0.10.0+dev-01512-g2c5cc09ea-dirty (2022-03-13-01:03)
	Licensed under GNU GPL v2
	For bug reports, read
	        http://openocd.org/doc/doxygen/bugs.html
	adapter speed: 20000 kHz
	
	
	M3ULCB:
	        M3W-CR7 - 2 CA57(s), 4 CA53(s), 1 CR7(s)
	        Boot Core - CR7
	
	Info : auto-selecting first available session transport "jtag". To override use 'transport select <transport>'.
	Info : Listening on port 6666 for tcl connections
	Info : Listening on port 4444 for telnet connections
	Info : ftdi: if you experience problems at higher adapter clocks, try the command "ftdi_tdo_sample_edge falling"
	Info : clock speed 20000 kHz
	Info : JTAG tap: r8a77960.cpu tap/device found: 0x5ba00477 (mfg: 0x23b (ARM Ltd.), part: 0xba00, ver: 0x5)
	Info : r8a77960.r7: hardware has 6 breakpoints, 4 watchpoints
	Info : starting gdb server for r8a77960.r7 on 3333
	Info : Listening on port 3333 for gdb connections
	Info : starting gdb server for r8a77960.a57.0 on 3334
	Info : Listening on port 3334 for gdb connections

#### Linux host console No.2

	telnet 127.0.0.1 4444
	r8a77950.r7 arp_examine


#### Linux host console No.3

	/path/to/croogdb/arm-none-eabi-gdb

	(gdb) target remote localhost:3333
	(gdb) interrupt
	(gdb) monitor reset halt

##### Test

	(gdb) info registers
	r0             0xe632ff38          3862101816
	r1             0xe6e88010          3873996816
	r2             0xe6e88024          3873996836
	r3             0x0                 0
	r4             0x0                 0
	r5             0xe632ffb0          3862101936
	r6             0xe632ff38          3862101816
	r7             0xe632ffac          3862101932
	r8             0xffffffff          4294967295
	r9             0xe6708004          3866132484
	r10            0xe632ff34          3862101812
	r11            0xe632ff18          3862101784
	r12            0xfffe              65534
	sp             0xe632ff10          0xe632ff10
	lr             0xff6e              0xff6e
	pc             0xeb103b20          0xeb103b20
	cpsr           0x600001d3          1610613203
	sp_usr         0x0                 0x0
	lr_usr         0x0                 0x0
	r8_fiq         0x0                 0
	r9_fiq         0x0                 0
	r10_fiq        0x0                 0
	r11_fiq        0x0                 0
	r12_fiq        0x0                 0
	sp_fiq         0x0                 0x0
	lr_fiq         0x0                 0x0
	sp_irq         0x0                 0x0
	lr_irq         0x0                 0x0
	sp_svc         0xe632ffc8          0xe632ffc8
	lr_svc         0xeb1006c8          0xeb1006c8
	sp_abt         0x0                 0x0
	lr_abt         0x0                 0x0
	sp_und         0x0                 0x0
	lr_und         0x0                 0x0
	spsr_fiq       0x42057252          1107653202
	spsr_irq       0x78004d03          2013285635
	spsr_svc       0xd0000081          3489661057
	spsr_abt       0xc001d88c          3221346444
	spsr_und       0x38020284          939655812
	(gdb) monitor reset halt
	JTAG tap: r8a77960.cpu tap/device found: 0x5ba00477 (mfg: 0x23b (ARM Ltd.), part: 0xba00, ver: 0x5)
	Debug regions are unpowered, an unexpected reset might have happened
	JTAG-DP STICKY ERROR
	DAP transaction stalled (WAIT) - slowing down
	DAP transaction stalled (WAIT) - slowing down
	DAP transaction stalled (WAIT) - slowing down
	DAP transaction stalled (WAIT) - slowing down
	DAP transaction stalled (WAIT) - slowing down
	DAP transaction stalled (WAIT) - slowing down
	DAP transaction stalled (WAIT) - slowing down
	DAP transaction stalled (WAIT) - slowing down
	DAP transaction stalled (WAIT) - slowing down
	Deferring arp_examine of r8a77960.a57.0
	Use arp_examine command to examine it manually!
	Deferring arp_examine of r8a77960.a57.1
	Use arp_examine command to examine it manually!
	Deferring arp_examine of r8a77960.a53.0
	Use arp_examine command to examine it manually!
	Deferring arp_examine of r8a77960.a53.1
	Use arp_examine command to examine it manually!
	Deferring arp_examine of r8a77960.a53.2
	Use arp_examine command to examine it manually!
	Deferring arp_examine of r8a77960.a53.3
	Use arp_examine command to examine it manually!
	r8a77960.r7: ran after reset and before halt ...
	target halted in ARM state due to debug-request, current mode: Supervisor
	cpsr: 0x600001d3 pc: 0xeb1025fc
	D-Cache: disabled, I-Cache: enabled
	(gdb)


