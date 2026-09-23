<<<<<<< HEAD
# AI Employee Helpdesk & IT Ticket Routing + RAG (n8n)

## Deskripsi singkat

Workflow n8n untuk assistant IT internal yang menggabungkan Retrieval-Augmented Generation (RAG) dengan Pinecone (vector DB), embeddings (Hugging Face), LLM (Groq), Google Drive untuk sourcing dokumen, dan Supabase untuk ticketing/log.

## Project Workflow

```mermaid
graph TD
    Telegram["Telegram Trigger"] --> LLMChainClass["Basic LLM Chain (Klasifikasi)"]
    Webhook["Webhook"] --> LLMChainClass
    
    Groq1["Groq Chat Model"] -.->|Model AI| LLMChainClass
    
    LLMChainClass --> CodeJS1["Code in JavaScript1 (Parse Intent)"]
    CodeJS1 --> Switch{"Switch (Intent)"}
    
    %% TICKET Branch
    Switch -->|TICKET| LLMChainTicket["Basic LLM Chain1 (Kategorisasi Tiket)"]
    Groq2["Groq Chat Model2"] -.->|Model AI| LLMChainTicket
    LLMChainTicket --> CodeJSTicket["Code in JavaScript (Format Tiket)"]
    CodeJSTicket --> SupabaseRow["Create a row (Supabase: tickets)"]
    SupabaseRow --> TeleTicket["Send a text message (Notifikasi Tiket)"]
    
    %% QUESTION Branch (RAG)
    Switch -->|QUESTION| AIAgent["AI Agent (Penjawab Pertanyaan)"]
    Groq3["Groq Chat Model1"] -.->|Model AI| AIAgent
    
    HFEmbed["Embeddings HuggingFace Inference1"] -.->|Embeddings| Pinecone["Pinecone Vector Store1"]
    Pinecone -.->|RAG Tool| AIAgent
    
    AIAgent --> TeleAnswer["Send a text message1 (Jawaban Telegram)"]
    AIAgent --> SupabaseLog["Create a row1 (Supabase: log interaksi)"]
```

## Fitur utama

- Menjawab pertanyaan karyawan menggunakan dokumen internal (RAG)
- Indexing dokumen Google Drive ke Pinecone (workflow `Set Database Vector`)
- Klasifikasi intent (QUESTION vs TICKET)
- Pembuatan tiket otomatis ke Supabase jika tidak ditemukan jawaban
- Notifikasi via Telegram

## Persyaratan

- n8n
- Pinecone account (index yang direkomendasikan: `it-knowledge-base`)
- Hugging Face API key (embedding model: `sentence-transformers/all-mpnet-base-v2`)
- Groq API key (LLM models digunakan dalam workflow)
- Google Drive OAuth2 (folder dokumentasi)
- Telegram bot token
- Supabase credentials (untuk menyimpan tiket dan log)

## Instalasi & Import

1. Buka n8n → Workflows → Add → Import from File.
2. Pilih file:
   - `AI Employee Helpdesk & IT Ticket Routing + RAG.json`
   - `Set Database Vector.json`
3. Re-bind credential pada setiap node (Pinecone, Hugging Face, Groq, Google Drive, Telegram, Supabase).
4. Untuk workflow `Set Database Vector`, set `folderToWatch` ke folder Google Drive berisi dokumen SOP/FAQ, lalu jalankan sekali atau tunggu trigger.

## Konfigurasi

Set credential / env berikut di n8n atau deployment Anda:

- `PINECONE_API_KEY` — akses ke Pinecone
- `PINECONE_INDEX` — nama index (mis. `it-knowledge-base`)
- `HUGGINGFACE_API_KEY` — untuk membuat embeddings
- `GROQ_API_KEY` — untuk pemanggilan LLM
- `GOOGLE_DRIVE_OAUTH` — untuk mengakses dokumen
- `TELEGRAM_TOKEN` — bot token untuk interaksi
- `SUPABASE_URL`, `SUPABASE_KEY` — untuk menyimpan tiket

## Alur Kerja Singkat

1. `Set Database Vector` (Google Drive Trigger): file baru/terupdate → download → splitter → buat embeddings → upsert ke Pinecone.
2. `AI Employee Helpdesk` (Trigger Telegram / Webhook): terima pertanyaan → klasifikasi intent (QUESTION / TICKET).
   - Jika `QUESTION`: lakukan retrieval top-k dari Pinecone → ringkas sumber → jawab ke user.
   - Jika jawaban tidak ditemukan atau user meminta, buat tiket: generate ID (format `IT-YYYY-XXXX`), simpan di Supabase, kirim notifikasi Telegram.

### Diagram Arsitektur (Mermaid)

```mermaid
flowchart LR
   subgraph Inputs
      TG[Telegram Trigger]
      WH[Webhook]
   end

   TG --> BLL[Basic LLM Chain]
   WH --> BLL

   BLL --> CJ[Code in JavaScript1]
   CJ --> SW[Switch (QUESTION / TICKET)]

   SW -- TICKET --> LLM1[Basic LLM Chain1] --> CJ2[Code in JavaScript] --> SUPA[Create a row (Supabase tickets)] --> TKT[Send a text message (Telegram ticket created)]

   SW -- QUESTION --> GM[Groq Chat Model] --> AGT[AI Agent] --> PINE[Pinecone Vector Store] --> RESP[Send a text message1 & Create a row1]

   subgraph Helpers
      EMB[Embeddings (Hugging Face)] --> PINE
   end
```

### Node & Credential Mapping

- `Telegram Trigger` / `Send a text message` / `Send a text message1`: `telegramApi` — example credential name: "Telegram account"
- `Groq Chat Model` / `Groq Chat Model1` / `Groq Chat Model2`: `groqApi` — "Groq account"
- `Pinecone Vector Store1`: `pineconeApi` — "Pinecone account"
- `Embeddings HuggingFace Inference1`: `huggingFaceApi` — "Hugging Face account"
- `Create a row` / `Create a row1`: `supabaseApi` — "Supabase account 2"

## Contoh Interaksi

- User: "Bagaimana cara reset VPN?"
- Bot: (menjawab dengan ringkasan yang diambil dari dokumentasi jika tersedia) atau
- Bot: "Maaf, saya tidak menemukan informasi. Apakah Anda ingin dibuatkan tiket ke Tim IT?"

## Catatan Indexing & Keamanan

- Jangan index dokumen yang mengandung secrets, kredensial, atau data sensitif.
- Batasi akses Pinecone dan Supabase menggunakan API keys dengan hak minimal.

## File dalam folder

- `AI Employee Helpdesk & IT Ticket Routing + RAG.json` — workflow utama
- `Set Database Vector.json` — workflow untuk membuat / memperbarui vector DB
- `Readme.MD` — dokumentasi ini

## Lisensi

Default: MIT (sesuaikan jika perlu)
=======
# AI Employee Helpdesk & IT Ticket Routing + RAG System

Sistem **AI Employee Helpdesk** berbasis **n8n**, **Groq (Llama 3)**, **Pinecone Vector Database**, dan **Supabase**. Sistem ini dirancang untuk menangani pesan masuk dari pengguna via **Telegram Bot**, mengklasifikasikan niat pengguna (*Intent Classification*), menjawab pertanyaan seputar kebijakan/SOP IT internal menggunakan **RAG (Retrieval-Augmented Generation)**, serta secara otomatis membuat dan merouting tiket gangguan ke tim IT terkait.

---

## 🛠️ Fitur Utama

1. **AI Dispatcher & Intent Classification**:
   - Memilih jalur antara **QUESTION** (Pertanyaan/Panduan) dan **TICKET** (Laporan Masalah/Permintaan Akses).
2. **RAG-Powered AI Agent (Helpdesk Knowledge Base)**:
   - Mencari jawaban langsung dari basis pengetahuan internal yang tersimpan di Pinecone Vector Database (`it-knowledge-base`).
   - Sumber kebenaran tunggal (*Source of Truth*) dari dokumen PDF IT yang diunggah.
3. **Automated Ticket Classification & Routing**:
   - Mengkategorikan masalah (*Hardware, Software, Network, Access Request, Security*).
   - Menentukan prioritas (*P1 - Critical, P2 - High, P3 - Medium, P4 - Low*).
   - Menyarankan tindakan awal (*Troubleshooting Step*) dan menentukan tim yang bertanggung jawab.
   - Format ID Tiket Otomatis: `IT-YYYY-XXXX`.
4. **Database & Logging**:
   - Menyimpan tiket aktif dan riwayat percakapan AI secara real-time ke **Supabase**.
5. **Auto Indexing / Knowledge Ingestion**:
   - Memicu pembaruan basis pengetahuan otomatis dari Google Drive ke Pinecone saat ada perubahan file PDF.

---

## 🏗️ Arsitektur & Diagram Alur Workflows

Proyek ini terdiri dari **2 Workflow Utama n8n** dan **1 File Basis Pengetahuan**:

### 1. Workflow Utama (`AI Employee Helpdesk & IT Ticket Routing + RAG.json`)
```mermaid
flowchart TD
    A[Telegram Trigger / Webhook] -->
    AB[Basic LLM Chain - 
    Groq Llama 3.3 70B] -->
    B[Klasifikasi Intent: QUESTION 
    / TICKET] -->
    C{Swicth Mode}
    C -->|QUESTION| E[AI Agent 
    + Pinecone Tool + 
    HuggingFace Embeddings]
       --> F[Kirim Balasan ke Telegram]
       E-->G[Simpan Log ke Supabase/
       ai_interaction_logs]
    C -->|TICKET| H[Basic LLM Chain - 
    Categorizer & Router]
    --> I[JS Code: Parse & 
    Gen Ticket ID]
    -->J[Simpan Tiket ke Supabase]
    -->K[Kirim Notifikasi Tiket ke 
       Telegram]
  
```

### 2. Workflow Ingest Vector (`Set Database Vector.json`)
```mermaid
flowchart TD
    A[Google Drive Trigger] -->
    AB[Download File PDF] -->
    B[Pinecone Vector Store] 
    B-->BC[Hugging Face Embedding]
    B-->C[Default Data Loader]
    C-->CD[Recursive Character
    Text Splitter]

```

---

## 📁 Struktur File Proyek

```text
.
├── AI Employee Helpdesk & IT Ticket Routing + RAG.json  # Workflow utama Helpdesk, Routing, & RAG
├── Set Database Vector.json                             # Workflow otomatisasi vectorization ke Pinecone
├── 📚 IT KNOWLEDGE BASE.pdf                             # Dokumen SOP, Kebijakan, & Panduan IT
└── README.md                                             # Dokumentasi proyek
```

---

## 📄 File Knowledge Base (Dokumen PDF)

Dokumen **Knowledge Base PDF** berfungsi sebagai *Source of Truth* untuk AI Agent. Dokumen ini memuat panduan operasional IT internal, antara lain:
* **Akses & Jaringan**: Panduan koneksi VPN, Wi-Fi kantor, & reset password.
* **Pengajuan Akses**: Prosedur permintaan akun GitLab, Server, & Database.
* **Perangkat & Perhitungan Troubleshooting**: Penanganan masalah Printer, Laptop, & Software Bug.
* **SOP & Aturan IT Internal**: Aturan penggunaan fasilitas IT perusahaan.

---

## 🚀 Panduan Setup & Instalasi

### 1. Prasyarat (Prerequisites)
* **n8n** (Self-hosted / Cloud)
* Akun **Telegram Bot** (dibuat via `@BotFather`)
* Akun **Groq API**
* Akun **Pinecone** (Buat index bernama `it-knowledge-base`)
* Akun **Hugging Face** (Inference API)
* Akun **Supabase**
* Akun **Google Cloud / Google Drive**

---

### 2. Konfigurasi Database Supabase

Buat dua tabel di Supabase dengan skema berikut:

#### Tabel `tickets`
```sql
CREATE TABLE tickets (
    id BIGSERIAL PRIMARY KEY,
    ticket_id VARCHAR(50) NOT NULL,
    telegram_chat_id BIGINT,
    raw_issue TEXT,
    category VARCHAR(50),
    priority VARCHAR(10),
    summary TEXT,
    suggested_action TEXT,
    assigned_team VARCHAR(100),
    status VARCHAR(20) DEFAULT 'OPEN',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

#### Tabel `ai_interaction_logs`
```sql
CREATE TABLE ai_interaction_logs (
    id BIGSERIAL PRIMARY KEY,
    telegram_chat_id BIGINT,
    intent VARCHAR(20),
    user_query TEXT,
    ai_response TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

### 3. Import Workflow ke n8n

1. Buka dashboard **n8n**.
2. Pilih **Workflows** > **Import from File**.
3. Import kedua file JSON:
   - `Set Database Vector.json`
   - `AI Employee Helpdesk & IT Ticket Routing + RAG.json`

---

### 4. Setup Credentials di n8n

Pastikan untuk mengonfigurasi credential berikut pada node terkait di n8n:
- **Telegram API**: Token Bot Telegram Anda.
- **Groq API**: API Key Groq Anda.
- **Pinecone API**: API Key & Environment Pinecone.
- **Hugging Face API**: Access Token Hugging Face (`sentence-transformers/all-mpnet-base-v2`).
- **Supabase API**: URL & Service Role / Anon Key Supabase.
- **Google Drive OAuth2 API**: Akses ke folder penyimpanan PDF Knowledge Base.

---

## ⚡ Cara Kerja System (Workflow Steps)

### A. Pengisian Vector Database
1. Unggah file `Knowledge_Base_IT.pdf` ke folder Google Drive yang dipantau.
2. Workflow `Set Database Vector` akan secara otomatis mengunduh file PDF tersebut, memecah teks (*Chunking* 800 karakter), membuat vector embedding via Hugging Face, dan menyimpannya ke **Pinecone Vector Store**.

### B. Penanganan Pesan Masuk
1. Pengguna mengirimkan pesan via Telegram.
2. **AI Dispatcher** mengklasifikasikan pesan:
   - **QUESTION**: AI Agent mencari jawaban dari Pinecone. Jika ditemukan, AI menjawab sesuai dokumentasi. Jika tidak, AI menyarankan pembuatan tiket.
   - **TICKET**: AI Router mengkategorikan tiket, menentukan prioritas & tim terkait, menyimpan tiket ke Supabase, dan mengirimkan konfirmasi ID Tiket ke pengguna.

---

## 📝 Lisensi & Kontribusi

Proyek ini dibuat untuk kebutuhan internal IT Service Management (ITSM). Bebas disesuaikan untuk skala industri atau portofolio AI/ML Engineering.
>>>>>>> ec6e0009ed27a86f1e297198f5fb2251aa5ae21c
