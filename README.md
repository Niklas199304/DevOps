## The 1.0 Beta Version will be released in March 2026

## The First 1.0 Alpha Versions are for Test-Users only. If you’d like to become a Test-User and get free access to all apps, please send me a request at [n.bloemen@grizzlar.de](mailto:n.bloemen@grizzlar.de)

# DevOps Lounge ✨🧭

**DevOps Lounge** is the public hub for the DevOps ecosystem: release notes, changelogs, supported versions, app compatibility, and roadmap highlights.
**No core source code** lives here — this repository is intentionally user-facing and partner-facing documentation only.

---

## 🔌 Open App Communication Protocol (IPC)

DevOps is built as a **multi-app platform**: each app runs as an **independent instance** inside a shared **project**, and exchanges data via a **central, local IPC bus**. This enables both internal and third‑party apps to **publish**, **subscribe**, and **process** data in real time (e.g., sensor streams, analysis results, status signals, events).

➡️ **Read the full specification here:**  [DevOps IPC Protocol](https://github.com/GR1ZZL4R/DevOps/blob/main/docs/IPC_Protocol.md)


**Core idea:**

* A **Project** is the container.
* Each **App Instance** is a potential participant on the bus.
* Communication is **local** (e.g., Local Socket / Named Pipe, e.g. via `QLocalSocket`).
* Messages are **not persisted** by the IPC layer.

Apps can (depending on host policy) read data published by other apps, forming an “open per‑instance data space”: what one app publishes can be consumed by others.

---

## Project structure and app instances

Everything is organized per project:

* **Project root:** `Projects/<Project>/`

* **One folder per app instance:**

  * `Projects/<Project>/<App Name #1>/`
  * Example: `Projects/MyProject/DataAnalyzer #1/`

Each instance also has IPC metadata describing how it participates on the bus:

* **Instance IPC metadata:**

  * `Projects/<Project>/<Instance>/ipc/stream.meta.json`

This metadata is the entry point for **discovery**, **debugging**, and **routing**, e.g.:

* “Which streams does this instance publish?”
* “Which payload version does it speak?”
* “Which topics/IDs does it use?”

---

## Central IPC concept: the DevOps Data Node (the bus)

Communication between apps flows through a central **DevOps Data Node** (logically: a “bus”; technically: a local IPC endpoint). This enables:

* **Loose coupling:** apps do not require hard, direct connections to each other.
* **Multi‑producer / multi‑consumer:** many senders and many receivers in parallel.
* **Third‑party integration:** external apps can behave like internal apps as long as they follow structure and protocol.
* **Real‑time focus:** messages are **frames over local IPC**, not file logs.

**Important:** IPC frames are **ephemeral** (not persisted). Persistence/logging is the job of dedicated apps (e.g., Logger/Recorder), not the IPC transport itself.


---

## For new developers: what to remember

* Everything lives in the **project folder**. Each app instance has its own **instance folder**.
* IPC is a **bus**. Apps publish/subscribe via the **central Data Node**.
* Messages are **frames**: local, fast, and **not persisted** by the IPC layer.
* Versioning is controlled by `payload_type`. Currently only **`1`** is valid (DevOps 1.0 Alpha 1).
* **CRC32 is required**. Every message ends with CRC32 over the full frame.
* Packages are **typed**. Data types come from the shared payload legend.

---

## TBD

* **Quick Start for third‑party apps**
* Example `stream.meta.json`
* Minimal “Hello Bus” publish/subscribe flow
* Minimal **MyDevOpsApp** Example


---

## 🎯 Purpose

* Provide **clear release communication** for DevOps Desktop and Apps
* Track **documentation changes** and **compatibility status** over time
* Offer **upgrade guidance** for users and partners
* Keep everything **transparent, stable, and easy to reference**

---

## 📦 What’s inside

* **Release Notes** — high-level summaries per DevOps version
* **Changelog** — documentation / compatibility / roadmap changes
* **Supported Versions** — what’s currently supported and published
* **App Versions Overview** — published app version metadata
* **Compatibility Matrix** — which app versions are verified against which DevOps versions
* **Roadmap Highlights** — short, user-facing roadmap signals

---

## 📰 Release Notes

| Version         | Summary                |
| --------------- | ---------------------- |
| **1.0 Alpha 1** | Test-User Version only |

---

## 🧾 Changelog

Track changes to documentation, compatibility status, and roadmap entries.

| Date | Area          | Change | Link/Ref |
| ---- | ------------- | ------ | -------- |
| TBD  | Docs          | TBD    | TBD      |
| TBD  | Compatibility | TBD    | TBD      |
| TBD  | Roadmap       | TBD    | TBD      |

---

## ✅ Supported Operating System

| Operating System  | Supported Version                |
| ----------------- | -------------------------------- |
| **Windows 10/11** | **1.0 Alpha 1**                  |
| **Linux Ubuntu**  | **1.0 Alpha 1** (not tested yet) |

---

## 🧭 App Versions Overview

**TBD** means the app version metadata has not been published a functional version yet but its in progress. 🔎

> Column header uses a forced line break: **DevOps** (line 1) + **1.0 Alpha 1** (line 2).

| App                  | DevOps_1.0_Alpha_1 | Notes                                                                                              |
| -------------------- | ------------------ | -------------------------------------------------------------------------------------------------- |
| 🏬 App Store         | **1.0 TBD**        | Marketplace and app discovery                                                                      |
| 📶 Bluetooth Monitor | **1.0 Release**    | Device discovery, telemetry readout, basic monitoring                                              |
| 🎥 Cam Journal       | **1.0 Beta**       | Video-based sessions, logging, and analysis workflow entry point                                   |
| ⌨️ Code Editor       | **1.0 Alpha**      | Code-based panels and IO-aware editing                                                             |
| 📈 DataAnalyzer      | **1.0 Beta**       | Plots, filters, and analysis building blocks                                                       |
| 🧪 DataLab           | **1.0 Alpha**      | Panel-based workspace (tables, editors, analyzers)                                                 |
| 🌐 ETH TERMINAL      | **1.0 TBD**        | Ethernet/network tools (planned)                                                                   |
| 🚌 CAN TERMINAL      | **1.0 Alpha**      | CAN 2.0 & CAN-FD source code is based on [SavvyCAN](https://github.com/collin80/SavvyCAN), with custom modifications and additions developed by us. |
| 📡 LoCo-Unit         | **1.0 TBD**        | LoCo-Unit device integration (planned)                                                             |
| 🔤 OCR Video         | **1.0 Alpha**      | OCR extraction from video overlays to CSV                                                          |
| 🧩 pyhbox Interface  | **1.0 Release**    | Sensor app integration and live data bridging (planned)                                            |
| 🔌 SERIAL TERMINAL   | **1.0 Release**    | Serial communication, logging, quick debugging                                                     |
| 📋 Table             | **1.0 Alpha**      | CSV/table panels and basic data inspection                                                         |
| 📶 WiFi Monitor      | **1.0 TBD**        | WiFi device monitoring (planned)                                                                   |

---

## Nice that you’re interested! Here’s a sneak peek :

## Preview @ [www.grizzlar.de/devops](http://www.grizzlar.de/devops)  Password: Preview1234

---

## 🗺️ Roadmap Highlights

| Status | Item        |
| ------ | ----------- |
| 🧷     | In Progress |
| ✅      | Done        |
| 🤝     | Planned     |

| Status | Roadmap                                                                  |
| ------ | ------------------------------------------------------------------------ |
| ✅      | Server -Infrastructur and -Communication, Ready to Scale Up.             |
| ✅      | DevOps App Store.                                                        |
| ✅      | Prototype Testing.                                                       |
| ✅      | Concept Freeze for Secrurity.                                            |
| 🧷     | Testing Alpha Version.                                                   |
| 🧷     | - Lizenz Secrurity.                                                      |
| ✅     | - Creation of a compiler pipeline for deliverable software in C and C++. |
| 🧷     | - Smoke-Tests for internal App Communications (>100 Mbit).               |
| 🧷     | - Ramp up Git-based actions automated DevOps software testing.           |
| 🤝     | Prelauch Alpha Version to Test Users.                                    |
| 🤝     | Collect feedback and take it into account for the Betea version.         |
| 🤝     | Launch Beta Version at March.                                            |

---

## 🧱 Repository rules

* ✅ Documentation, compatibility info, and roadmap highlights are welcome.
* ❌ No proprietary / core DevOps source code.
* ✅ Keep entries short, factual, and versioned.
* ✅ Prefer adding changelog entries when you change tables or statuses.

---

## 📝 How to update

1. Update **App Versions Overview** when an app publishes a new version.
2. Add/adjust **Compatibility Matrix** once results are verified.
3. Add a row to **Changelog** whenever you change compatibility, support windows, or roadmap items.
4. Add a **Release Notes** entry for every new DevOps Desktop release.

---

## 📄 License

DevOps uses a Named-User licensing model. A license is assigned to a single user account and may be used on multiple devices, but only one active session per user is permitted at any time. DevOps itself is provided free of charge. Additional apps are available through the DevOps App Store and may be offered for free, as paid products, or with time-limited trial periods. If a second session is started, the existing session may be signed out automatically.
