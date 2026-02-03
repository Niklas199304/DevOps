# IPC Protocol

This protocol defines communication over the **central DevOps Data Node** between all apps.
It allows **third-party apps** to integrate and communicate within the system like a **bus**.

---

## Foreword — Project structure & transport

* Projects live under: `Projects/<Project>/`
* Each app instance has its own instance folder in the project, e.g.: `Projects/<Project>/<App Name #1>/`
* IPC metadata per instance: `Projects/<Project>/<Instance>/ipc/stream.meta.json`
* IPC data is sent over **local socket frames** (e.g., `QLocalSocket` / Named Pipe) and is **not persisted**.
* Each app can read data from other apps, effectively an open data bank per app.

**Source of Truth (SoT):** This file defines the central IPC policy.

---

## Naming convention (important)

To keep everything unambiguous, this protocol uses these terms strictly:

* **Message**: the full transport frame (**Header + Container + CRC**)
* **Header**: fixed message header
* **Container**: the message body (selected by `container_type`)
* **Payload**: the bytes inside a container item (typed via `payload_type`)

There are two different type fields:

1. **`container_type`** (message-level, in the header)
   Selects the **container format** that follows in the message.

2. **`payload_type`** (container-level, inside a container item)
   Describes **how to interpret the payload bytes** using the shared **Payload Types List** (legend).

---

## 1. Message description

Every message consists of:

1. **Header** (fixed)
2. **Container** (depends on `container_type`)
3. **CRC32** (required footer)

### 1.1 Header (fixed)  



| Field            |   Size | Type   | Notes                                                               |
| ---------------- | -----: | ------ | ------------------------------------------------------------------- |
| `id`             | 32bit | uint32 | Deterministic CRC32 of the instance ASCII/UTF-8 name; stable while the name is unchanged |
| `time_flag`      |  1bit | bit    | `0` = no timestamp, `1` = timestamp present                         |
| `timestamp`      | 32bit | uint32 | Present **only if** `time_flag = 1`                                 |
| `container_type` |  8bit | uint8  | Container selector (see Section 1.2)                                |

> **Packing rule:** Bit/byte alignment and bit order must be consistent across all participants.

### 1.2 Container types (`container_type`)

| Value   | Meaning                                          |
| ------- | ------------------------------------------------ |
| `1`     | Multi Use Cases Container (defined in Section 2) |
| `2–244` | No definition yet → **invalid** in Alpha 1       |
| `255`   | INTERNAL                                         |

### 1.3 CRC32 (required)

* **CRC32 is appended at the end of each message**.
* CRC32 covers the **complete message**, including `container_type` and all container bytes.

---

## 2. Container description

### 2.1 `container_type = 1` — Multi Use Cases Container

This container type can hold multiple **container items** inside one message.

#### 2.1.1 Extended header (`container_type = 1`)

| Field             |  Size | Type  | Notes   |
| ----------------- | ----: | ----- | ------- |
| `number_of_items` | 8bit | uint8 | `0–255` |

#### 2.1.2 Container item definition (repeated `number_of_items` times)

Each container item consists of:

| Field                 |     Size | Type   | Notes                                              |
| --------------------- | -------: | ------ | -------------------------------------------------- |
| `id`                  |   32bit | uint32 | Application-defined container identifier           |
| `payload_type`        |    8bit | uint8  | **Payload Types List** value (legend in Section 3) |
| (`number_of_payload`) |    8bit | uint8  | Depends on `payload_type`                          |
| (`size_of_payload`)   |    8bit | uint8  | Depends on `payload_type`                          |
| `payload`             | variable | bytes  | `number_of_payload * size_of_payload` bytes        |

> **CRC:** The message-level CRC32 (Section 1.3) protects the complete container and all container items.

### 2.2 `container_type = 2–244`

* No definition yet → **invalid** in this Alpha.

### 2.3 `container_type = 255` — INTERNAL

* Reserved for internal use.

---

## 2.4 Detailed example — One IPC message with CAN 2.0 + CAN FD + strings + metadata (`container_type = 1`)

Below is a fully worked example of a **single Message** with:

* **1× CAN 2.0 frame** (`payload_type = 1  MESSAGE_CAN_2_0`)
* **1× CAN FD frame** (`payload_type = 2  MESSAGE_CAN_FD`)
* **1× TEXT** (`payload_type = 64 TEXT_UTF8`)
* **1× metadata JSON** (`payload_type = 94 JSON_UTF8`)

> **Endian note (example only):** The byte examples below assume **little-endian** for all multi-byte integers.
>
> **Packing note (example only):** The spec defines `time_flag` as **1 bit**. For readability, this example encodes it as a **1-byte flags field** (`0x01` = timestamp present). Implementations may pack bits differently, but must do so **consistently across all participants**.

### 2.4.1 Message header (fixed)

**ID convention (required):** `id` is derived from the app instance name using CRC32 over UTF-8 bytes:

* `id = CRC32(UTF8("CAN Terminal #1"))` → `0xCBE33DCD`
* encoded as `uint32` (little-endian in this example): `CD 3D E3 CB`

| Field            | Value (human)                           | Encoded (hex, little-endian) |
| ---------------- | --------------------------------------- | ---------------------------- |
| `id`             | `CRC32("CAN Terminal #1") = 0xCBE33DCD` | `CD 3D E3 CB`                |
| `time_flag`      | `1` (timestamp present)                 | `01`                         |
| `timestamp`      | `0x000F4240` (1,000,000)                | `40 42 0F 00`                |
| `container_type` | `1` (Multi Use Cases Container)         | `01`                         |

**Header bytes (example):**

```text
CD 3D E3 CB  01  40 42 0F 00  01
|-- id ---|   tf   |timestamp|  ct
```

### 2.4.2 Container header (`container_type = 1`)

| Field             | Value | Encoded (hex) |
| ----------------- | ----: | ------------- |
| `number_of_items` |   `4` | `04`          |

So the container begins with:

```text
04
```

### 2.4.3 Container items (4 items)

Each item:

* `id` (uint32)
* `payload_type` (uint8)
* (`number_of_payload`) (uint8) — **Depends on `payload_type`**
* (`size_of_payload`) (uint8) — **Depends on `payload_type`**
* `payload` — `number_of_payload * size_of_payload` bytes

---

#### Item 1 — CAN 2.0 frame (`payload_type = 1  MESSAGE_CAN_2_0`)

**Goal:** Transport exactly one CAN 2.0 frame.

**Item header:**

| Field                 | Value (human)         | Encoded (hex) |
| --------------------- | --------------------- | ------------- |
| `id`                  | `0x00000011`          | `11 00 00 00` |
| `payload_type`        | `1 (MESSAGE_CAN_2_0)` | `01`          |
| (`number_of_payload`) | `1`                   | `01`          |
| (`size_of_payload`)   | `16` bytes            | `10`          |

**Payload encoding for one CAN 2.0 frame (example struct, 16 bytes):**

| Offset | Field      | Size | Example value             | Hex (little-endian where applicable) |
| -----: | ---------- | ---: | ------------------------- | ------------------------------------ |
|      0 | `can_id`   |    4 | `0x00000123`              | `23 01 00 00`                        |
|      4 | `dlc`      |    1 | `8`                       | `08`                                 |
|      5 | `flags`    |    1 | `0x00`                    | `00`                                 |
|      6 | `reserved` |    2 | `0x0000`                  | `00 00`                              |
|      8 | `data[8]`  |    8 | `DE AD BE EF 01 02 03 04` | `DE AD BE EF 01 02 03 04`            |

So the **payload (16 bytes)** is:

```text
23 01 00 00  08 00 00 00  DE AD BE EF 01 02 03 04
```

Full **Item 1 bytes**:

```text
11 00 00 00  01  01 10  23 01 00 00 08 00 00 00 DE AD BE EF 01 02 03 04
|--id---|  pt  np sz   |----------- payload (16) ------------------------|
```

---

#### Item 2 — CAN FD frame (`payload_type = 2  MESSAGE_CAN_FD`)

**Goal:** Transport exactly one CAN FD frame (example uses 32 bytes of data).

**Item header:**

| Field                 | Value (human)        | Encoded (hex) |
| --------------------- | -------------------- | ------------- |
| `id`                  | `0x00000012`         | `12 00 00 00` |
| `payload_type`        | `2 (MESSAGE_CAN_FD)` | `02`          |
| (`number_of_payload`) | `1`                  | `01`          |
| (`size_of_payload`)   | `40` bytes           | `28`          |

**Payload encoding for one CAN FD frame (example struct, 40 bytes):**

| Offset | Field       | Size | Example value            | Hex                                                                                               |
| -----: | ----------- | ---: | ------------------------ | ------------------------------------------------------------------------------------------------- |
|      0 | `can_id`    |    4 | `0x1ABCDEFF`             | `FF DE BC 1A`                                                                                     |
|      4 | `dlc`       |    1 | `13` (FD DLC→32 bytes)   | `0D`                                                                                              |
|      5 | `flags`     |    1 | `0x03` (example: FD+BRS) | `03`                                                                                              |
|      6 | `reserved`  |    2 | `0x0000`                 | `00 00`                                                                                           |
|      8 | `data_len`  |    1 | `32`                     | `20`                                                                                              |
|      9 | `reserved2` |    7 | zeros                    | `00 00 00 00 00 00 00`                                                                            |
|     16 | `data[32]`  |   32 | `00..1F`                 | `00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 10 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F` |

So the **payload (40 bytes)** is:

```text
FF DE BC 1A  0D 03 00 00  20 00 00 00 00 00 00 00
00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
10 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F
```

Full **Item 2 bytes**:

```text
12 00 00 00  02  01 28  <40 bytes payload shown above>
```

---

#### Item 3 — TEXT (`payload_type = 64  TEXT_UTF8`)

**Goal:** Transport one UTF-8 string.

Example string:

```text
"Ride mode = SPORT"
```

Length = 17 bytes.

**Item header:**

| Field                 | Value (human)    | Encoded (hex) |
| --------------------- | ---------------- | ------------- |
| `id`                  | `0x00000020`     | `20 00 00 00` |
| `payload_type`        | `64 (TEXT_UTF8)` | `40`          |
| (`number_of_payload`) | `17`             | `11`          |
| (`size_of_payload`)   | `1`              | `01`          |

**Payload bytes (UTF-8):**

```text
52 69 64 65 20 6D 6F 64 65 20 3D 20 53 50 4F 52 54
R  i  d  e     m  o  d  e     =     S  P  O  R  T
```

Full **Item 3 bytes**:

```text
20 00 00 00  40  11 01  52 69 64 65 20 6D 6F 64 65 20 3D 20 53 50 4F 52 54
```

---

#### Item 4 — Metadata JSON (`payload_type = 94  JSON_UTF8`)

**Goal:** Attach metadata as JSON (UTF-8 bytes).

Example JSON (minified):

```json
{"source":"loco-unit","bus":"CAN1","topic":"telemetry","session":"2026-02-03T12:34:56Z","note":"example"}
```

Let `N = len(json_utf8_bytes)`.

**Item header:**

| Field                 | Value            | Notes               |
| --------------------- | ---------------- | ------------------- |
| `id`                  | `0x00000021`     | metadata item       |
| `payload_type`        | `94 (JSON_UTF8)` | JSON_UTF8           |
| (`number_of_payload`) | `N`              | byte length of JSON |
| (`size_of_payload`)   | `1`              | bytes               |

> Encoding rule here is the same as TEXT: `number_of_payload = number of bytes`, `size_of_payload = 1`.

---

### 2.4.4 Full message composition (high level)

```text
[Header]
  id (4) + time_flag (1*) + timestamp (4) + container_type (1)
[Container]
  number_of_items (1)
  Item 1 (CAN 2.0)
  Item 2 (CAN FD)
  Item 3 (TEXT)
  Item 4 (JSON metadata)
[CRC32]
  crc32 (4)
```

### 2.4.5 Example hex dump (CRC omitted here)

Below is a *partial* hex dump that includes Header, container header, and the complete Item 1 + Item 3.
(Item 2 and Item 4 are shown above and can be appended directly.)

```text
CD 3D E3 CB 01 40 42 0F 00 01   04
11 00 00 00 01 01 10 23 01 00 00 08 00 00 00 DE AD BE EF 01 02 03 04
20 00 00 00 40 11 01 52 69 64 65 20 6D 6F 64 65 20 3D 20 53 50 4F 52 54
... (Item 2 CAN FD bytes)
... (Item 4 JSON bytes)
<CRC32 4 bytes>
```

> **CRC32:** The final CRC32 is computed over **all bytes from the start of `id` through the end of the container**, and then appended as 4 bytes.

---

## 3. Payload Types List (legend)

This legend is used by **`payload_type`** inside container items (Section 2.1.2).

### 3.1 Protocol / message payloads (1–50)

```text
1 MESSAGE_CAN_2_0
2 MESSAGE_CAN_FD
3 MESSAGE_MODBUS
4 MESSAGE_PLACEHOLDER_04
5 MESSAGE_PLACEHOLDER_05
6 MESSAGE_PLACEHOLDER_06
7 MESSAGE_PLACEHOLDER_07
8 MESSAGE_PLACEHOLDER_08
9 MESSAGE_PLACEHOLDER_09
10 MESSAGE_PLACEHOLDER_10
11 MESSAGE_PLACEHOLDER_11
12 MESSAGE_PLACEHOLDER_12
13 MESSAGE_PLACEHOLDER_13
14 MESSAGE_PLACEHOLDER_14
15 MESSAGE_PLACEHOLDER_15
16 MESSAGE_PLACEHOLDER_16
17 MESSAGE_PLACEHOLDER_17
18 MESSAGE_PLACEHOLDER_18
19 MESSAGE_PLACEHOLDER_19
20 MESSAGE_PLACEHOLDER_20
21 MESSAGE_PLACEHOLDER_21
22 MESSAGE_PLACEHOLDER_22
23 MESSAGE_PLACEHOLDER_23
24 MESSAGE_PLACEHOLDER_24
25 MESSAGE_PLACEHOLDER_25
26 MESSAGE_PLACEHOLDER_26
27 MESSAGE_PLACEHOLDER_27
28 MESSAGE_PLACEHOLDER_28
29 MESSAGE_PLACEHOLDER_29
30 MESSAGE_PLACEHOLDER_30
31 MESSAGE_PLACEHOLDER_31
32 MESSAGE_PLACEHOLDER_32
33 MESSAGE_PLACEHOLDER_33
34 MESSAGE_PLACEHOLDER_34
35 MESSAGE_PLACEHOLDER_35
36 MESSAGE_PLACEHOLDER_36
37 MESSAGE_PLACEHOLDER_37
38 MESSAGE_PLACEHOLDER_38
39 MESSAGE_PLACEHOLDER_39
40 MESSAGE_PLACEHOLDER_40
41 MESSAGE_PLACEHOLDER_41
42 MESSAGE_PLACEHOLDER_42
43 MESSAGE_PLACEHOLDER_43
44 MESSAGE_PLACEHOLDER_44
45 MESSAGE_PLACEHOLDER_45
46 MESSAGE_PLACEHOLDER_46
47 MESSAGE_PLACEHOLDER_47
48 MESSAGE_PLACEHOLDER_48
49 MESSAGE_PLACEHOLDER_49
50 MESSAGE_PLACEHOLDER_50
```

### 3.2 Primitives (51–69)

```text
51 INT8
52 UINT8
53 INT16
54 UINT16
55 INT32
56 UINT32
57 INT64
58 UINT64
59 FLOAT16
60 FLOAT32
61 FLOAT64
62 BOOL
63 CHAR_ASCII
64 TEXT_UTF8
65 TEXT_UTF16LE
66 TEXT_UTF16BE
67 TEXT_UTF32LE
68 TEXT_UTF32BE
69 BYTES
```

### 3.3 Vectors (70–81)

```text
70 VECTOR_INT8
71 VECTOR_UINT8
72 VECTOR_INT16
73 VECTOR_UINT16
74 VECTOR_INT32
75 VECTOR_UINT32
76 VECTOR_INT64
77 VECTOR_UINT64
78 VECTOR_FLOAT16
79 VECTOR_FLOAT32
80 VECTOR_FLOAT64
81 VECTOR_BOOL
```

### 3.4 Matrices (82–93)

```text
82 MATRIX_INT8
83 MATRIX_UINT8
84 MATRIX_INT16
85 MATRIX_UINT16
86 MATRIX_INT32
87 MATRIX_UINT32
88 MATRIX_INT64
89 MATRIX_UINT64
90 MATRIX_FLOAT16
91 MATRIX_FLOAT32
92 MATRIX_FLOAT64
93 MATRIX_BOOL
```

### 3.5 Structured / serialization formats (94–100)

```text
94 JSON_UTF8
95 BSON
96 CBOR
97 MSGPACK
98 PROTOBUF
99 FLATBUFFERS
100 CAPNPROTO
```

### 3.6 Images (101–113)

```text
101 IMAGE_RAW_GRAY8
102 IMAGE_RAW_GRAY16
103 IMAGE_RAW_RGB24
104 IMAGE_RAW_BGR24
105 IMAGE_RAW_RGBA32
106 IMAGE_RAW_BGRA32
107 IMAGE_RAW_RGB48
108 IMAGE_RAW_RGBA64
109 IMAGE_JPEG
110 IMAGE_PNG
111 IMAGE_WEBP
112 IMAGE_BMP
113 IMAGE_TIFF
```

### 3.7 Audio (114–132)

```text
114 AUDIO_PCM_S8
115 AUDIO_PCM_U8
116 AUDIO_PCM_S16LE
117 AUDIO_PCM_S16BE
118 AUDIO_PCM_U16LE
119 AUDIO_PCM_U16BE
120 AUDIO_PCM_S24LE
121 AUDIO_PCM_S24BE
122 AUDIO_PCM_S32LE
123 AUDIO_PCM_S32BE
124 AUDIO_PCM_FLOAT32LE
125 AUDIO_PCM_FLOAT32BE
126 AUDIO_PCM_FLOAT64LE
127 AUDIO_PCM_FLOAT64BE
128 AUDIO_OPUS
129 AUDIO_AAC
130 AUDIO_MP3
131 AUDIO_WAV
132 AUDIO_FLAC
```

### 3.8 Time (133–137)

```text
133 TIME_NS
134 TIME_US
135 TIME_MS
136 TIME_S
137 TIME_ISO8601_UTF8
```

### 3.9 Identifiers / network (138–143)

```text
138 UUID_16
139 UUID_32
140 UUID_128
141 MAC_ADDRESS
142 IPV4
143 IPV6
```

### 3.10 Key/value and tabular text (144–146)

```text
144 KEY_VALUE_UTF8
145 CSV_UTF8
146 TSV_UTF8
```

### 3.11 Binary structures / bit packing (147–149)

```text
147 STRUCT_BINARY
148 PACKED_BITS
149 RESERVED
```

### 3.12 Reserved/free ranges

```text
150–199 free (project-specific, sequential)
200–254 free (experimental)
255 RESERVED_INTERNAL
```

---

## Documentation style (short)

* First describe the **Message Header** (Header fields, Container Types, CRC).
* Then document each `container_type` as its own subsection.
* Per `container_type`: include the **extended header**, **container item layout**, and references to the **Payload Types List**.
* Always mention CRC32 as a **required message footer**.
