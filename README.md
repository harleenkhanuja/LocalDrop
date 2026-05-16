<div align="center">

<br/>

<pre>
██╗      ██████╗  ██████╗ █████╗ ██╗     ██████╗ ██████╗  ██████╗ ██████╗ 
██║     ██╔═══██╗██╔════╝██╔══██╗██║     ██╔══██╗██╔══██╗██╔═══██╗██╔══██╗
██║     ██║   ██║██║     ███████║██║     ██║  ██║██████╔╝██║   ██║██████╔╝
██║     ██║   ██║██║     ██╔══██║██║     ██║  ██║██╔══██╗██║   ██║██╔═══╝ 
███████╗╚██████╔╝╚██████╗██║  ██║███████╗██████╔╝██║  ██║╚██████╔╝██║     
╚══════╝ ╚═════╝  ╚═════╝╚═╝  ╚═╝╚══════╝╚═════╝ ╚═╝  ╚═╝ ╚═════╝ ╚═╝     
</pre>

### ✦ Browser-based LAN file sharing — no client software, no cloud, no limits ✦

<br/>

[![Java](https://img.shields.io/badge/Java_17-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://www.java.com)
[![Jakarta EE](https://img.shields.io/badge/Jakarta_EE_6.0-007396?style=for-the-badge&logo=jakarta&logoColor=white)](https://jakarta.ee)
[![JSP](https://img.shields.io/badge/JSP_3.1-4A90D9?style=for-the-badge&logo=java&logoColor=white)](https://jakarta.ee/specifications/pages/)
[![MySQL](https://img.shields.io/badge/MySQL_8-4479A1?style=for-the-badge&logo=mysql&logoColor=white)](https://www.mysql.com)
[![HikariCP](https://img.shields.io/badge/HikariCP_5.1-00ACC1?style=for-the-badge&logo=java&logoColor=white)](https://github.com/brettwooldridge/HikariCP)
[![Maven](https://img.shields.io/badge/Maven_3-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)](https://maven.apache.org)
[![Tomcat](https://img.shields.io/badge/Tomcat_10.1-F8DC75?style=for-the-badge&logo=apachetomcat&logoColor=black)](https://tomcat.apache.org)
[![JSTL](https://img.shields.io/badge/JSTL_3.0-6DB33F?style=for-the-badge&logo=java&logoColor=white)](https://jakarta.ee/specifications/tags/)
[![Logback](https://img.shields.io/badge/Logback_SLF4J-A9225C?style=for-the-badge&logo=java&logoColor=white)](https://logback.qos.ch)
[![JDBC](https://img.shields.io/badge/JDBC-003545?style=for-the-badge&logo=java&logoColor=white)](#)

<br/>

> **Any device. Any file. Same Wi-Fi. Done.**  
> LocalDrop is a production-grade, Java EE–based LAN file-sharing server.  
> Open the IP in a browser — upload, download, search, and track transfers in real time.

<br/>

---

</div>

## ◈ What Is LocalDrop?

Getting a file from your laptop to another device on the same network should take seconds — not require a cloud service, a USB cable, or a third-party app install. **LocalDrop** is a fully self-hosted LAN file-sharing server you deploy once on any machine in your network. From that point, any phone, tablet, or PC on the same Wi-Fi just opens a browser URL to upload or download files — no software installation required on the receiving end.

It's built like a production system: SHA-256 integrity checking on every upload, a transfer lifecycle engine with full audit logging, a connection-pooled database backend, soft deletes for referential safety, and buffered I/O so it handles large files without blowing up heap memory.

---

## ◈ Why I Built This

Most "quick file transfer" options have a catch:

- **Cloud storage** (Drive, Dropbox) — your file travels to an external server before reaching the next device
- **AirDrop** — Apple-ecosystem only, notoriously unreliable across OS versions
- **Bluetooth** — painfully slow, requires pairing friction
- **USB** — physical cables, not always available

LocalDrop solves the real problem: **get any file to any device on your local network, instantly, from a browser tab** — with zero data leaving your premises. It was also a deliberate decision to build this with the full Java EE stack (Servlets, JSP, JSTL, JDBC), not a modern framework — because understanding the stack underneath the frameworks is what separates engineers from copy-pasters.

---

## ◈ Features

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│   🌐  Browser-based Access     — no app install on receiving devices     │
│   📤  Upload & Download        — buffered 8 KB streaming, up to 500 MB  │
│   🔒  SHA-256 Integrity Check  — every file hashed on-the-fly at upload  │
│   🔍  File Search              — server-side LIKE search across filenames │
│   📊  Transfer Lifecycle       — PENDING → IN_PROGRESS → DONE/FAILED     │
│   ❌  Transfer Cancellation    — cancel in-progress transfers mid-stream  │
│   📈  Download Analytics       — per-file download counter               │
│   🖥️  Real-time Stats Dash     — active files, transfers, storage used   │
│   📡  LAN URL Auto-detection   — server displays its own local IP        │
│   🔄  Dashboard Auto-refresh   — live polling every 30 seconds           │
│   💾  Persistent Storage       — files survive Tomcat restarts           │
│   🛡️  SQL Injection Safe       — all queries use PreparedStatements      │
│   🔗  Connection Pooling       — HikariCP pool (up to 10 connections)    │
│   📝  Full Audit Trail         — every transfer logged with status+time  │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## ◈ Architecture

```
Browser (any device on the LAN)
         │
         │  HTTP  (no WebSockets, no React — just clean Java EE)
         ▼
┌─────────────────────────────────────────────────────────┐
│            Apache Tomcat 10.1  (WAR deployment)         │
│                                                         │
│   MainServlet.java                                      │
│   ├── /upload   → parse multipart, stream to disk,      │
│   │              compute SHA-256, persist metadata       │
│   ├── /download → stream file in 8 KB chunks            │
│   ├── /files    → list + search files (LIKE query)      │
│   ├── /transfers→ audit log (HTML + JSON endpoint)      │
│   ├── /cancel   → set transfer status → CANCELLED       │
│   └── /delete   → soft delete (is_active = 0)           │
│                                                         │
│   Database.java  ←─── HikariCP (pool max=10)            │
│   └── PreparedStatement JDBC calls only                 │
│                                                         │
│   JSP Views (WEB-INF/views/)                            │
│   ├── index.jsp       ← Dashboard + LAN URL display     │
│   ├── files.jsp       ← File browser + search           │
│   ├── upload.jsp      ← Upload form                     │
│   ├── transfers.jsp   ← Transfer log with cancel        │
│   └── error.jsp       ← Centralised error handling      │
└────────────────────────────────┬────────────────────────┘
                                 │  JDBC
                                 ▼
                    ┌────────────────────────┐
                    │     MySQL 8 Database    │
                    │                         │
                    │  shared_files           │
                    │  ├── file metadata      │
                    │  └── SHA-256 hash       │
                    │                         │
                    │  transfers              │
                    │  └── full audit log     │
                    │                         │
                    │  v_active_files  (view) │
                    │  v_transfer_stats(view) │
                    └────────────────────────┘

                    ~/LANShare_Uploads/
                    └── persists outside webapps/
                        (survives WAR redeployments)
```

---

## ◈ Tech Stack

| Layer | Technology | Why This Choice |
|---|---|---|
| **Language** | Java 17 | LTS release — modern features with enterprise-grade stability |
| **Web Layer** | Jakarta Servlet 6.0 + JSP 3.1 | Full control over the request/response lifecycle — no magic, no abstraction leaks |
| **Templating** | JSTL 3.0 (GlassFish impl 3.0.1) | Clean server-side rendering without mixing Java logic into markup |
| **Connection Pool** | HikariCP 5.1 | The fastest JDBC connection pool available; minimal overhead for concurrent LAN requests |
| **Database** | MySQL 8 + JDBC (PreparedStatements) | Relational model fits the file+transfer relationship; PreparedStatements for injection safety |
| **Build Tool** | Maven 3 (WAR packaging) | Dependency management + reproducible builds + standard lifecycle |
| **Server** | Apache Tomcat 10.1 | Jakarta EE 10 compliant; widely deployed; simple WAR drop-in deployment |
| **Logging** | Logback + SLF4J | Structured, configurable logging across the full servlet lifecycle |
| **Frontend** | JSP + JSTL + CSS | No JS framework overhead — the UI is deliberately thin; the backend does the work |

---

## ◈ Database Schema

**`shared_files`** — file metadata registry

| Column | Type | Notes |
|---|---|---|
| `file_id` | `BIGINT` PK | Auto-increment |
| `filename` | `VARCHAR(255)` | Stored name (timestamp-prefixed for uniqueness) |
| `original_name` | `VARCHAR(500)` | User-visible display name |
| `file_size` | `BIGINT` | Size in bytes |
| `file_type` | `VARCHAR(100)` | MIME type |
| `file_hash` | `VARCHAR(64)` | SHA-256 hex digest — computed during upload stream |
| `upload_time` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` |
| `download_count` | `INT` | Incremented on each successful download |
| `is_active` | `TINYINT(1)` | Soft-delete flag — preserves transfer history |

**`transfers`** — full audit log per session

| Column | Type | Notes |
|---|---|---|
| `transfer_id` | `BIGINT` PK | Auto-increment |
| `file_id` | `BIGINT` FK | References `shared_files` |
| `transfer_type` | `ENUM('UPLOAD','DOWNLOAD')` | Direction |
| `status` | `ENUM(...)` | `PENDING → IN_PROGRESS → COMPLETED / FAILED / CANCELLED` |
| `start_time` / `end_time` | `TIMESTAMP` | Session duration tracking |
| `bytes_transferred` | `BIGINT` | Progress tracking |
| `error_message` | `TEXT` | Populated on failure |

**SQL Views:** `v_active_files` and `v_transfer_stats` for dashboard aggregation queries.

---

## ◈ Key Engineering Decisions

**Buffered 8 KB I/O streaming** — Files never load entirely into heap memory. Both upload and download stream in 8 KB chunks, making 500 MB transfers safe under normal JVM heap settings.

**Single-pass SHA-256** — The integrity hash is computed inside the upload stream itself — no second read pass, no temp file. The digest wraps the same `InputStream` that writes to disk.

**HikariCP connection pooling** — A pool of up to 10 database connections handles concurrent LAN requests without the overhead of creating a new connection per request.

**PreparedStatements throughout** — Zero dynamic SQL string concatenation anywhere in `Database.java`. Every query is parameterised, eliminating the SQL injection surface entirely.

**Soft deletes** — Files are never hard-deleted. `is_active = 0` preserves the foreign-key relationship with the transfers table, keeping the full audit log intact.

**Persistent upload directory** — Files are written to `~/LANShare_Uploads`, outside `webapps/`. Files survive WAR redeployments and Tomcat restarts — a real production concern that most demo projects ignore.

**Transfer state machine** — Every upload and download goes through `PENDING → IN_PROGRESS → COMPLETED / FAILED / CANCELLED`. Cancellations flip the status without deleting the record, giving a complete history of every transfer attempt.

**LAN IP auto-detection** — The server enumerates all network interfaces at request time, filters out loopback and virtual adapters, and surfaces the active IPv4 LAN address — so the shareable URL is always correct without any manual configuration.

---

## ◈ Project Structure

```
LocalDrop/
│
├── pom.xml                          ← Maven build (WAR packaging, all deps pinned)
├── db/
│   └── schema.sql                   ← Tables, views, indexes — run once to bootstrap
│
└── src/main/
    ├── java/com/p2pshare/
    │   ├── MainServlet.java          ← Central dispatcher — all HTTP routes handled here
    │   ├── Database.java             ← JDBC data layer — HikariCP pool, PreparedStatements
    │   └── Models.java               ← POJOs: SharedFile, Transfer (with formatting helpers)
    │
    ├── webapp/
    │   ├── WEB-INF/
    │   │   ├── web.xml               ← Jakarta EE 6.0 descriptor (error pages, session config)
    │   │   └── views/
    │   │       ├── index.jsp         ← Dashboard: stats, recent files, LAN URL
    │   │       ├── files.jsp         ← File browser with search
    │   │       ├── upload.jsp        ← Upload form (multipart)
    │   │       ├── transfers.jsp     ← Transfer log with cancellation
    │   │       └── error.jsp         ← Centralised error page
    │   ├── css/                      ← Stylesheets
    │   └── index.jsp                 ← Entry-point redirect
    │
    └── resources/
        ├── db.properties             ← ⚠ Gitignored — create manually (see Setup)
        └── logback.xml               ← Logback/SLF4J configuration
```

---

## ◈ API Routes

| Method | Path | Description |
|---|---|---|
| `GET` | `/` or `/index` | Dashboard — live stats, recent files, LAN URL display |
| `GET` | `/files` | File browser (with optional `?search=` query) |
| `GET` | `/upload` | Upload form |
| `POST` | `/upload` | Process multipart upload — stream to disk + hash + log |
| `GET` | `/download?id=` | Stream file download in 8 KB chunks |
| `POST` | `/files/delete` | Soft-delete a file (`is_active = 0`) |
| `GET` | `/transfers` | Transfer audit log (HTML view) |
| `GET` | `/transfers?format=json` | Transfer log as JSON — used by dashboard auto-refresh |
| `POST` | `/transfers/cancel` | Cancel an in-progress transfer |

---

## ◈ Setup & Run

### Prerequisites

- Java 17+
- Maven 3.8+
- MySQL 8.0+
- Apache Tomcat 10.1

### Steps

**1. Clone the repository**
```bash
git clone https://github.com/harleenkhanuja/LocalDrop.git
cd LocalDrop
```

**2. Bootstrap the database**
```bash
mysql -u root -p < db/schema.sql
```

**3. Create `src/main/resources/db.properties`**
> ⚠️ This file is gitignored and must be created manually.

```properties
db.url=jdbc:mysql://127.0.0.1:3306/p2p_fileshare?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
db.username=root
db.password=yourpassword
```

**4. Build the WAR**
```bash
mvn clean package
```

**5. Deploy to Tomcat**

Copy `target/LocalDrop.war` into Tomcat's `webapps/` directory and start Tomcat.

**6. Open in browser**

```
http://localhost:8080/LocalDrop
```

The dashboard auto-detects your machine's LAN IP and displays a shareable link like:
```
http://192.168.x.x:8080/LocalDrop
```
Any device on the same Wi-Fi can open that URL — no setup required on their end.

---

## ◈ What I'd Build Next

- [ ] **WebSocket progress bar** — real-time byte-level transfer progress streamed to the browser
- [ ] **QR code on dashboard** — scan to connect from phone without typing the IP
- [ ] **PIN-protected downloads** — optional one-time access codes for sensitive files
- [ ] **Multi-file batch upload** — drag a folder, upload everything at once
- [ ] **Docker support** — containerise the full stack (Tomcat + MySQL) for zero-friction deployment
- [ ] **File expiry** — auto-delete files after N hours or N downloads

---

## ◈ What I Learned

Java EE forces you to understand things that frameworks normally hide. Writing `MainServlet.java` as a central dispatcher taught me exactly how HTTP request routing, multipart parsing, and response streaming actually work — before reaching for Spring MVC.

The HikariCP integration made the concurrency model explicit: you have a fixed pool of connections, concurrent requests compete for them, and the pool size is an architectural decision with real tradeoffs. The transfer lifecycle state machine — `PENDING → IN_PROGRESS → COMPLETED / FAILED / CANCELLED` — is conceptually small but getting it right (with proper failure handling, cancellation propagation, and audit log integrity) turned out to be the most interesting engineering challenge in the project.

The biggest surprise: how much complexity lives inside "just copy a file from one device to another." Buffered I/O, hash computation, connection pooling, soft deletes for referential integrity, LAN IP auto-detection — each is a small decision with real production consequences.

---

<div align="center">

**Made by [Harleen Khanuja](https://www.linkedin.com/in/harleenkhanuja/)**


*MIT License — free to use for educational purposes.*

</div>
