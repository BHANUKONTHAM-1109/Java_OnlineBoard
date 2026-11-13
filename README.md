# 🎨 Java Online Whiteboard

> A lightweight, real-time collaborative desktop drawing application built in pure Java using **Java Swing GUI**, **TCP Socket Programming**, and **Multithreading**.

[![Java Version](https://img.shields.io/badge/Java-8%2B%20%7C%2017%20%7C%2021%20%7C%2025-orange.svg)](https://www.oracle.com/java/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Architecture](https://img.shields.io/badge/Architecture-Client--Server-brightgreen.svg)]()
[![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)]()

---

## 📑 Table of Contents

- [📌 Project Overview](#-project-overview)
- [✨ Key Features](#-key-features)
- [🛠️ Tech Stack & Concepts](#️-tech-stack--concepts)
- [🏗️ System Architecture & Data Flow](#️-system-architecture--data-flow)
- [📂 Codebase Structure](#-codebase-structure)
- [🖥️ UI Controls & Toolbar](#️-ui-controls--toolbar)
- [▶️ Step-by-Step Setup & Execution](#️-step-by-step-setup--execution)
- [🌐 Multi-Computer LAN Setup](#-multi-computer-lan-setup)
- [❓ Frequently Asked Questions & Troubleshooting](#-frequently-asked-questions--troubleshooting)
- [🎯 Technical Highlights (For Interviews & Viva)](#-technical-highlights-for-interviews--viva)
- [🔮 Planned Enhancements](#-planned-enhancements)
- [👨‍💻 Author & License](#-author--license)

---

## 📌 Project Overview

**Java Online Whiteboard** is a distributed drawing application where multiple participants can sketch, take notes, and collaborate on a shared canvas in real time. 

Unlike heavy web-based collaboration tools (like Miro or Figma) that require browsers, cloud accounts, and internet access, this application:
- Operates on a **lightweight, self-hosted client-server architecture**.
- Runs completely **offline over Local Area Networks (LAN) or Wi-Fi**.
- Requires **zero external dependencies or third-party libraries** — it runs on standard Java SE.

---

## ✨ Key Features

- **⚡ Real-Time Synchronization:** Sub-millisecond stroke delivery across all active client windows.
- **👥 Multi-Client Collaboration:** Supports multiple simultaneous clients connecting to a single central server.
- **🎨 Full RGB Color Palette:** Native color picker dialog (`JColorChooser`) for custom colors.
- **🧽 Dedicated Eraser:** One-click toggle to erase content without losing selected canvas dimensions.
- **📏 Dynamic Stroke Sizing:** Interactive slider to adjust brush thickness from `1px` up to `20px`.
- **🗑️ Synchronized Canvas Reset:** One-click clear button that clears the whiteboard across all connected peers.
- **📶 LAN & Localhost Ready:** Connect between multiple terminal windows on the same PC or across distinct laptops on the same Wi-Fi router.

---

## 🛠️ Tech Stack & Concepts

| Technology / Concept | Implementation in Project |
|---|---|
| **Language** | Java (JDK 8 or higher, fully tested on JDK 25) |
| **User Interface** | Java Swing (`JFrame`, `JPanel`, `JToolBar`, `JSlider`, `JButton`) |
| **Graphics Engine** | Java AWT (`Graphics2D`, `BasicStroke`, `Color`) |
| **Networking** | Java Socket API (`ServerSocket`, `Socket` over TCP port 5000) |
| **Concurrency** | Java Multithreading (`Thread`, Runnable lambdas, synchronization locks) |
| **Data Transmission** | Java Object Serialization (`ObjectOutputStream`, `ObjectInputStream`) |

---

## 🏗️ System Architecture & Data Flow

The project follows a centralized **Star Network Topology** using a Publish-Subscribe broadcast model:

```text
                  ┌─────────────────────────────────────────┐
                  │       WhiteboardServer (Port 5000)      │
                  │   - ServerSocket daemon loop            │
                  │   - Synchronized List<ObjectOutputStream>│
                  │   - Broadcast engine                    │
                  └────────────────────┬────────────────────┘
                                       │
            ┌──────────────────────────┼──────────────────────────┐
            │                          │                          │
            ▼                          ▼                          ▼
     ┌──────────────┐           ┌──────────────┐           ┌──────────────┐
     │   Client 1   │           │   Client 2   │           │   Client 3   │
     │ WhiteboardGUI│           │ WhiteboardGUI│           │ WhiteboardGUI│
     └──────────────┘           └──────────────┘           └──────────────┘
```

### Detailed Event Lifecycle

```text
User draws with mouse on Client 1
       │
       ▼
1. DrawingPanel.mouseDragged(MouseEvent) captures (prevX, prevY) -> (x, y)
       │
       ▼
2. Encapsulated into Message object (x1, y1, x2, y2, color, size, clear=false)
       │
       ▼
3. Local DrawingPanel adds Message to lines list & triggers repaint()
       │
       ▼
4. Serialized via ObjectOutputStream over TCP Socket to Port 5000
       │
       ▼
5. WhiteboardServer worker thread receives Message via ObjectInputStream
       │
       ▼
6. Server iterates over synchronized client pool and forwards Message
       │
       ▼
7. Client 2 & Client 3 background listener threads read Message
       │
       ▼
8. Remote lines added to respective DrawingPanel & repaint() renders strokes
```

---

## 📂 Codebase Structure

The project has been cleaned and organized into a standard Java layout:

```text
Java_OnlineBoard/
├── src/
│   ├── Message.java             # Network data transfer object (DTO)
│   ├── WhiteboardClient.java    # Swing GUI interface & client network handler
│   └── WhiteboardServer.java    # Multithreaded TCP broadcast server
├── .gitignore                   # Ignores build binaries (.class, bin/)
├── LICENSE                      # MIT Open Source License
└── README.md                    # Project documentation & run guide
```

### Component Details

1. **[`Message.java`]**
   - Implements `java.io.Serializable`.
   - Stores stroke start coordinates `(x1, y1)`, end coordinates `(x2, y2)`, `color`, `size`, and a `clear` flag.
2. **[`WhiteboardServer.java`]**
   - Listens on TCP port `5000`.
   - Maintains a thread-safe list of active client output streams (`synchronized(clients)`).
   - Spawns a dedicated worker thread for each client to handle incoming packets and broadcast them to all connected peers.
3. **[`WhiteboardClient.java`]**
   - Prompts for server address (`JOptionPane`).
   - Renders a 900x600 interactive canvas window.
   - Contains a background daemon thread that continuously listens for incoming network messages and refreshes the canvas.

---

## 🖥️ UI Controls & Toolbar

| Control | Type | Functionality |
|---|---|---|
| **🎨 Color** | Button | Opens the system `JColorChooser` dialog to pick any custom RGB/HEX color. |
| **🧽 Eraser** | Button | Sets active drawing color to `Color.WHITE` to erase existing strokes. |
| **🗑️ Clear** | Button | Emits a reset packet (`clear = true`), wiping the canvas for all users instantly. |
| **📏 Brush Size**| Slider (`1–20`) | Adjusts `BasicStroke` line thickness in pixels (default is `3px`). |
| **🖱️ Canvas** | Drawing Area | Left-click and drag to draw smooth continuous freehand vector lines. |

---

## ▶️ Step-by-Step Setup & Execution

### Prerequisites
- **Java JDK 8 or newer** installed (JDK 11, 17, 21, or 25 recommended).
- Check your installation by opening a terminal and typing:
  ```powershell
  javac -version
  java -version
  ```

---

### 1. Compile the Code
Open your terminal in the project root directory and compile the source files into a `bin/` output folder:

```powershell
javac -d bin src/*.java
```

---

### 2. Launch the Server
Start the central server first:

```powershell
java -cp bin WhiteboardServer
```

**Expected terminal output:**
```text
🎨 Whiteboard Server running on port 5000
```
> Keep this terminal running in the background.

---

### 3. Launch Client 1
Open a **new terminal tab or window** and execute:

```powershell
java -cp bin WhiteboardClient
```

1. An input dialog will prompt: **"Enter Server IP (use localhost for same PC):"**.
2. Type `localhost` and press **OK**.
3. The Whiteboard window will open immediately.

---

### 4. Launch Client 2 (Testing Real-Time Sync)
Open another terminal tab and launch a second client:

```powershell
java -cp bin WhiteboardClient
```
1. Type `localhost` and press **OK**.
2. Arrange both client windows side-by-side.
3. Draw in either window — your drawing will instantly mirror onto the other canvas!

---

## 🌐 Multi-Computer LAN Setup

To use the whiteboard across two or more computers connected to the same Wi-Fi / LAN network:

1. **Find the Server PC's Local IP Address:**
   - On Windows: Run `ipconfig` in Command Prompt / PowerShell. Look for **IPv4 Address** (e.g., `192.168.1.15`).
   - On macOS/Linux: Run `ifconfig` or `ip a` (e.g., `192.168.1.20`).
2. **Start Server on the Host PC:**
   ```powershell
   java -cp bin WhiteboardServer
   ```
3. **Allow Port 5000 in Firewall:**
   - Ensure the host's Windows Defender Firewall allows incoming connections on TCP Port `5000`.
4. **Connect from Client PCs:**
   - Run `java -cp bin WhiteboardClient` on the other computers.
   - Enter the **Host's IPv4 address** (e.g., `192.168.1.15`) instead of `localhost`.

---

## 👨‍💻 Author & License

* **Author:** Bhanu K
* **License:** This project is licensed under the [MIT License](LICENSE) — free to use, modify, and distribute for educational and commercial purposes.
