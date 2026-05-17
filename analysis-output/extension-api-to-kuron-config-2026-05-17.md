# Analisis Extension API → Kuron Config

**Tanggal**: 2026-05-17
**Scope**: `src/all`, `src/id`, `src/en`
**Tujuan**: Identifikasi extension yang menggunakan API (JSON) dan bisa dikonversi ke Kuron config

---

## Ringkasan

Analisis mendalam terhadap seluruh extension di `src/all`, `src/id`, dan `src/en` untuk mengidentifikasi mana yang menggunakan **API (JSON)** dan bisa dikonversi ke **Kuron config** (`source-config-rest-template.json` atau `source-config-scraper-template.json`).

Metode: membaca source code Kotlin setiap extension, mengidentifikasi pattern API endpoint, dan memetakan ke template Kuron.

---

## Tier 1 — Clean REST API (Paling Cocok untuk Kuron Config)

Extension ini punya endpoint JSON yang bersih, predictable, dan langsung map ke template `source-config-rest-template.json`.

| Extension | Lang | Base URL | API Endpoints | Notes |
|-----------|------|----------|---------------|-------|
| ~~**MangaDex**~~ | all | `api.mangadex.org` | `/manga`, `/chapter`, `/cover`, `/at-home/server/{chapterId}` | ✅ **Sudah diimplementasi di Kuron. Skip.** |
| **Komik Cast** | id | `be.komikcast.cc` | `/series`, `/series/{id}`, `/series/{id}/chapters` | Backend terpisah dari frontend (`v2.komikcast.fit`). JSON bersih. |
| **KomikNesia** | id | `api-be.komiknesia.my.id/api` | `/contents`, `/comic/{id}`, `/comic/{id}/chapters` | Dedicated REST API backend. Frontend di `02.komiknesia.asia`. |
| **Shinigami** | id | `api.shngm.io` | `/v1/manga/list`, `/v1/manga/{id}` | REST API v1. Frontend di `g.shinigami.asia`. |
| **Roseveil** | id | `api.roseveil.org/api` | `/search`, `/manga/{id}`, `/manga/{id}/chapters` | Dedicated REST API. Frontend di `roseveil.org`. |
| **ScansGG** | en | `api.scans.gg` | `/series`, `/chapters`, `/series/{slug}` | REST API terpisah. Frontend di `scans.gg`. |
| **Twicomi** | all | `twicomi.com` | `/api/manga/featured/list`, `/api/manga/list`, `/api/manga/{slug}` | Pagination `page_no`/`page_limit`. API di subdomain sama. |
| **YSK Comics** | all | `ysk-comics.com` | `/api/home/best-comics`, `/api/home/latest-comics`, `/api/comic/{slug}` | REST sederhana. Springboard URL pattern `/{lang}/comic/{stub}`. |
| **PandaChaika** | all | `panda.chaika.moe` | `/search/?json=&page=`, `/api?archive={id}` | REST dengan query params. Support JSON output via `&json=`. |
| **Hentara** | en | `hentara.com` | API via helper `Api.comic()`, `Api.episode()` | REST API. Endpoint terstruktur rapi. |
| **XoManga** | en | `xomanga.site` | `/index.json`, `/our-works.html` | Static JSON file (`index.json`). Sederhana. |
| **ReiManga** | en | `{domain}` | `/api/manga/trending`, `/api/manga`, `/api/manga/{id}` | REST API + RSC hybrid (ada scraping untuk beberapa endpoint). |
| **Corona EX** | all | `{domain}` | `/api/comics/{id}`, `/api/comics` | REST API. Domain configurable. |
| **DreamTeams Scans** | id | `dreamteams.space` | `/$apiBaseUrl/series/comic/{slug}` | REST API. Frontend di `dreamteams.space`. |
| **Manta** | en | `manta.net` | `/manta/v1/search/series`, `/front/v1/series/{id}`, `/front/v1/episodes/{id}` | REST API. **Butuh auth**. |
| **K Manga** | en | `kmanga.kodansha.com` | `/api/ranking/all`, `/api/title/list`, `/api/web/top/updated/title` | REST API. **Butuh auth**. Domain configurable. |
| **WebNovel** | en | `webnovel.com` | API endpoints via `BASE_API_ENDPOINT` | REST API. Kompleks, banyak endpoint. |
| **ComicK Fanmade** | en | `comickfan.com` | `/api/comics/{id}/chapter-list`, `/advanced-search` | REST API. Frontend + API di domain sama. |

---

## Tier 2 — GraphQL API (Bisa tapi Perlu Adaptasi)

Menggunakan GraphQL. Kuron perlu support GraphQL query atau butuh custom adapter.

| Extension | Lang | Base URL | GraphQL Endpoint | Notes |
|-----------|------|----------|------------------|-------|
| **Luscious** | all | `luscious.net` | `/graphql/nobatch/` | GraphQL POST dengan query + variables. Banyak filter (genre, language, content type). Query: `AlbumList`, `AlbumGet`, `AlbumListOwnPictures`. Populer tapi kompleks. |
| **Yabai** | all | `yabai.si` | `/g` | GraphQL POST. Cloudflare bypass. |
| **StashApp** | all | self-hosted | `/graphql` | Self-hosted. GraphQL. Bukan source publik. |

---

## Tier 3 — API tapi Ada Kompleksitas Tambahan

API-based tapi ada logic yang sulit dikonversi ke config standar.

| Extension | Lang | Kompleksitas | Detail |
|-----------|------|--------------|--------|
| **Cubari** | all | WebView JS interceptor | Fetch data dari JavaScript di halaman web. Bukan pure API call. Perlu `RemoteStorageUtils` yang inject JS ke WebView. |
| **HDoujin** | all | WebView Cloudflare bypass + encryption | WebView-based Cloudflare bypass. URL encryption untuk image. |
| **Koharu** | all | Custom clearance + image decryption | Anti-bot clearance. Image decryption dengan public key. |
| **QToon** | all | Domain rotation + encrypted responses | Custom domain rotation. Encrypted API responses (decrypt sebelum parse). |
| **SakuraManhwa** | all | Crypto helper + image decryption | Crypto helper untuk decrypt API response & image. Custom interceptor dengan header `NX`. |
| **Softkomik** | id | Next.js RSC | React Server Components. Parsing dari HTML script tag. Bukan REST murni. Cocok scraper template. |
| **DoujinDesu Unoriginal** | id | Next.js RSC | React Server Components. Parsing dari HTML. Cocok scraper template. |
| **e621** | all | JSON API tapi bukan manga-centric | `/pools.json`. Struktur unik untuk image board. Bukan manga reader typical. |
| **NamiComi** | all | Custom constants + auth | API dengan auth. Constants terpisah di `NamiComiConstants`. |

---

## Tier 4 — Self-Hosted (Bukan untuk Config Publik)

Untuk server pribadi user, bukan source publik yang bisa di-host config-nya.

| Extension | Lang | Notes |
|-----------|------|-------|
| **Komga** | all | Self-hosted comic server. User config URL, username, password sendiri. Multi-library. |
| **Lanraragi** | all | Self-hosted archive reader. |
| **StashApp** | all | Self-hosted media server (GraphQL). |

---

## Pattern API yang Ditemukan

### Pola 1: Standard REST (paling umum, langsung map ke Kuron REST template)

```
GET /api/series?page={page}&limit={limit}
GET /api/series/{id}
GET /api/series/{id}/chapters
GET /api/chapters/{id}/pages
```

**Contoh nyata dari codebase:**

- **Komik Cast**: `GET /series?page={page}`
- **Shinigami**: `GET /v1/manga/list`
- **Twicomi**: `GET /api/manga/featured/list?page_no={page}&page_limit=24`
- **YSK Comics**: `GET /api/home/best-comics`
- **ScansGG**: `GET /api/series`
- **Roseveil**: `GET /api/search`

**Template Kuron**: `source-config-rest-template.json` → **langsung map**

### Pola 2: GraphQL

```
POST /graphql
Body: { query: "...", variables: { page: 1, filters: [...] } }
```

**Contoh nyata:**

- **Luscious**: `POST /graphql/nobatch/?operationName=AlbumList&query=...&variables=...`
- **Yabai**: `POST /g`

**Template Kuron**: Butuh GraphQL support atau custom adapter

### Pola 3: Next.js RSC / Embedded JSON (cocok scraper template)

```
GET /page (HTML)
→ Parse <script id="__NEXT_DATA__"> atau RSC payload
→ Extract JSON dari dalam HTML
```

**Contoh nyata:**

- **Softkomik**: `GET /komik/library` dengan header `RSC: 1`
- **DoujinDesu Unoriginal**: RSC parsing

**Template Kuron**: `source-config-scraper-template.json`

---

## Rekomendasi Prioritas untuk Kuron

### Prioritas 1 — Mudah + High Value (langsung map ke REST template)

| Extension | Alasan |
|-----------|--------|
| ~~**MangaDex**~~ | ✅ Sudah diimplementasi |
| **Komik Cast** | Indonesian, API dedicated backend, JSON bersih |
| **KomikNesia** | Indonesian, API dedicated backend |
| **Shinigami** | Indonesian, REST API v1 |
| **Roseveil** | Indonesian, API dedicated |
| **ScansGG** | English, REST API bersih |

### Prioritas 2 — Medium Effort

| Extension | Alasan |
|-----------|--------|
| **Twicomi** | REST sederhana, pagination jelas |
| **YSK Comics** | REST sederhana, endpoint sedikit |
| **PandaChaika** | REST dengan query params, sudah support JSON output |
| **Hentara** | REST API terstruktur |
| **XoManga** | Static JSON file, sangat sederhana |
| **ReiManga** | REST + RSC hybrid, sebagian bisa REST |

### Prioritas 3 — Perlu GraphQL Support

| Extension | Alasan |
|-----------|--------|
| **Luscious** | GraphQL, populer tapi kompleks |
| **Yabai** | GraphQL, butuh Cloudflare bypass |

### Prioritas 4 — Kompleks / Perlu Custom Adapter

| Extension | Alasan |
|-----------|--------|
| **Cubari** | WebView JS interceptor, bukan pure API |
| **Manta** | Butuh auth |
| **K Manga** | Butuh auth |
| **WebNovel** | Kompleks, banyak endpoint |

---

## Mapping ke Template Kuron

### REST Template (`source-config-rest-template.json`)

Cocok untuk semua **Tier 1** extension:

```json
{
  "source": "komikcast",
  "baseUrl": "https://be.komikcast.cc",
  "api": {
    "endpoints": {
      "allGalleries": "/series?page={page}",
      "search": "/series?title={query}&page={page}",
      "detail": "/series/{id}"
    },
    "list": {
      "items": "$.data[*]",
      "pagination": { "offsetMode": false },
      "fields": { ... }
    },
    "detail": {
      "fields": { ... },
      "chapters": {
        "endpoint": "/series/{id}/chapters",
        "items": "$.data[*]"
      }
    },
    "images": {
      "mode": "direct",
      "baseUrl": "https://images.komikcast.cc"
    }
  }
}
```

### Scraper Template (`source-config-scraper-template.json`)

Cocok untuk **Tier 3** yang pakai Next.js RSC atau HTML parsing:

```json
{
  "source": "softkomik",
  "baseUrl": "https://softkomik.co",
  "scraper": {
    "urlPatterns": {
      "home": { "url": "/komik/library", ... },
      "detail": "/komik/{id}",
      "chapter": "/komik/{id}"
    },
    "selectors": { ... }
  }
}
```

### GraphQL (Belum Ada Template)

Perlu template baru atau custom adapter untuk **Tier 2**:

```json
{
  "source": "luscious",
  "baseUrl": "https://members.luscious.net",
  "graphql": {
    "endpoint": "/graphql/nobatch/",
    "queries": {
      "allGalleries": "query AlbumList($input: AlbumListInput!) { ... }",
      "detail": "query AlbumGet($id: ID!) { ... }"
    }
  }
}
```

---

## Catatan Penting

1. **Verifikasi real website wajib**: Walaupun extension sudah ada di codebase, URL API bisa berubah. Selalu cek endpoint asli sebelum bikin config.

2. **Auth-required sources**: Manta dan K Manga butuh login. Tidak cocok untuk config publik tanpa mekanisme auth di Kuron.

3. **Self-hosted sources**: Komga, Lanraragi, StashApp bukan target config publik. User config sendiri di app.

4. **Multi-language**: Luscious, Twicomi support banyak bahasa. Config Kuron bisa punya `defaultLanguage` yang berbeda.

5. **Image hosting terpisah**: Beberapa extension (Komik Cast, Twicomi) punya CDN terpisah untuk gambar. Perlu field `imageBaseUrl` di config.

6. **Pagination**: Ada 2 pola:
   - **Offset-based**: `?offset=0&limit=30` (ScansGG)
   - **Page-based**: `?page=1&limit=24` (Twicomi, Komik Cast)
   - Template Kuron sudah support keduanya via `offsetMode` di pagination config.
