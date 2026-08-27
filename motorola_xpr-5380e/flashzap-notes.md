fudally_usb.dll

`HANDLE __cdecl FUN_10001450(HDEVINFO param_1,PSP_DEVICE_INTERFACE_DATA param_2,int param_3)`

`0562FB00  02F31F24  "\\\\?\\usb#vid_0cad&pid_1000#5&12c8f4c0&0&2#{cfeddc1a-a89e-46dc-afba-b2caa907ef28}"`




```
FZAPworker::writeSrec()
        |
        v
FZAPworker::writeMemory()
        |
        v
SBEPobject::writePacket()
        |
        v
commPort virtual write()
        |
        v
USBPort::writePort()
        |
        v
FudallyUSBDevice::write()
        |
        v
fudallyUSB_write()
        |
        v
Windows WriteFile()
        |
        |  HANDLE
        v
fudally.sys
        |
        v
Windows USB subsystem / USBD.sys
        |
        v
USB device
```






```
SREC file
  │
  ▼
SrecParser
  │
  ▼
dataObject
  │
  ▼
FZAPworker::writeSrec
  │
  ▼
FZAPworker::writeMemory
  │
  ▼
transferData
  │
  ▼
SBEPobject::writePacket
  │
  ▼
USBPort::writePort
  │
  ▼
FudallyUSBDevice::write
  │
  ▼
fudallyUSB_write
  │
  ▼
WriteFile(...)
  │
  ▼
Windows device handle
  │
  ▼
USB driver / USB stack
  │
  ▼
USB device
```





```
CreateFileA(local_200,
            0xc0000000,
            3,
            0,
            3,
            0xc0000000,
            0);
```



0xc0000000 = GENERIC_READ | GENERIC_WRITE
3 = FILE_SHARE_READ | FILE_SHARE_WRITE
3 = OPEN_EXISTING


```
enumerate device interfaces
        ↓
for GUID {CFEDDC1A-A89E-46DC-AFBA-B2CAA907EF28}
        ↓
get DevicePath
        ↓
return/open that DevicePath
```







```
SREC data
   ↓
SBEP framing
   ↓
USBPort
   ↓
fudally_usb.dll
   ↓
CreateFile() on a fudally.sys device interface
   ↓
overlapped ReadFile()/WriteFile()
   ↓
fudally.sys
   ↓
USB request / IRP
   ↓
USBD.sys / Windows USB stack
   ↓
vendor-specific USB interface
   ↓
radio bootloader
```



```
             SREC/XML parser
                    │
                    ▼
                SrecRep
                    │
                    ▼
               dataObject
                    │
                    ▼
              writeSrec()
                    │
                    ▼
              writeMemory()
                    │
          ┌─────────┴─────────┐
          │                   │
      partition             other
       types                 type
    1/2/3/6                  ?
          │                   │
          ▼                   ▼
 Request_Access()       raw SBEP packet
 File_Access()               │
          │                   │
          ▼                   ▼
      transferData()     SBEPobject
          │                   │
          └─────────┬─────────┘
                    ▼
              bootloader
```



```
                    SREC file
                       │
                       ▼
                 parse S-record
                       │
                       ▼
             ┌── contiguous? ──┐
             │                 │
            yes                no
             │                 │
             ▼                 ▼
       append bytes       finalize current
             │             dataObject
             │                 │
             │                 ▼
             │          start new object
             │
             └──────────┐
                        ▼
                  finalize at EOF
                        │
                        ▼
                  SrecRep linked
                  list of segments
```


```
struct dataObject {
    void *vftable;          // 0x00

    uint32_t size;          // 0x04
    uint32_t startId;       // 0x08
    uint32_t endId;         // 0x0c

    uint8_t  type;          // 0x10
    uint8_t  reserved11;    // 0x11
    uint16_t id;             // 0x12

    uint32_t optionFlags;   // 0x14

    char    *path;           // 0x18
    uint8_t *data;           // 0x1c
    dataObject *next;        // 0x20
};
```

```
enum DataObjectType {
    TYPE_SREC       = 0,
    TYPE_RAW_NAND   = 1,
    TYPE_FTL        = 2,
    TYPE_FILESYSTEM = 3,
    TYPE_OTP        = 6,
    TYPE_UNKNOWN    = 0xff
};
```

```
struct SrecRep {
    // unknown 0x00-0x0b

    uint32_t numObjects;     // 0x0c

    // unknown 0x10-0x17

    dataObject *first;       // 0x18
    dataObject *last;        // 0x1c

    // unknown 0x20-0x23

    uint32_t totalBytes;     // 0x24
};
```





```
                         parseFile(path, validate)
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
               SREC text                    XML container
                    │                           │
              S0/S1/S2/S3/S7              root/Partition
                    │                           │
                    │                     Type / attributes
                    │                           │
                    │                      hex Payload
                    │                           │
                    ▼                           ▼
             binary data buffer          binary data buffer
                    │                           │
                    └─────────────┬─────────────┘
                                  │
                                  ▼
                            dataObject
                                  │
                   ┌──────────────┼──────────────┐
                   │              │              │
                address          type          size
                   │              │              │
                   └──────────────┼──────────────┘
                                  │
                                  ▼
                             SrecRep list
                                  │
                 ┌────────────────┼───────────────┐
                 ▼                ▼               ▼
             dataObject       dataObject      dataObject
```





