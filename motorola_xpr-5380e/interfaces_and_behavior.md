# interfaces
## USB through rear accessory port
### main/normal/app mode
USB: 1 device, 2 configurations (RNDIS for windows and CDC-ECM for linux)
IP (default 192.168.10.1)
TCP ports 8002 (XNL), 8501 (AT debug), 8502 (GPS passthrough). Unverified: TCP 8004 (secure XNL), UDP 4009 (OTAP RUDP), 4010 (OTAP switchover), 4011 (net filter), 4014 (XCMP over CAI), 4064 (air tracer), 64414 (OTAR)
XNL
XCMP


PORT      STATE         SERVICE
4009/udp  open|filtered chimera-hwm
4010/udp  open|filtered samsung-unidex
4011/udp  open|filtered altserviceboot
4014/udp  closed        taiclock
4064/udp  open|filtered ice-srouter
64414/udp closed        unknown

### test mode

USB same as above
TCP same as above EXCEPT 8809 additionally open (unknown)
UDP same as above




### boot mode
USB: 1 device, 1 configuration, 1 interface (vendor specific - Motorola Flash), 2 endpoints (bulk in/bulk out)
on windows, meanto to use flashzap (fudally_usb.dll/fudally.sys). send packets on 1 endpoint, receive on other endpoint
SBEP
XCMP

## wifi
not tested

## ethernet interface for remote control head
not tested

# behavior

ramloader is written using WRITEMEM, but READMEM is unsupported both in boot mode and normal mode
old ramloader writes to 0x1040_0000, which returns invalid option
new but wrong version ramloader writes to 0xc040_0000 successful, and BOOTJMPEXEC looks succesful too! can then unlock security!
other partitions besides ramloader are written using FTL_ACCESS or FILE_ACCESS

FTL type

L2_BOOTLOADER:          id=0x0010, optionFlags=0, start=0x0000, end=0x0036
GLOBAL_MARKER:          id=0x0020, optionFlags=0, start=0x0000, end=0x0000
PSDT:                   id=0x0021, optionFlags=0, start=0x0001, end=0x0001
CONFIGURATION:          id=0x0022, optionFlags=0, start=0x0010, end=0x0010
L3_BOOTLOADER:          id=0x0024, optionFlags=0, start=0x0040, end=0x00d9
KERNEL:                 id=0x0025, optionFlags=0, start=0x0440, end=0x0766
ROOT_FILE_SYSTEM:       id=0x0026, optionFlags=0, start=0x1040, end=0x14ed
FIRMWARE:               id=0x0027, optionflags=0, start=0x2040, end=0x3b33
PARTITION_TABLE:        id=0x0030, optionFlags=0, start=0x0000, end=0x0000
APP_UTILITIES:          id=0x0031, optionFlags=1, start=0x0001, end=0x12cf

2 kilobytes per block


FileSystem type

CONTROL_HEAD:           id=0x0034, optionFlags=0
FILE_CHANGE_LIST:       id=0x0034, optionFlags=0
DATA_IMAGE-numeric-fs:  id=0x0034, optionFlags=0

can read directory contents of stuff under /opt/pcr/rel/data/, acts weird when trying to view outside of that

