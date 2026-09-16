# PRD: TuntasUMKM

**AI Agent Operasional untuk Usaha Mikro & Kecil**

---

| Field | Detail |
|---|---|
| **Nama Proyek** | TuntasUMKM |
| **Versi Dokumen** | 1.0 |
| **Tanggal** | 18 Agustus 2026 |
| **Kompetisi** | AI HackFest 2026 – IDwebhost |
| **Kategori** | Business Automation / Customer Service |
| **Tema Kompetisi** | *"Build Agent, Deliver Impact"* |
| **Framework** | OpenClaw / Hermes (dipilih saat registrasi) |
| **Infrastruktur** | Cloud VPS AI Hosting – 4 Core CPU / 4 GB RAM / 20 GB SSD |

---

## 1. Ringkasan Eksekutif

TuntasUMKM adalah AI Agent yang mengubah percakapan pelanggan usaha mikro menjadi pekerjaan operasional yang **selesai**: memahami kebutuhan, mengecek katalog dan stok, membuat draft pesanan, menyiapkan invoice, menyusun balasan, dan mencatat tindak lanjut — semuanya dengan **persetujuan pemilik** sebelum eksekusi final.

> **Satu kalimat positioning:**
> *"TuntasUMKM bukan chatbot yang menjawab — ia adalah karyawan operasional digital yang menyelesaikan pesanan dari chat sampai invoice, dengan pemilik tetap pegang kendali."*

---

## 2. Problem Statement

### 2.1 Masalah Inti

Usaha mikro dan kecil (UMKM) di Indonesia mengelola sebagian besar transaksi melalui chat (WhatsApp, DM Instagram, marketplace chat). Alur kerja saat ini bersifat **manual, linear, dan rawan error**:

```
Pelanggan chat → Pemilik baca satu-satu → Salin data ke spreadsheet
→ Cek stok manual → Hitung total manual → Tulis invoice manual
→ Balas chat → Catat follow-up → Ulangi untuk chat berikutnya
```

### 2.2 Titik Sakit (Pain Points)

| # | Pain Point | Dampak |
|---|---|---|
| 1 | Pemilik membaca & membalas chat satu per satu | Waktu terbuang 2–4 jam/hari hanya untuk administrasi chat |
| 2 | Data pesanan disalin manual dari chat ke spreadsheet | Risiko typo, duplikasi, dan pesanan terlewat |
| 3 | Stok tidak sinkron antara chat dan catatan | Pelanggan mendapat janji barang yang ternyata habis |
| 4 | Tidak ada standar balasan | Respons lambat, inkonsisten, dan tidak profesional |
| 5 | Tidak ada audit trail | Sulit melacak siapa pesan apa, kapan, dan sudah ditindaklanjuti atau belum |

### 2.3 Mengapa Masalah Ini Relevan untuk Juri

- **Spesifik & nyata:** Setiap juri bisa membayangkan skenario toko online kecil yang kewalahan.
- **Terukur:** Waktu proses, akurasi, dan jumlah intervensi bisa dihitung.
- **Aman:** Tidak memerlukan data pribadi nyata; cukup dataset sintetis.
- **End-to-end:** Dari pesan masuk sampai invoice + balasan, semua terlihat dalam satu demo.

---

## 3. Goals & Non-Goals

### 3.1 Goals (In-Scope)

| # | Goal | Kriteria Keberhasilan |
|---|---|---|
| G1 | Agent mengekstrak intent & entitas dari pesan chat pelanggan | Akurasi ekstraksi ≥ 85% pada 40 kasus uji |
| G2 | Agent mengecek stok & harga dari knowledge base (RAG) | 100% jawaban merujuk sumber, 0 halusinasi harga/stok |
| G3 | Agent membuat draft order & invoice secara otomatis | Draft terbentuk dalam < 10 detik per order |
| G4 | Agent menyiapkan balasan pelanggan yang akurat | Balasan konsisten dengan data order & kebijakan toko |
| G5 | Agent meminta approval manusia sebelum eksekusi final | 100% keputusan finansial/stok melewati approval gate |
| G6 | Agent mencatat audit log & metrik operasional | Dashboard menampilkan completion rate, waktu, intervensi |
| G7 | Agent mencegah order duplikat | 0 duplikasi pada pengujian retry/pesan ganda |

### 3.2 Non-Goals (Out-of-Scope)

| # | Non-Goal | Alasan |
|---|---|---|
| NG1 | Integrasi payment gateway nyata | Di luar scope kompetisi; cukup draft invoice |
| NG2 | Integrasi WhatsApp API / kanal chat nyata | Gunakan web inbox simulasi dengan dataset sintetis |
| NG3 | Multi-tenant / multi-toko | Fokus satu toko demo untuk menjaga kompleksitas |
| NG4 | Penanganan komplain/retur penuh | Jadikan fitur bonus; fokus utama: order baru & tanya stok |
| NG5 | Penggunaan data pelanggan nyata | Wajib data sintetis sesuai UU PDP & aturan kompetisi |
| NG6 | Fine-tuning model custom | Gunakan model default panitia; fokus pada orchestration |

---

## 4. User Personas

### 4.1 Persona Utama: Pemilik UMKM (Primary User)

| Atribut | Detail |
|---|---|
| Nama fiktif | Bu Ratna |
| Usaha | Toko kue kering rumahan, 15 varian produk |
| Kanal jualan | WhatsApp & DM Instagram |
| Volume chat | 30–60 pesan/hari |
| Tech literacy | Rendah–menengah; bisa pakai spreadsheet dasar |
| Kebutuhan utama | Otomatisasi administrasi tanpa kehilangan kendali |
| Ketakutan utama | AI salah kirim harga, salah janji stok, atau balas sembarangan ke pelanggan |

### 4.2 Persona Sekunder: Pelanggan (Simulated)

| Atribut | Detail |
|---|---|
| Perilaku | Chat dengan bahasa informal, typo, singkatan, tanya harga, minta varian |
| Ekspektasi | Balasan cepat, akurat, dan jelas |

### 4.3 Persona Juri (Audience Demo)

| Atribut | Detail |
|---|---|
| Kebutuhan | Melihat agent bekerja end-to-end, bukan sekadar menjawab teks |
| Ekspektasi | Arsitektur jelas, metrik terukur, VPS terlihat, guardrail nyata |

---

## 5. Functional Requirements

### 5.1 Workflow 1: Order Baru (Priority: P0 – Wajib)

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  INTAKE      │────▶│ UNDERSTANDING│────▶│  GROUNDING   │
│ Terima pesan │     │ Ekstrak JSON │     │ RAG katalog  │
└─────────────┘     └──────────────┘     └──────┬───────┘
                                                 │
┌─────────────┐     ┌──────────────┐     ┌──────▼───────┐
│  RESPONSE    │◀────│  GUARDRAIL   │◀────│    TOOL      │
│ Balas chat   │     │ Approval     │     │ Cek stok,    │
│ + invoice    │     │ gate         │     │ hitung, draft│
└─────────────┘     └──────────────┘     └──────────────┘
                                                 │
                                          ┌──────▼───────┐
                                          │  ANALYTICS   │
                                          │ Log + metrik │
                                          └──────────────┘
```

**Detail langkah:**

| Tahap | Input | Proses | Output | Guardrail |
|---|---|---|---|---|
| 1. Intake | Pesan teks pelanggan (dari web inbox) | Terima & simpan pesan mentah | `message_id`, timestamp, raw text | – |
| 2. Understanding | Raw text | LLM ekstrak: produk, jumlah, varian, alamat, intent | JSON terstruktur + confidence score | Jika confidence < 0.7 → flag untuk review manual |
| 3. Grounding | JSON entitas | RAG ke knowledge base: katalog, harga, kebijakan | Context chunks + sumber dokumen | Agent **tidak boleh** mengarang harga/stok |
| 4. Tool Execution | JSON + context | Panggil tool: `check_stock()`, `calculate_total()`, `create_draft_order()`, `generate_invoice_draft()` | Draft order + invoice di database | Idempotency key cegah duplikasi |
| 5. Guardrail | Draft order + invoice | Tampilkan di approval queue untuk pemilik | Tombol **Approve** / **Reject** / **Edit** | Tidak ada eksekusi final tanpa approval |
| 6. Response | Approved order | Generate balasan ringkas + lampiran invoice | Teks balasan siap kirim | Template berbasis data, bukan free-form |
| 7. Analytics | Semua log di atas | Catat waktu, status, intervensi | Baris di dashboard metrik | – |

### 5.2 Workflow 2: Pertanyaan Stok / Harga (Priority: P0 – Wajib)

| Tahap | Proses | Output |
|---|---|---|
| Intake | Terima pesan: *"Kue nastar masih ada gak? Berapa sekilo?"* | Raw text |
| Understanding | Intent: `query_stock_price`; entitas: produk = kue nastar, satuan = kg | JSON |
| Grounding | RAG → ambil harga & stok dari knowledge base | Data terverifikasi |
| Response | Balas: *"Kue nastar tersedia, Rp85.000/kg. Stok saat ini 12 kg. Mau order?"* | Teks balasan |
| Analytics | Catat query, waktu respons, sumber | Log |

> **Catatan:** Workflow ini **tidak memerlukan approval gate** karena bersifat informatif, bukan transaksional.

### 5.3 Workflow 3: Komplain / Retur (Priority: P2 – Bonus)

| Tahap | Proses |
|---|---|
| Intake | Terima pesan komplain |
| Understanding | Intent: `complaint`; ekstrak produk, masalah, tanggal order |
| Grounding | Ambil kebijakan retur dari knowledge base |
| Response | Balas empathi + info kebijakan + buat tiket follow-up |
| Guardrail | Tiket masuk approval queue untuk ditindaklanjuti pemilik |

> **Jika waktu tidak cukup, workflow ini cukup berupa stub yang menghasilkan tiket tanpa alur penuh.**

---

## 6. Non-Functional Requirements

| Aspek | Requirement |
|---|---|
| **Latency** | Waktu dari pesan masuk sampai draft tersedia: ≤ 10 detik (order), ≤ 5 detik (tanya stok) |
| **Reliability** | Schema validation pada setiap tool call; timeout 30 detik; retry maksimal 2x dengan exponential backoff |
| **Idempotency** | Setiap order memiliki `idempotency_key`; pesan duplikat tidak membuat order ganda |
| **Audit Trail** | Setiap state transition dicatat: timestamp, actor (agent/human), action, before/after |
| **Fallback** | Jika LLM gagal / timeout → fallback ke rule-based parser untuk ekstraksi dasar |
| **Keamanan** | Tidak ada data pribadi nyata; semua dataset sintetis; tidak ada endpoint publik tanpa autentikasi |
| **Resource** | Harus berjalan pada VPS 4 Core / 4 GB RAM / 20 GB SSD; tidak memerlukan GPU |
| **Compliance** | Patuh UU ITE, UU PDP, dan Terms of Service kompetisi; tidak ada scraping, hacking, atau DDoS |

---

## 7. Arsitektur Teknis

### 7.1 Diagram Arsitektur

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CLOUD VPS AI HOSTING                         │
│                    (4 Core / 4 GB RAM / 20 GB SSD)                  │
│                                                                     │
│  ┌──────────────┐                                                   │
│  │  WEB INBOX   │  ◀── Simulasi chat pelanggan (dataset sintetis)   │
│  │  (Frontend)  │                                                   │
│  └──────┬───────┘                                                   │
│         │ HTTP POST /message                                        │
│         ▼                                                           │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                   AGENT ORCHESTRATOR                          │   │
│  │              (OpenClaw / Hermes Framework)                    │   │
│  │                                                              │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────────────┐     │   │
│  │  │ State      │  │ LLM        │  │ Prompt Templates   │     │   │
│  │  │ Machine    │  │ (Default)  │  │ (System + Few-shot)│     │   │
│  │  └────────────┘  └────────────┘  └────────────────────┘     │   │
│  │                                                              │   │
│  │  ┌────────────────────────────────────────────────────────┐   │   │
│  │  │                  TOOL REGISTRY                         │   │   │
│  │  │  check_stock() │ calc_total() │ create_order()         │   │   │
│  │  │  gen_invoice() │ get_policy() │ create_ticket()        │   │   │
│  │  └────────────────────────────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────────────┘   │
│         │                    │                    │                  │
│         ▼                    ▼                    ▼                  │
│  ┌─────────────┐    ┌──────────────┐    ┌──────────────────┐       │
│  │  KNOWLEDGE  │    │   DATABASE   │    │  APPROVAL QUEUE  │       │
│  │  BASE (RAG) │    │  (SQLite /   │    │  (Frontend +     │       │
│  │             │    │   Postgres)  │    │   Backend)       │       │
│  │ - Katalog   │    │              │    │                  │       │
│  │ - Harga     │    │ - orders     │    │ - Pending list   │       │
│  │ - Kebijakan │    │ - inventory  │    │ - Approve/Reject │       │
│  │ - Jam ops   │    │ - invoices   │    │ - Edit draft     │       │
│  │ - Ongkir    │    │ - audit_log  │    │                  │       │
│  └─────────────┘    │ - metrics    │    └──────────────────┘       │
│                     └──────────────┘                                │
│         │                                                           │
│         ▼                                                           │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                   METRICS DASHBOARD                           │   │
│  │  Completion Rate │ Avg Time │ Intervention Rate │ Accuracy    │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                   RESPONSE GENERATOR                          │   │
│  │  Template-based reply + invoice attachment                    │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.2 Komponen Teknis

| Komponen | Teknologi | Keterangan |
|---|---|---|
| **Framework Agent** | OpenClaw atau Hermes | Dipilih saat registrasi; gunakan model default panitia |
| **LLM** | Model default AI Hosting | Tidak menggunakan API eksternal berbayar |
| **RAG / Knowledge Base** | Vector store ringan (ChromaDB / FAISS) | Embedding dari model default; dokumen: katalog, kebijakan, FAQ |
| **Database** | SQLite (ringan) atau PostgreSQL | Tabel: `products`, `orders`, `invoices`, `audit_log`, `metrics` |
| **Backend API** | Python (FastAPI) atau Node.js (Express) | Endpoint: `/message`, `/approve`, `/reject`, `/metrics` |
| **Frontend** | React / Vue / HTML+JS sederhana | Halaman: Inbox, Approval Queue, Dashboard Metrik |
| **State Machine** | Custom atau library (XState / transisi manual) | States: `received → understanding → grounding → tool_exec → pending_approval → approved → responded → completed` |
| **Deployment** | VPS AI Hosting (CloudBaik) | Wajib deploy di VPS panitia; bukan localhost |

### 7.3 Tool Registry (Function Calling)

```python
# Contoh definisi tool (pseudo-code)

tools = [
    {
        "name": "check_stock",
        "description": "Cek ketersediaan stok produk di database",
        "parameters": {
            "product_name": "string",
            "variant": "string (optional)"
        },
        "returns": {"available": "boolean", "quantity": "integer", "unit": "string"}
    },
    {
        "name": "calculate_total",
        "description": "Hitung total harga berdasarkan produk, jumlah, dan ongkir",
        "parameters": {
            "items": "array of {product, quantity, variant}",
            "shipping_address": "string (optional)"
        },
        "returns": {"subtotal": "float", "shipping": "float", "total": "float"}
    },
    {
        "name": "create_draft_order",
        "description": "Buat draft order di database dengan status pending_approval",
        "parameters": {
            "customer_message_id": "string",
            "items": "array",
            "total": "float"
        },
        "returns": {"order_id": "string", "status": "pending_approval"}
    },
    {
        "name": "generate_invoice_draft",
        "description": "Generate draft invoice berdasarkan order",
        "parameters": {
            "order_id": "string"
        },
        "returns": {"invoice_id": "string", "content": "string"}
    },
    {
        "name": "get_store_policy",
        "description": "Ambil kebijakan toko (retur, jam operasional, dll) via RAG",
        "parameters": {
            "query": "string"
        },
        "returns": {"policy_text": "string", "source": "string"}
    }
]
```

### 7.4 State Machine

```
                    ┌──────────┐
                    │ RECEIVED │
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │UNDERSTAND│──▶ [confidence < 0.7] ──▶ FLAG_REVIEW
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │ GROUNDING│──▶ [no match] ──▶ FALLBACK_RESPONSE
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │TOOL_EXEC │──▶ [tool error] ──▶ RETRY (max 2x) ──▶ ERROR_LOG
                    └────┬─────┘
                         │
               ┌─────────┴─────────┐
               │                   │
          [transaksional]     [informatif]
               │                   │
        ┌──────▼──────┐     ┌─────▼─────┐
        │  PENDING    │     │ RESPONDED │
        │  APPROVAL   │     └─────┬─────┘
        └──────┬──────┘           │
               │                  │
        ┌──────┴──────┐     ┌────▼────┐
        │             │     │COMPLETED│
   ┌────▼───┐   ┌─────▼──┐  └─────────┘
   │APPROVED│   │REJECTED│
   └────┬───┘   └────┬───┘
        │             │
   ┌────▼─────┐  ┌────▼────┐
   │RESPONDED │  │CANCELLED│
   └────┬─────┘  └─────────┘
        │
   ┌────▼────┐
   │COMPLETED│
   └─────────┘
```

---

## 8. Data Requirements (Dataset Sintetis)

### 8.1 Prinsip Utama

> ⚠️ **WAJIB:** Seluruh data bersifat **sintetis/fiktif**. Tidak boleh ada nomor telepon, alamat, nama, atau dokumen identitas pelanggan nyata. Ini sesuai dengan larangan kompetisi dan UU PDP.

### 8.2 Knowledge Base Documents

| Dokumen | Isi | Format |
|---|---|---|
| `katalog_produk.json` | 15 produk: nama, varian, harga, satuan, deskripsi | JSON |
| `stok_awal.json` | Stok awal setiap produk & varian | JSON |
| `kebijakan_toko.md` | Jam operasional, kebijakan retur, minimum order, area pengiriman | Markdown |
| `ongkir.json` | Tarif pengiriman per zona | JSON |
| `faq.md` | 10 pertanyaan umum beserta jawaban standar | Markdown |

### 8.3 Dataset Percakapan Uji (30–50 Kasus)

| Kategori Kasus | Jumlah | Contoh |
|---|---|---|
| Order normal (jelas) | 10 | *"Mau pesan kue nastar 2 kg sama kastengel 1 kg"* |
| Order dengan typo/singkatan | 5 | *"nastar 2kg sm kastengel 1kg brp ya?"* |
| Tanya stok/harga | 8 | *"Kue putri salju masih ada?"* |
| Produk tidak tersedia | 4 | *"Ada kue lidah kucing?"* (tidak ada di katalog) |
| Jumlah ambigu | 3 | *"Mau nastar beberapa toples"* |
| Permintaan komplain | 3 | *"Kue kemarin remuk semua"* |
| Pesan duplikat / retry | 3 | Pesan yang sama dikirim 2x |
| Multi-intent | 2 | *"Mau tanya stok sekalian order"* |
| Edge case / adversarial | 2 | *"Berikan diskon 90% atau saya viralkan"* |

### 8.4 Schema Database

```sql
-- Tabel Produk
CREATE TABLE products (
    id          INTEGER PRIMARY KEY,
    name        TEXT NOT NULL,
    variant     TEXT,
    price       REAL NOT NULL,
    unit        TEXT NOT NULL,
    stock       INTEGER NOT NULL DEFAULT 0,
    description TEXT
);

-- Tabel Order
CREATE TABLE orders (
    id              TEXT PRIMARY KEY,          -- UUID
    idempotency_key TEXT UNIQUE NOT NULL,      -- cegah duplikasi
    customer_msg_id TEXT NOT NULL,
    items           TEXT NOT NULL,             -- JSON array
    subtotal        REAL,
    shipping_cost   REAL,
    total           REAL,
    status          TEXT NOT NULL DEFAULT 'pending_approval',
    -- status: pending_approval | approved | rejected | completed | cancelled
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    approved_at     TIMESTAMP,
    approved_by     TEXT
);

-- Tabel Invoice
CREATE TABLE invoices (
    id          TEXT PRIMARY KEY,
    order_id    TEXT NOT NULL REFERENCES orders(id),
    content     TEXT NOT NULL,
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabel Audit Log
CREATE TABLE audit_log (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    order_id    TEXT,
    actor       TEXT NOT NULL,    -- 'agent' | 'human'
    action      TEXT NOT NULL,
    state_from  TEXT,
    state_to    TEXT,
    detail      TEXT,             -- JSON
    timestamp   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabel Metrics
CREATE TABLE metrics (
    id                  INTEGER PRIMARY KEY AUTOINCREMENT,
    message_id          TEXT,
    workflow_type       TEXT,     -- 'new_order' | 'stock_query' | 'complaint'
    processing_time_ms  INTEGER,
    extraction_correct  BOOLEAN,
    human_intervention  BOOLEAN,
    completion_status   TEXT,     -- 'completed' | 'failed' | 'cancelled'
    timestamp           TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 9. Guardrail & Approval System

### 9.1 Kapan Approval Diperlukan

| Aksi | Perlu Approval? | Alasan |
|---|---|---|
| Menjawab pertanyaan stok/harga | ❌ Tidak | Informatif, tidak mengubah state |
| Membuat draft order | ❌ Tidak | Draft belum final |
| **Mengurangi stok** | ✅ **Ya** | Berdampak pada inventaris |
| **Mengirim invoice ke pelanggan** | ✅ **Ya** | Komitmen finansial |
| **Mengirim balasan yang bersifat janji** | ✅ **Ya** | Risiko reputasi |
| Membuat tiket komplain | ✅ **Ya** | Perlu penilaian manusia |

### 9.2 Approval Queue UI

```
┌─────────────────────────────────────────────────────────┐
│  📋 APPROVAL QUEUE                          [3 pending] │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │ ORDER #ORD-20260915-001                       │    │
│  │ Pelanggan: Pelanggan_Sintetis_07              │    │
│  │ Items: Nastar 2kg (Rp170.000)                 │    │
│  │        Kastengel 1kg (Rp95.000)               │    │
│  │ Ongkir: Rp15.000                              │    │
│  │ TOTAL: Rp280.000                              │    │
│  │                                               │    │
│  │ [ ✅ Approve ]  [ ✏️ Edit ]  [ ❌ Reject ]    │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │ ORDER #ORD-20260915-002                       │    │
│  │ ...                                           │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 9.3 Anti-Halusinasi Rules

| Rule | Implementasi |
|---|---|
| Agent **tidak boleh** menyebut harga yang tidak ada di knowledge base | Prompt: *"Jika harga tidak ditemukan di konteks, jawab: 'Mohon maaf, saya perlu konfirmasi ke pemilik toko.'"* |
| Agent **tidak boleh** mengarang stok | Tool `check_stock()` adalah satu-satunya sumber kebenaran stok |
| Agent **tidak boleh** menjanjikan diskon/promo | Prompt: *"Jangan menawarkan diskon, promo, atau penawaran khusus kecuali tertulis di knowledge base."* |
| Agent **tidak boleh** mengirim pesan tanpa approval | State machine: transisi ke `RESPONDED` hanya dari `APPROVED` |

---

## 10. Metrik Keberhasilan & Benchmark

### 10.1 Metrik yang Diukur

| Metrik | Cara Ukur | Target |
|---|---|---|
| **Akurasi ekstraksi order** | % kasus dengan produk, jumlah, varian benar | ≥ 85% |
| **Akurasi jawaban stok/harga** | % jawaban yang match dengan database | 100% |
| **Completion rate** | % kasus yang mencapai draft/approval tanpa error | ≥ 90% |
| **Waktu pemrosesan** | Rata-rata ms dari pesan masuk → draft tersedia | ≤ 10.000 ms |
| **Human intervention rate** | % kasus yang butuh koreksi manual | ≤ 15% |
| **Duplicate prevention** | Jumlah order ganda pada uji retry | 0 |
| **Citation/grounding rate** | % jawaban kebijakan yang punya rujukan sumber | 100% |

### 10.2 Format Pelaporan di Dashboard

```
╔══════════════════════════════════════════════════╗
║           TUNTASUMKM - DASHBOARD METRIK          ║
╠══════════════════════════════════════════════════╣
║                                                  ║
║  Total Pesan Diproses    : 40                    ║
║  Completion Rate         : 92.5%  (37/40)       ║
║  Akurasi Ekstraksi       : 87.5%  (35/40)       ║
║  Akurasi Stok/Harga      : 100%   (40/40)       ║
║  Avg Processing Time     : 6.2 detik             ║
║  Human Intervention      : 12.5%  (5/40)        ║
║  Duplicate Orders        : 0                     ║
║  Grounding Rate          : 100%                  ║
║                                                  ║
║  ┌──────────────────────────────────────────┐    ║
║  │  [Bar chart: completion by workflow]     │    ║
║  │  Order Baru     ████████████████ 95%     │    ║
║  │  Tanya Stok     █████████████████ 100%   │    ║
║  │  Komplain       ████████████ 75%         │    ║
║  └──────────────────────────────────────────┘    ║
║                                                  ║
║  Baseline (Manual): 40 order × 3 menit = 120 mnt║
║  Dengan Agent     : 40 order × 6 detik  = 4 mnt ║
║  Penghematan      : ~97% waktu pemrosesan       ║
║                                                  ║
╚══════════════════════════════════════════════════╝
```

> ⚠️ **Penting:** Laporkan angka apa adanya. Jika akurasi 86%, tulis 86% dan jelaskan kasus gagal. Transparansi > klaim sempurna.

---

## 11. UI/UX Requirements

### 11.1 Halaman yang Dibutuhkan

| Halaman | Fungsi | Prioritas |
|---|---|---|
| **Web Inbox** | Simulasi chat pelanggan; kirim pesan sintetis; lihat balasan agent | P0 |
| **Approval Queue** | Daftar draft order/invoice yang menunggu persetujuan; tombol approve/reject/edit | P0 |
| **Order Detail** | Lihat detail satu order: items, total, status, audit log | P0 |
| **Metrics Dashboard** | Tampilkan metrik benchmark secara visual | P0 |
| **Knowledge Base Viewer** | Lihat dokumen RAG yang dipakai (opsional, bagus untuk demo) | P1 |

### 11.2 Prinsip Desain

- **Sederhana & fungsional:** Tidak perlu UI cantik; yang penting jelas dan bisa didemokan.
- **Mobile-friendly tidak wajib:** Demo dilakukan di desktop/laptop.
- **Gunakan framework CSS ringan** (Tailwind, Bootstrap) agar cepat dibangun.

---

## 12. Prompt Engineering Strategy

### 12.1 System Prompt (Agent Orchestrator)

```
Kamu adalah TuntasUMKM, agen operasional untuk toko [NAMA_TOKO_DEMO].

TUGAS UTAMA:
- Mengekstrak intent dan entitas dari pesan pelanggan.
- Menggunakan tool yang tersedia untuk mengecek stok, menghitung harga, dan membuat draft order.
- TIDAK BOLEH mengarang harga, stok, atau kebijakan. Selalu gunakan data dari knowledge base atau tool.
- Jika informasi tidak ditemukan, jawab: "Mohon maaf, saya perlu konfirmasi ke pemilik toko."
- Untuk keputusan transaksional (kurangi stok, kirim invoice), buat draft dan tunggu approval.

FORMAT OUTPUT:
Selalu output dalam JSON terstruktur:
{
  "intent": "new_order | stock_query | price_query | complaint | other",
  "confidence": 0.0-1.0,
  "entities": { ... },
  "tool_calls": [ ... ],
  "response_draft": "...",
  "requires_approval": true/false,
  "source_documents": ["katalog_produk.json", ...]
}

BATASAN:
- Jangan pernah memberikan diskon atau promo yang tidak ada di knowledge base.
- Jangan pernah mengirim pesan langsung ke pelanggan tanpa approval.
- Jika confidence < 0.7, set requires_approval = true dan flag untuk review.
```

### 12.2 Few-Shot Examples (Minimal 3)

Sertakan contoh:
1. Order normal → ekstraksi benar → tool call → draft
2. Tanya stok → jawab dari RAG → tanpa approval
3. Produk tidak ada → fallback response → sarankan alternatif

---

## 13. Compliance & Constraints

| Aspek | Requirement |
|---|---|
| **UU PDP** | Tidak ada data pribadi nyata. Semua nama, alamat, nomor telepon bersifat sintetis dan diberi label `[SYNTHETIC]` |
| **UU ITE** | Tidak ada hacking, DDoS, scraping yang melanggar ToS |
| **Aturan Kompetisi** | Tidak ada unauthorized access; tidak ada penyimpanan/penyebaran data pribadi tanpa izin |
| **AI Model** | Gunakan model default panitia. Jika pakai API eksternal, tanggung jawab peserta dan harus ada fallback |
| **Framework** | Wajib menggunakan OpenClaw atau Hermes (dipilih saat registrasi) |
| **Deployment** | Wajib di-deploy di VPS AI Hosting yang disediakan panitia |
| **Orisinalitas** | Karya orisinal, belum pernah diikutsertakan di kompetisi lain |
| **Transparansi AI** | Jika menggunakan AI untuk membantu development, jelaskan di artikel |

---

## 14. Deliverables Kompetisi

### 14.1 Video Demo (5–10 menit)

| Menit | Konten |
|---|---|
| 0:00 – 1:00 | **Hook & Masalah:** Siapa Bu Ratna, mengapa chat UMKM bikin kewalahan, data singkat |
| 1:00 – 2:00 | **Solusi & Arsitektur:** Apa itu TuntasUMKM, diagram arsitektur, framework yang dipakai |
| 2:00 – 4:00 | **Demo Workflow 1 (Order Baru):** Pesan masuk → ekstraksi → RAG → tool call → draft → approval → balasan |
| 4:00 – 5:30 | **Demo Workflow 2 (Tanya Stok):** Pesan masuk → jawab dari knowledge base → tanpa approval |
| 5:30 – 6:30 | **Dashboard VPS AI Hosting:** Tunjukkan terminal, dashboard CloudBaik, deployment berjalan |
| 6:30 – 8:00 | **Benchmark & Metrik:** Tampilkan hasil 40 kasus uji, before-after, keterbatasan |
| 8:00 – 8:30 | **Roadmap & Penutup:** Apa yang bisa dikembangkan, terima kasih |

**Checklist Video:**
- [ ] Durasi 5–10 menit
- [ ] Landscape 16:9, minimal 1080p
- [ ] Watermark logo IDwebhost di sudut video (permanen)
- [ ] Penyebutan "IDwebhost" dan "AI Hosting" minimal 1x (verbal atau lower-third)
- [ ] Memperlihatkan dashboard & terminal VPS AI Hosting
- [ ] Diunggah ke YouTube/TikTok/IG Reels (publik atau unlisted, **bukan** private)
- [ ] Tidak menggunakan musik/footage berhak cipta

### 14.2 Artikel (Minimal 800 kata)

**Struktur artikel:**

| Section | Konten |
|---|---|
| Judul | Menarik, mengandung kata kunci |
| Latar Belakang | Masalah UMKM dengan chat, mengapa ini penting |
| Solusi | Apa itu TuntasUMKM, bagaimana ia bekerja |
| Arsitektur | Diagram + penjelasan komponen |
| Demo & Hasil | Screenshot, metrik benchmark, before-after |
| Keterbatasan | Jujur tentang apa yang belum sempurna |
| Roadmap | Pengembangan ke depan |
| **Backlink Wajib** | Anchor "AI Hosting" → `idwebhost.com/ai-hosting/` |
| **Backlink Wajib** | Anchor "Cloud VPS" → `cloudbaik.com` |

**Checklist Artikel:**
- [ ] Minimal 800 kata
- [ ] Orisinal, belum pernah dipublikasikan
- [ ] Dipublikasikan di platform publik (blog, Medium, LinkedIn Articles)
- [ ] Dapat diindeks mesin pencari (bukan private/paywall)
- [ ] Dua backlink wajib dengan anchor text yang benar
- [ ] Transparansi penggunaan AI dalam development (jika ada)

---

## 15. Timeline & Milestones

| Periode | Deliverable | Status |
|---|---|---|
| **18–31 Agustus** | Registrasi, join WA group, pilih kategori, siapkan problem statement, buat dataset sintetis, definisikan metrik | ⬜ |
| **1 September** | Technical meeting, pastikan batch/resource, kunci framework, uji akses VPS | ⬜ |
| **4–10 September** | Selesaikan database, RAG pipeline, tool registry, satu workflow order end-to-end | ⬜ |
| **11–17 September** | Tambah approval gate, audit log, idempotency, retry/timeout, dashboard, workflow tanya stok | ⬜ |
| **18–22 September** | Jalankan benchmark 30–50 kasus, perbaiki failure mode, kumpulkan screenshot metrik | ⬜ |
| **23–25 September** | **FREEZE FITUR.** Deploy versi final, uji dari VPS bersih, siapkan backup & demo script | ⬜ |
| **26–28 September** | Rekam video 5–10 menit, edit, pastikan semua checklist video terpenuhi | ⬜ |
| **29 September** | Publikasikan artikel 800+ kata dengan backlink, cek indeksabilitas | ⬜ |
| **30 September** | **SUBMIT.** Isi formulir submit, verifikasi semua link, simpan bukti | ⬜ |

---

## 16. Risk Register

| Risiko | Probabilitas | Dampak | Mitigasi |
|---|---|---|---|
| Model default tidak cukup akurat untuk ekstraksi bahasa informal | Sedang | Tinggi | Siapkan fallback rule-based parser; few-shot examples yang kuat |
| VPS resource terbatas (4 GB RAM) | Rendah | Sedang | Gunakan SQLite; hindari model besar; optimasi memory |
| Waktu development tidak cukup untuk semua fitur | Sedang | Tinggi | Prioritaskan P0 (order + stok); komplain jadi bonus; freeze di 23 Sept |
| RAG mengambil dokumen yang salah | Sedang | Sedang | Chunking yang baik; metadata filtering; fallback jika tidak ada match |
| Video melebihi/kurang dari durasi | Rendah | Tinggi | Script video dulu; latihan 2x sebelum rekam; target 8 menit |
| Backlink artikel tidak terindeks | Rendah | Sedang | Publikasikan H-2; cek via Google Search; gunakan platform yang pasti terindeks |
| Perubahan aturan saat technical meeting | Rendah | Sedang | Ikuti aturan terbaru; revisi checklist setelah TM |

---

## 17. Definition of Done (DoD)

Proyek TuntasUMKM dianggap **selesai dan siap submit** ketika:

- [ ] Agent dapat menerima pesan dari web inbox
- [ ] Agent mengekstrak intent & entitas dengan confidence score
- [ ] Agent melakukan RAG ke knowledge base dan menampilkan sumber
- [ ] Agent memanggil tool (cek stok, hitung total, buat draft)
- [ ] Agent meminta approval manusia untuk keputusan transaksional
- [ ] Agent menghasilkan balasan yang akurat setelah approval
- [ ] Audit log mencatat setiap state transition
- [ ] Dashboard menampilkan minimal 5 metrik utama
- [ ] Idempotency key mencegah order duplikat
- [ ] Benchmark 30–50 kasus telah dijalankan dan hasilnya dicatat
- [ ] Aplikasi ter-deploy di VPS AI Hosting dan dapat diakses
- [ ] Video demo 5–10 menit telah direkam dengan semua checklist terpenuhi
- [ ] Artikel 800+ kata telah dipublikasikan dengan backlink wajib
- [ ] Formulir submit telah diisi dan link terverifikasi

---

## 18. Appendix

### A. Contoh Dataset Sintetis – Katalog Produk

```json
[
  {
    "id": 1,
    "name": "Kue Nastar",
    "variant": "Original",
    "price": 85000,
    "unit": "kg",
    "stock": 12,
    "description": "Kue nastar isi selai nanas homemade"
  },
  {
    "id": 2,
    "name": "Kue Kastengel",
    "variant": "Keju Edam",
    "price": 95000,
    "unit": "kg",
    "stock": 8,
    "description": "Kue kastengel premium dengan keju edam"
  },
  {
    "id": 3,
    "name": "Putri Salju",
    "variant": "Original",
    "price": 75000,
    "unit": "kg",
    "stock": 15,
    "description": "Kue putri salju lembut dengan taburan gula halus"
  }
]
```

### B. Contoh Pesan Uji

```
[PESAN UJI #01 - Order Normal]
"Halo, saya mau pesan kue nastar 2 kg sama kastengel 1 kg. Kirim ke Jakarta Selatan ya."

[EXPECTED EXTRACTION]
{
  "intent": "new_order",
  "confidence": 0.95,
  "entities": {
    "items": [
      {"product": "Kue Nastar", "quantity": 2, "unit": "kg"},
      {"product": "Kue Kastengel", "quantity": 1, "unit": "kg"}
    ],
    "shipping_area": "Jakarta Selatan"
  },
  "requires_approval": true
}

[PESAN UJI #15 - Produk Tidak Tersedia]
"Ada kue lidah kucing gak?"

[EXPECTED RESPONSE]
"Mohon maaf, saat ini kami belum menyediakan kue lidah kucing. 
Kami memiliki nastar, kastengel, dan putri salju. Apakah ada yang bisa saya bantu?"
```

### C. Referensi Dokumen Kompetisi

- Halaman kompetisi: [idwebhost.com/competition](https://idwebhost.com/competition)
- Artikel pengumuman: [idwebhost.com/blog/ai-hackfest-2026/](https://idwebhost.com/blog/ai-hackfest-2026/)
- Formulir registrasi: [forms.gle/aFnhbAPvs3q1zZD3A](https://forms.gle/aFnhbAPvs3q1zZD3A)
- Formulir submit: [forms.gle/s6y8vLzTgosz8rZ39](https://forms.gle/s6y8vLzTgosz8rZ39)
- AI Hosting: [idwebhost.com/ai-hosting/](https://idwebhost.com/ai-hosting/)
- Cloud VPS: [cloudbaik.com](https://cloudbaik.com)

---

*Dokumen ini disusun sebagai panduan pengembangan proyek TuntasUMKM untuk AI HackFest 2026. Semua keputusan teknis final harus disesuaikan dengan ketentuan terbaru dari panitia pada technical meeting 1 September 2026.*

---

**End of PRD**