# IPC Protocol — DevOps 1.0 

This protocol defines communication over the **central DevOps Data Node** between all apps.
It enables **third‑party apps** to integrate and communicate within the system using a **bus** model.

---

## Foreword — Project structure & transport

* Projects live under: `Projects/<Project>/`
* Each app instance has its own instance folder, e.g.: `Projects/<Project>/<App Name #1>/`
* IPC metadata per instance: `Projects/<Project>/<Instance>/ipc/stream.meta.json`
* IPC data is transported via **local socket frames** (e.g., `QLocalSocket` / Named Pipe).
* IPC is **ephemeral**: messages are **not persisted** by the IPC transport.
* Depending on host policy, each app may read data from other apps (effectively an open data bank per app).

**Source of Truth (SoT):** This document defines the central IPC protocol and policy.

---

## 1. Message header (fixed)

Every message consists of:

1. **Fixed header** (always present)
2. **Payload** (depends on `payload_type`)
3. **CRC32 footer** (always present)

### 1.1 Fixed header layout

| Field          |   Size | Type   | Notes                                       |
| -------------- | -----: | ------ | ------------------------------------------- |
| `id`           | 32 bit | uint32 | Fixed 4‑byte value (constant; not variable) |
| `time_flag`    |  1 bit | bit    | `0` = no timestamp, `1` = timestamp present |
| `timestamp`    | 32 bit | uint32 | Present **only if** `time_flag = 1`         |
| `payload_type` |  8 bit | uint8  | Message payload type (see below)            |

> **Note:** Bit/byte packing rules (alignment) are defined by the implementation and must be consistent across all participants.

---

## 2. Message payload types (message-level)

|   Value | Meaning                                                   |
| ------: | --------------------------------------------------------- |
|     `1` | DevOps 1.0 (currently valid)                              |
| `2–255` | Reserved for later software versions (invalid in Alpha 1) |

---

## 3. Message CRC32 (required)

* **CRC32 is mandatory** and appended at the end of each message.
* CRC32 covers the **complete message**, including `payload_type` and **all payload bytes** (all packages, if any).

---

## 4. Payload Type `1` — Multi‑use container

Payload Type `1` is a container that can hold multiple **packages** inside one frame.
This enables combining different structures in a single message (e.g., timestamp + sensor values + JSON status).

### 4.1 Extended header (payload_type = 1)

| Field                |  Size | Type  | Notes   |
| -------------------- | ----: | ----- | ------- |
| `number_of_packages` | 8 bit | uint8 | `0–255` |

### 4.2 Package layout (payload_type = 1)

Each package consists of:

| Field                |     Size | Type   | Notes                                         |
| -------------------- | -------: | ------ | --------------------------------------------- |
| `id`                 |   32 bit | uint32 | Application-defined package identifier        |
| `payload_type`       |    8 bit | uint8  | **Package Payload Type** (see legend below)   |
| `number_of_payloads` |    8 bit | uint8  | Element count                                 |
| `size_of_payloads`   |    8 bit | uint8  | Bytes per element                             |
| `payload`            | variable | bytes  | `number_of_payloads * size_of_payloads` bytes |

### 4.3 Payload types `2–255`

* **No definition yet → invalid** in this Alpha.

---

## 5. Package Payload Types (legend)

The package payload type is a shared **type legend** for primitives, vectors/matrices, text, serialization formats, media, and special identifiers.

### 5.1 Primitives (1–19)

```text
1  INT8
2  UINT8
3  INT16
4  UINT16
5  INT32
6  UINT32
7  INT64
8  UINT64
9  FLOAT16
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
```

### 5.2 Vectors (20–31)

```text
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
```

### 5.3 Matrices (32–43)

```text
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
```

### 5.4 Structured / serialization formats (44–50)

```text
44 JSON_UTF8
45 BSON
46 CBOR
47 MSGPACK
48 PROTOBUF
49 FLATBUFFERS
50 CAPNPROTO
```

### 5.5 Images (51–63)

```text
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
```

### 5.6 Audio (64–82)

```text
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
```

### 5.7 Time (83–87)

```text
83 TIME_NS
84 TIME_US
85 TIME_MS
86 TIME_S
87 TIME_ISO8601_UTF8
```

### 5.8 Identifiers / network (88–93)

```text
88 UUID_16
89 UUID_32
90 UUID_128
91 MAC_ADDRESS
92 IPV4
93 IPV6
```

### 5.9 Key/value and tabular text (94–96)

```text
94 KEY_VALUE_UTF8
95 CSV_UTF8
96 TSV_UTF8
```

### 5.10 Binary structures / bit packing (97–99)

```text
97 STRUCT_BINARY
98 PACKED_BITS
99 RESERVED
```

### 5.11 Reserved/free ranges

```text
100–199 free (project-specific, sequential)
200–254 free (experimental)
255     RESERVED_INTERNAL
```

---

## Documentation style (short)

* First describe the **Message Header** (header fields, message payload types, CRC).
* Then document each `payload_type` as its own subsection.
* Per `payload_type`: include the **extended header**, **package layout**, and references to the **payload type legend**.
* Always mention CRC32 as a **required message footer**.
