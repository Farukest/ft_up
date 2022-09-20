	 / _____)             _              | |
	( (____  _____ ____ _| |_ _____  ____| |__
	 \____ \| ___ |    (_   _) ___ |/ ___)  _ \
	 _____) ) ____| | | || |_| ____( (___| | | |
	(______/|_____)_|_|_| \__)_____)\____)_| |_|
	  (C)2020 Semtech

SX1302 LoRa Gateway project
===========================

## 1. Core library: libloragw

This directory contains the sources of the library to build a gateway based on
a Semtech LoRa SX1302 concentrator chip (a.k.a. concentrator).
Once compiled all the code is contained in the libloragw.a file that will be
statically linked (ie. integrated in the final executable).

The library also comes with few basic tests programs that are used to test the
different sub-modules of the library.

Please refer to the readme.md file located in the libloragw directory for
more details.

## 2. Helper programs

Those programs are included in the project to provide examples on how to use
the HAL library, and to help the system builder test different parts of it.

### 2.1. packet_forwarder ###

The packet forwarder is a program running on the host of a Lora gateway that
forwards RF packets receive by the concentrator to a server through a IP/UDP
link, and emits RF packets that are sent by the server.

	((( Y )))
	    |
	    |
	+- -|- - - - - - - - - - - - -+        xxxxxxxxxxxx          +--------+
	|+--+-----------+     +------+|       xx x  x     xxx        |        |
	||              | USB |      ||      xx  Internet  xx        |        |
	|| Concentrator |<----+ Host |<------xx     or    xx-------->|        |
	||              | SPI |      ||      xx  Intranet  xx        | Server |
	|+--------------+     +------+|       xxxx   x   xxxx        |        |
	|   ^                    ^    |           xxxxxxxx           |        |
	|   | PPS  +-----+  NMEA |    |                              |        |
	|   +------| GPS |-------+    |                              +--------+
	|          +-----+            |
	|                             |
	|            Gateway          |
	+- - - - - - - - - - - - - - -+

Uplink: radio packets received by the gateway, with metadata added by the
gateway, forwarded to the server. Might also include gateway status.

Downlink: packets generated by the server, with additional metadata, to be
transmitted by the gateway on the radio channel. Might also include
configuration data for the gateway.

Please refer to the readme.md file located in the packet_forwarder directory
for more details.

### 2.2. util_net_downlink ###

The downlink packet sender is a simple helper program listening on a single
UDP port, responding to PUSH_DATA and PULL_DATA datagrams with proper ACK, and
sending downlink JSON packets to the socket, with given frame parameters, at
regular time interval.
It is a network packet sender.

It can also be used as a UDP packet logger to store received uplinks in a
local CSV file.

Please refer to the readme.md file located in the util_net_downlink directory
for more details.

### 2.3. util_chip_id ###

This utility configures the SX1302 to be able to retrieve its EUI. It can then
be used as a Gateway ID.

### 2.4. util_boot ###

On used for a USB gateway, this software switches the concentrator in DFU mode
in order to program its internal STM32 MCU.

### 2.5. util_spectral_scan ###

This software allows to scan the spectral band using the additional sx1261 radio
of the Semtech Corecell reference design.

## 3. Helper scripts

### 3.1. tools/reset_lgw.sh

This script is used to perform the basic initialization of the SX1302 through
the GPIOs defined by the CoreCell reference design.
It gets the SX1302 out of reset and set the Power Enable pin.
This script is called by every program provided here which accesses the SX1302.
It MUST be located in the same directory as the executable of the program.

## 4. Compile, install and run instructions

All the libraries and test programs can be compiled and installed from the
root directory of this project.

### 4.1. Clean and compile everything

`make clean all`

### 4.2. Install executables and associated files in one directory

First edit the target.cfg file located in the root directory of the project
in order to configure where the executables have to be installed.

`TARGET_IP` : sets the IP address of the host of the gateway. In case the
project is compiled on the gateway host itself (Raspberry Pi...), this can
be left set to `localhost`.

`TARGET_DIR` : sets the directory on the gateway host file system in which
the executables must be copied. Note that the directory MUST exist when
invoking the install command.

`TARGET_USR` : sets the linux user to be used to perform the SSH/SCP command
for copying the executables.

In order to avoid entering the user password when installing the files, the
following steps have to be followed.

Lets say you want to copy between two hosts host_src and host_dest (they can
be the same). host_src is the host where you would run the scp command,
irrespective of the direction of the file copy!

* On host_src, run this command as the user that runs scp<br/>
`ssh-keygen -t rsa`

This will prompt for a passphrase. Just press the enter key. It'll then
generate an identification (private key) and a public key. Do not ever share
the private key with anyone! ssh-keygen shows where it saved the public key.
This is by default ~/.ssh/id_rsa.pub
* Transfer the id_rsa.pub file to host_dest<br/>
`ssh-copy-id -i ~/.ssh/id_rsa.pub user@host_dest`

You should be able to log on host_dest without being asked for a password.

Now that everything is set, the following command can be invoked:<br/>
`make install`

In order to also install the packet forwarder JSON configuration files:<br/>
`make install_conf`

### 4.3. Cross-compile from a PC

* Add the path to the binaries of the compiler corresponding to the target
platform to the `PATH` environment variable.
* set the `ARCH` environment variable to `arm`.
* set the `CROSS_COMPILE` environment variable to the prefix corresponding to
the compiler for the target platform.

An example for a Raspberry Pi target:

* `export PATH=[path]/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin`
* `export ARCH=arm`
* `export CROSS_COMPILE=arm-linux-gnueabihf-`

Then, from the same console where the previous environment variables have been
set, do:

`make clean all`

## 5. USB

This project provides support for both SPI or USB gateways. For USB interface,
the concentrator board has a STM32 MCU with which the linux host will
communicate to configure the sx1302 and the associated radios. The STM32 acts
as a USB <-> SPI bridge.

The STM32 MCU has to be programmed with the binary provided in the `mcu_bin`
directory of this project. For more details about how to flash it, please refer
to the `util_boot/readme.md` instructions.

Each test utility of the project can be used using the `-u -d /dev/ttyACMx'
command line option, or with the proper configuration in the packet forwarder
global_conf.json file.

## 6. Third party libraries

This project relies on several third-party open source libraries, that can be
found in the `libtools` directory.
* parson: a JSON parser (http://kgabis.github.com/parson/)
* tinymt32: a pseudo-random generator (only used for debug/test)

## 7. Changelog

### v2.1.0 ###

> #### Updates

This release only targets USB Corecell (no change for SPI connexion type).
The USB-SPI bridge firmware, which runs on the STM32 MCU of the USB Corecell, has
been updated for API clean-up and robustness improvements.

> #### Changes

* MCU: USB-SPI bridge firmware binary v1.0.0.
	* Removed obsolete commands (ORDER_ID__REQ_SPI, ORDER_ID__ACK_SPI)
	* Command index shifted after obsolete commands removal (ORDER_ID__REQ_MULTIPLE_SPI, ORDER_ID__ACK_MULTIPLE_SPI)
	* Command parser sends ORDER_ID__UNKNOW_CMD in case of wrong command size.
	* Code clean-up (typo fixed, comments added...)
	* Implemented Error_Handler() function to reset the MCU in case of fatal error.
	* Fixed a potential roll-over issue in read_write_spi() function.
	* Increased delay tolerance for host feedback on USB transfers.
* HAL: Command interface updated for MCU firmware v1.0.0.
	* Removed obsolete commands from enum order_id_e
	* Shifted commands enum index according to USB-SPI bridge update.
	* Removed decode_ack_spi_access() unused function.
* HAL: Added timing debug information under DEBUG_MCU.

### v2.0.2 ###

> #### Updates

Fixed AGC firmware version check for sx1255/sx1257 based platforms (full-duplex
gateways...).

> #### Changes

* HAL: AGC firmware version for sx1255/sx1257 based gateways is v6.
* HAL: minor cosmetic changes & typo fixing.

### v2.0.1 ###

> #### Updates

The fine timestamping feature has been fully validated with this release.

> #### Changes

* HAL: Adjusted the freq_offset field of received packets, to take into account
the channel IF resolution error.
* HAL: Refined the fine timestamp offset compared to Gateway v2, by taking into
account the frequency offset of the received packet.
* HAL: Fixed the preamble length for FSK downlinks
* MCU: Removed the binary compiled in debug mode.
* util_spectral_scan: actually use the nb_scan input argument which was ignored.

### v2.0.0 ###

> #### New features

* Added support for USB interface between the HOST and the concentrator,
for sx1250 based concentrator only.
* Added support for Listen-Before-Talk for AS923 region, using the additional
sx1261 radio from the Semtech Corecell reference design v3.
* Added support for Spectral Scan with additional sx1261 radio from the Semtech
Corecell reference design v3.
* Added support for SX1303 chip, for further Fine Timestamping support.
* Merged the master-fdd-cn490 branch to bring support for CN490 Full-Duplex
reference design. It is an integration of the releases v1.1.0, v1.1.1, v1.1.2
described below.

> #### Changes

* HAL: Reworked the complete communication layer. A new loragw_com module has
been introduced to handle switching from a USB or a SPI communication interface,
aligned function prototypes for sx125x, sx1250 and sx1261 radios. For USB, a
mode has been added to group SPI write commands request to the STM32 MCU, in
order to optimize latency during time critical configuration phases.
* HAL: Added preliminary support for Fine Timestamping for TDOA localization.
* HAL: Updated AGC firmware to v6: add configurable delay for PA to start, add
Listen-Before-Talk support.
* HAL: Added new API function lgw_demod_setconf() to set global demodulator
settings.
* HAL: Added new API functions for Spectral Scan.
* Packet Forwarder: The type of interface is configurable in the
global_conf.json file: com_type can be "USB" or "SPI".
* Packet Forwarder: Changed the parameters to configure fine timestamping in the
global_conf.json.
* Packet Forwarder: Added sections to configure the spectral scan and
Listen-Before-Talk features.
* Packet Forwarder: Added a new thread for background spectral scan example,
to show how to use the spectral scan API provided by the HAL, without
interfering with the main tasks of the gateway (aka Receive uplinks and transmit
downlinks).
* Packet Forwarder: Added "nhdr" field parsing from "txpk" JSON downlink request
in order to be able to send beacon request from Network Server.
* Packet Forwarder: Added chan_multiSF_All in global_conf.json to choose which
spreading factors to enable for multi-sf demodulators.
* Packet Forwarder: Updated PROTOCOL.md to v1.6.
* Tools: added util_spectral_scan, a standalone spectral scanner utility.

> #### Notes

* This release has been validated on the Semtech Corecell reference design v3
with USB interface.

### v1.1.2 ###

> Integrated in ***v2.0.0*** from ***master-fdd-cn490*** branch.

* packet forwarder: updated global_conf.json.sx1255.CN490.full-duplex with RSSI
temperature compensation coefficients, and updated RSSI offset for radio 1.

### v1.1.1 ###

> Integrated in ***v2.0.0*** from ***master-fdd-cn490*** branch.

* HAL: Updated SX1302 LNA/PA LUT configuration for Full Duplex CN490 reference
design.
* test_loragw_hal_rx/tx: added --fdd option to enable Full Duplex
* packet forwarder: updated global_conf.json.sx1255.CN490.full-duplex for CN490
reference design.

### v1.1.0 ###

> Integrated in ***v2.0.0*** from ***master-fdd-cn490*** branch.

* HAL: Added support for CN490 full duplex reference design.

### v1.0.5 ###

* HAL: Fixed packet timestamp issue which was "jumping in time" in specific
conditions.
* HAL: Workaround hardware issue when reading 32-bits registers (timestamp, nb
bytes in RX buffer...)
* HAL: Fixed potential endless loop in sx1302_tx_abort() in SPI access fails.
* Packet Forwarder: Added global_conf.json.sx1250.US915 for US915 band
* test_hal_rx: added command line to specify RSSI offset to be applied

### v1.0.4 ###

* Added missing LICENSE.TXT file
* HAL & Packet Forwarder: added support for sx1250-based reference design for
CN490 region
* Packet Forwarder: disabled beaconing by default

### v1.0.3 ###

* HAL: Fixed scheduled downlink time precision by taking the tx start delay into
account.
* HAL: Fixed timestamp correction calculation for BW250 & BW500
* HAL: Fixed possible buffer overflow in lgw_receive() function
* HAL: Keep packet received in RX buffer when the buffer allocated to receive
the packets is too small. Remaining packets will be fetched on the next
lgw_receive calls (aligned on SX1301 behaviour).

### v1.0.2 ###

* Fixed compilation warnings reported by latest versions of GCC
* Reworked handling of temperature sensor
* Clean-up of unused files
* Added instructions and configuration files for packet forwarder auto-start
with systemd
* Added SX1250 radio calibration at startup

### v1.0.1 ###

* Packet Forwarder: Updated TX gain LUT in global_conf.json.sx1250 with proper
calibration

### v1.0.0 ###

* HAL: Initial official release for SX1302 CoreCell Reference Design.

### v0.0.1 ###

* HAL: Initial private release for TAP program

## 8. Legal notice

The information presented in this project documentation does not form part of
any quotation or contract, is believed to be accurate and reliable and may be
changed without notice. No liability will be accepted by the publisher for any
consequence of its use. Publication thereof does not convey nor imply any
license under patent or other industrial or intellectual property rights.
Semtech assumes no responsibility or liability whatsoever for any failure or
unexpected operation resulting from misuse, neglect improper installation,
repair or improper handling or unusual physical or electrical stress
including, but not limited to, exposure to parameters beyond the specified
maximum ratings or operation outside the specified range.

SEMTECH PRODUCTS ARE NOT DESIGNED, INTENDED, AUTHORIZED OR WARRANTED TO BE
SUITABLE FOR USE IN LIFE-SUPPORT APPLICATIONS, DEVICES OR SYSTEMS OR OTHER
CRITICAL APPLICATIONS. INCLUSION OF SEMTECH PRODUCTS IN SUCH APPLICATIONS IS
UNDERSTOOD TO BE UNDERTAKEN SOLELY AT THE CUSTOMER'S OWN RISK. Should a
customer purchase or use Semtech products for any such unauthorized
application, the customer shall indemnify and hold Semtech and its officers,
employees, subsidiaries, affiliates, and distributors harmless against all
claims, costs damages and attorney fees which could arise.

*EOF*