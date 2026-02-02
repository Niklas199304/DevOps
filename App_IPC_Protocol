
# IPC Protocol DevOps 1.0 Alpha 1 

This protocol defines communication over the central DevOps data node between all apps. 
It allows third-party apps to integrate and communicate within the system like a bus.

Foreword (project structure and layout):
- Projects live under Projects/<Project>/
- Each app instance has its own instance folder in the project (e.g. Projects/<Project>/<App Name #1>/)
- IPC metadata per instance: Projects/<Project>/<Instance>/ipc/stream.meta.json
- IPC data is sent over local socket frames (QLocalSocket/Named Pipe) and is not persisted.
- Each app can read data from other apps, effectively an open data bank per app.

Source of Truth (SoT): This file defines the central IPC policy.

1. Message Header Description
    a. Header
    b. Payload Types (Message)
    c. CRC
        - CRC32 is appended at the end of each message.
        - CRC32 covers the complete message including payload_type and all packages.

2. Payload Types Description
    1. Multi Use Cases Payload (payload_type = 1)
        a. Extended Message Header,
            - Number of Packages: 8 bit (0 - 255)
        b. Packages Definition
            - id: 32 bit
            - payload_type: 8 bit (List of Package Payload Types, see legend below)
            - number_of_payloads: 8 bit
            - size_of_payloads: 8 bit (bytes per element)
            - payload: number_of_payloads * size_of_payloads bytes
        c. Packages
        d. CRC

    2. no definition yet ==> invalid
    ...
    255. no definition yet ==> invalid

## IMessage Header (fixed)
- ID: 32 bit (fixed 4-byte value; not variable)
- time_flag: 1 bit (0 = no timestamp, 1 = timestamp present)
- timestamp: 32 bit (uint32, only if time_flag=1)
- payload_type: 8 bit (List of Payload Types)

## Message Payload Types (Message)
1: DevOps 1.0 (currently valid)
2-255: reserved for later software versions

## Message CRC32 (required)
- CRC32 is appended at the end of each message.
- CRC32 covers the complete message including payload_type and all packages.

## Payload Type 1

## Message Header extended (payload_type = 1)
- number_of_packages: 8 bit (number of packages)

## Packages Definition (payload_type = 1)
Per package:
- id: 32 bit
- payload_type: 8 bit (List of Package Payload Types)
- number_of_payloads: 8 bit
- size_of_payloads: 8 bit (bytes per element)
- payload: number_of_payloads * size_of_payloads bytes

## Package Payload Types List

1 INT8
2 UINT8
3 INT16
4 UINT16
5 INT32
6 UINT32
7 INT64
8 UINT64
9 FLOAT16
10 FLOAT32
11 FLOAT64
12 BOOL
13 CHAR_ASCII
14 TEXT_UTF8
15 TEXT_UTF16LE
16 TEXT_UTF16BE
17 TEXT_UTF32LE
18 TEXT_UTF32BE
19 BYTES

20 VECTOR_INT8
21 VECTOR_UINT8
22 VECTOR_INT16
23 VECTOR_UINT16
24 VECTOR_INT32
25 VECTOR_UINT32
26 VECTOR_INT64
27 VECTOR_UINT64
28 VECTOR_FLOAT16
29 VECTOR_FLOAT32
30 VECTOR_FLOAT64
31 VECTOR_BOOL

32 MATRIX_INT8
33 MATRIX_UINT8
34 MATRIX_INT16
35 MATRIX_UINT16
36 MATRIX_INT32
37 MATRIX_UINT32
38 MATRIX_INT64
39 MATRIX_UINT64
40 MATRIX_FLOAT16
41 MATRIX_FLOAT32
42 MATRIX_FLOAT64
43 MATRIX_BOOL

44 JSON_UTF8
45 BSON
46 CBOR
47 MSGPACK
48 PROTOBUF
49 FLATBUFFERS
50 CAPNPROTO

51 IMAGE_RAW_GRAY8
52 IMAGE_RAW_GRAY16
53 IMAGE_RAW_RGB24
54 IMAGE_RAW_BGR24
55 IMAGE_RAW_RGBA32
56 IMAGE_RAW_BGRA32
57 IMAGE_RAW_RGB48
58 IMAGE_RAW_RGBA64
59 IMAGE_JPEG
60 IMAGE_PNG
61 IMAGE_WEBP
62 IMAGE_BMP
63 IMAGE_TIFF

64 AUDIO_PCM_S8
65 AUDIO_PCM_U8
66 AUDIO_PCM_S16LE
67 AUDIO_PCM_S16BE
68 AUDIO_PCM_U16LE
69 AUDIO_PCM_U16BE
70 AUDIO_PCM_S24LE
71 AUDIO_PCM_S24BE
72 AUDIO_PCM_S32LE
73 AUDIO_PCM_S32BE
74 AUDIO_PCM_FLOAT32LE
75 AUDIO_PCM_FLOAT32BE
76 AUDIO_PCM_FLOAT64LE
77 AUDIO_PCM_FLOAT64BE
78 AUDIO_OPUS
79 AUDIO_AAC
80 AUDIO_MP3
81 AUDIO_WAV
82 AUDIO_FLAC

83 TIME_NS
84 TIME_US
85 TIME_MS
86 TIME_S
87 TIME_ISO8601_UTF8

88 UUID_16
89 UUID_32
90 UUID_128
91 MAC_ADDRESS
92 IPV4
93 IPV6

94 KEY_VALUE_UTF8
95 CSV_UTF8
96 TSV_UTF8

97 STRUCT_BINARY
98 PACKED_BITS
99 RESERVED

100–199 free (project-specific, sequential)
200–254 free (experimental)
255 RESERVED_INTERNAL

## Documentation Style (short)
- First describe the Message Header (Header, Payload Types, CRC).
- Then document each payload_type as its own subsection.
- Per payload_type: Extended Header, package layout, and references to the payload type legend.
- Always mention CRC as a required message footer.


