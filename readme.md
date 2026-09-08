# Facebook Group Scraper & Automated ETL Pipeline

An end-to-end data engineering pipeline designed to continuously extract, clean, deduplicate, and store Facebook group post data with minimal disk space overhead.

The project uses a decoupled architecture: browser-based DOM scraping handles raw data extraction, while an automated background Python ETL script handles data transformation, schema normalization, primary key hashing, and raw file purging.

---

## 🏗️ Architecture

```text
[ Facebook Feed ] 
       │ (Auto "See More" Expander via DevTools)
       ▼
[ Instant Data Scraper Extension ]
       │ (3-Min Auto Download)
       ▼
[ Downloads Directory ] ─── (Raw facebook*.csv files)
       │
       ▼
[ Python ETL Engine ]
       ├── Extracts core fields (Author, URL, Captions)
       ├── Merges text fragments & cleans whitespace
       ├── Generates SHA-256 Primary Keys (`post_id`)
       ├── Deduplicates records globally
       └── Appends to Master CSV / Database Target
       │
       ▼
[ Disk Cleanup ] ─── (Purges raw CSVs to prevent storage bloat)

```

---

## 🌟 Key Features

* **DOM Text Auto-Expansion:** DevTools console snippet automatically clicks "See More" buttons prior to extraction to capture un-truncated post captions.
* **Deterministic Hashing:** Generates a 16-character SHA-256 hash derived from the post URL (`post_id`) to maintain primary key integrity across multi-run ingestion.
* **Schema Normalization:** Merges fragmented React DOM text columns into a clean single caption field while filtering out noise (UI button labels, tracking hashes, image links).
* **Storage Optimization:** Automatically purges processed raw CSV downloads after ingestion to maintain zero-disk-bloat during continuous scraping sessions.
* **Safe Error Handling:** Includes fallback write operations if output files are locked by external applications (e.g., Excel).

---

## 📁 Repository Structure

```text
├── helpers/
│   ├── see_more_expander.js     # DevTools console snippet for expanding text
│   └── ids_auto_downloader.js   # DevTools console snippet for IDS extension
├── process_and_purge.py          # Main Python ETL and cleanup script
├── clean_facebook_posts.csv      # Generated master target dataset
├── requirements.txt              # Python dependencies
└── README.md

```

---

## 📊 Target Data Schema

| Field | Type | Description |
| --- | --- | --- |
| `post_id` | `VARCHAR(16)` | Primary Key (16-character deterministic SHA-256 hash of `post_url`) |
| `post_url` | `TEXT` | Clean permalink to the Facebook post |
| `author_name` | `VARCHAR(255)` | Poster's display name |
| `author_url` | `TEXT` | Poster's profile link |
| `post_caption` | `TEXT` | Consolidated and cleaned post body text |
| `scraped_at` | `TIMESTAMP` | Record ingestion timestamp |

---

## 🚀 Getting Started

### 1. Prerequisites

* **Python 3.9+**
* **Google Chrome** with the **Instant Data Scraper** extension installed.

Install dependencies:

```bash
pip install pandas

```

### 2. Execution Steps

1. **Start the Expander Script:**
* Open the target Facebook group in Chrome.
* Open DevTools (`F12` $\rightarrow$ **Console** tab), paste `helpers/see_more_expander.js`, and press `Enter`.


2. **Configure & Start Instant Data Scraper:**
* Open Instant Data Scraper on the Facebook page and select the main post container.
* Set delay ranges (`3` to `8` seconds).
* Open DevTools on the extension popup window, paste `helpers/ids_auto_downloader.js` into the console, and hit `Enter`.
* Click **Start crawling**.


3. **Run the ETL & Cleanup Pipeline:**
* Run the Python script to process downloaded files, merge them into the master file, and purge the raw CSVs:


nya
```bash
python process_and_purge.py
