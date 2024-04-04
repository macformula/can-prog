# CAN - Flash Documentation

> *For detailed explanation of concepts see [this document](https://docs.google.com/document/d/1RpIHiQbIHJfaAxLyOGzX1HnmwUP_ogc84IS5asevDy8/edit?usp=sharing)*

## Terminology

| Term | Explanation |
| --- | --- |
| BOOT_ADD0 & BOOT_ADD1 | An option byte (programmable setting) ; set in STM32 Cube Programmer |
| Boot Pin | A hardware pin on the STM32F767ZI ; controls which memory area to boot into |
| STM32 Cube Programmer | An application which allows the flashing, and memory manipulation of STM32 MCU’s |
| System Bootloader | A bootloader made by STM located at memory address: 0x0100 0000. Enables flashing the MCU via CAN, UART, SPI and more… |

## Setup

1. Boot Option Bytes:
    1. There are two option bytes relevant to CAN Flash, these are BOOT_ADD0 and BOOT_ADD1
    2. Set BOOT_ADD0 to value 0x80 (address 0x00200000), and BOOT_ADD1 to value 0x40 (address 0x00100000) in Option Bytes of STM32CubeProgrammer. If option byte values are already configured as such, leave as is. See Figure 1 to see different boot addresses
    3. These options bytes are chosen by the firmware on reset via the boot pin. If the boot pin is LOW, BOOT_ADD0 is chosen and vice versa.
    2. Verify:
        1. Tie the boot pin to high ; choose BOOT_ADD1
        2. In STM32 Cube Programmer: go into MCU Core tab > press any reset option
        3. In STM32 Cube Programmer: go into MCU Core tab > press run and then halt
        4. In STM32 Cube Programmer: go into MCU Core tab > check if the LR ( instruction pointer ) register is in the memory range of 0x1FFX XXXX
        5. If so, the MCU is currently in system bootloader

2. Read/Write protection option bytes:
    1. While BOOT_ADD0 and BOOT_ADD1 are the most important, other option bytes must be left in their DEFAULT to enable reading and writing to flash memory
    2. These are the write protection option bytes: nWRP# and read protection option byte: RDP
    3. Ensure all these option bytes are left as their default configuration. See Figure 2 for default option bytes.
    4. For RDP: DO NOT CHANGE IT TO CC (Level 3 Protection) → DEBUG OPTIONS WILL BE DISABLED FOREVER
    5. Verify:
        1. Check with Figure 2
        2. In STM32 Cube Programmer : go into Erasing & Programming tab > do a full chip erase for flash memory
        3. If it doesn’t work, your read/write protections are set wrong

3. Can-prog download, and raspberry pi setup:
    1. In order to program the STM32 MCU with can-prog you will need a computer with a CAN interface → in testing we used a raspberry pi with a CAN Hat for our CAN bus setup
    2. For flashing with CAN we will need a bitrate of 125000 for our CAN network
    3. Using `ifconfig` you can check which networks are set up on your system → you should see either can0 or can1 ; either is fine to use, we will use can1
    4. Verify:
        1. Using `ip -details -statistics link show can1` ; see if the CAN network is configured at the required bitrate of 125000. If so, skip the steps below
        2. Using `sudo ip link set can1 down` ; pull the CAN network down 
        3. Using `sudo ip link set can1 type can bitrate 125000` ; configure the CAN network to use the required bitrate of 125000
        4. Using `sudo ip link set can1 up` ; bring the CAN network up again
    5. Git clone this repo’s version of can-prog and install

4. CAN - 2 Wiring, BOOT pin setup and CAN Transceiver:
    1. You will need a CAN transceiver for the STM32. We require the CAN-2 interface on the STM32, specifically pins PB_5 for CAN_RX and PB_13 for CAN_TX.
    2. Drive BOOT pin high (See Figure 3)
    3. Connect the CAN bus by connecting CAN2_RD from PB_5 and CAN2_TD from PB_13 into CAN_RX and CAN_TX on the CAN transceiver. Note that there are other CAN2_RD and CAN2_TD pins, but it must be from PB_5 and PB_13. Also note that there are multiple PB_5 and PB_13 pins, and that PB_5 and PB_13 must be connected to the ones in Figure 4.a OR Figure 4.b, but must not be combined (i.e. do not connect PB_5 from Figure 4.a, and PB_13 from Figure 4.b)

## Procedure

1. Write the binary to the board
    1. i.e canprog -n can0 -f bin stm32 write [file_name] -a 0x08000000
        1. -n →  interface name
        2. -f → file format (hex, bin)
        3. -a → address to write → default is 0x0800 0000 DON’T CHANGE
        4. [file_name] → file to write
2. Run the flashed application
    1. i.e canprog -n can0 stm32 go -a 0x08000000

## Additional Notes

1. Unlock, lock, and speed commands with can-prog tool are not necessary
    1. The unlock and lock commands change the readout protection and option bytes ; in our use case we do not need to lock the chip after flashing → therefore do not use
    2. The speed command changes the bitrate of the CAN-2 interface on the STM32, however the default is already set correctly upon system reset → therefore do not use

## Appendix

![]()
**Figure 1**

![]()
**Figure 2**

![]()
**Figure 3**

![]()
**Figure 4.a**

![]()
**Figure 4.b**