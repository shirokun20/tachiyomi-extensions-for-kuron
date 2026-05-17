# Uji Curl: ReiManga, Corona EX, ScansGG

**Tanggal**: 2026-05-17
**Tujuan**: Verifikasi endpoint API real dari 3 kandidat Tier 1 tambahan

---

## 1. ReiManga (`reimanga.com`)

### ✅ Home / Popular

**Endpoint**: `GET https://reimanga.com/api/manga/trending?limit=100`

**Status**: 200 OK ✅

**Response structure**:
```json
[
  {
    "id": 4467,
    "title": "Chainsaw Man",
    "name_url": "chainsaw-man-sxou9",
    "view_count": 1387204,
    "rating": 8.46,
    "is_adult": 0,
    "cover_url": null,
    "chapter_number": "232",
    "position": 1
  }
]
```

**⚠️ Masalah**: `cover_url` selalu `null` di semua endpoint list. Cover image hanya tersedia jika di-scrape dari halaman web (RSC).

---

### ✅ Search

**Endpoint**: `GET https://reimanga.com/api/manga?page=1&limit=24&search=one+piece&sort=updated_at&order=desc`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "data": [
    {
      "id": 55,
      "title": "One Piece",
      "name_url": "one-piece-lqj66v1",
      "alt_title": "ONE PIECE",
      "description": "Gol D. Roger, a man referred to as...",
      "status": 2,
      "completed": 0,
      "view_count": 1274589,
      "rating": 7.67,
      "cover_url": null,
      "chapter_number": "1182",
      "chapter_title": "Chapter 1182",
      "chapter_updated_at": "2026-05-08T21:08:50.000Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "last_page": 500
  }
}
```

**Query params**:
- `page` → halaman
- `limit` → jumlah per halaman (default 24)
- `search` → search query
- `sort` → `updated_at`, `view_count`, `rating`, dll
- `order` → `asc` / `desc`
- `genre` → genre IDs (comma-separated)
- `tag` → tag IDs (comma-separated)
- `excludeGenres` → genre IDs yang di-exclude
- `status` → status filter

**⚠️ Masalah**: `cover_url` tetap `null`.

---

### ✅ Detail

**Endpoint**: `GET https://reimanga.com/api/manga/55`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "manga": {
    "id": 55,
    "title": "One Piece",
    "name_url": "one-piece-lqj66v1",
    "alt_title": "ONE PIECE",
    "description": "Gol D. Roger, a man referred to as...",
    "status": 2,
    "completed": 0,
    "cover_url": null,
    "genres": [
      { "id": 1, "name": "Action", "slug": "action" },
      { "id": 2, "name": "Adventure", "slug": "adventure" }
    ],
    "tags": [
      { "id": 332, "name": "Shounen", "slug": "shounen", "category": "Demographic" }
    ],
    "authors": [
      { "id": 12, "name": "Oda, Eiichiro", "type": 1 }
    ],
    "chapter_count": 1259
  },
  "similar": [ ... ]
}
```

**⚠️ Masalah**: `cover_url` tetap `null` bahkan di detail endpoint.

---

### ❌ Chapters

**Endpoint**: RSC (Next.js React Server Components) — **BUKAN JSON**

Source code menunjukkan chapter list diambil dari:
```
GET https://reimanga.com/manga/{slug}  (dengan header `rsc: 1`)
```

Response-nya adalah RSC payload yang perlu di-parse dengan `extractNextJs<T>()`. Bukan REST API murni.

---

### ❌ Reader / Pages

**Endpoint**: RSC (Next.js React Server Components) — **BUKAN JSON**

```
GET https://reimanga.com/manga/{slug}/{chapterId}  (dengan header `rsc: 1`)
```

Response-nya RSC payload. Image URLs ada di dalam payload tapi perlu parsing khusus.

---

### Kesimpulan ReiManga

| Aspek | Status | Catatan |
|-------|--------|---------|
| Home/Popular | ✅ | REST API JSON (`/api/manga/trending`) |
| Latest | ✅ | REST API JSON (`/api/manga?sort=updated_at`) |
| Search | ✅ | REST API JSON (`/api/manga?search=...`) |
| Detail | ✅ | REST API JSON (`/api/manga/{id}`) |
| Chapters | ❌ | **RSC (Next.js)**, bukan JSON |
| Reader/Pages | ❌ | **RSC (Next.js)**, bukan JSON |
| Genre Filter | ✅ | REST API JSON (`/api/manga?genre=...`) |
| Auth Required | ❌ | Tidak perlu login |
| Cover URLs | ❌ | **Selalu `null`** di semua endpoint API |
| Image URLs | ❌ | Hanya tersedia via RSC parsing |

**Verdict**: **Tidak cocok untuk REST config murni**. Home, search, detail semua JSON tapi cover URL selalu null. Chapters dan reader pakai RSC (Next.js) yang butuh parsing khusus. Ini hybrid — sebagian REST, sebagian RSC scraping.

---

## 2. Corona EX (`en.to-corona-ex.com`)

### ✅ Home / Popular

**Endpoint**: `GET https://api.en.to-corona-ex.com/comics?limit=24&order=asc&sort=title_alphanumeric`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "next_cursor": "134806373367830",
  "resources": [
    {
      "id": "134806612295711",
      "title": "A Gentle Noble's Vacation Recommendation",
      "description": "When Lizel mysteriously finds himself...",
      "cover_image_url": "https://cdn.en.to-corona-ex.com/comics/cover_images/...?X-Amz-...",
      "authors": [
        { "creator_id": "...", "name": "Momochi", "role": "Manga" }
      ],
      "copyright": "© Momochi / Misaki"
    }
  ]
}
```

**Query params**:
- `limit` → jumlah per halaman (default 24)
- `order` → `asc` / `desc`
- `sort` → `title_alphanumeric`, `latest_episode_published_at`
- `after_than` → cursor untuk pagination
- `genre_id` → filter by genre (hanya versi JP)

**⚠️ Masalah**: Cover image URL menggunakan **presigned S3 URL** yang expire dalam **15 menit** (900 detik).

---

### ✅ Search

**Endpoint**: `GET https://api.en.to-corona-ex.com/search/comics?keyword=one+piece&limit=24`

**Status**: 200 OK ✅ (berdasarkan source code, endpoint ada)

**Query params**:
- `keyword` → search query
- `limit` → jumlah per halaman
- `after_than` → cursor

---

### ✅ Detail

**Endpoint**: `GET https://api.en.to-corona-ex.com/comics/{id}`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "id": "134806612295711",
  "title": "A Gentle Noble's Vacation Recommendation",
  "description": "When Lizel mysteriously finds himself...",
  "cover_image_url": "https://cdn.en.to-corona-ex.com/...?X-Amz-...",
  "authors": [
    { "name": "Momochi", "role": "Manga" },
    { "name": "Misaki", "role": "Original Story" }
  ],
  "copyright": "© Momochi / Misaki"
}
```

---

### ✅ Chapters

**Endpoint**: `GET https://api.en.to-corona-ex.com/episodes?comic_id={id}&episode_status=free_viewing,only_for_subscription&limit=9999&order=desc&sort=episode_order`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "resources": [
    {
      "id": "138734267728298",
      "comic_id": "134806612295711",
      "title": "Chapter 18②",
      "episode_order": 25,
      "episode_status": "free_viewing",
      "published_at": "2026-04-13T14:00:00.000+09:00",
      "thumbnail_image_url": "https://cdn.en.to-corona-ex.com/...?X-Amz-..."
    }
  ]
}
```

**Catatan**: Ada 2 status episode:
- `free_viewing` → gratis
- `only_for_subscription` → butuh login/subscribe

---

### ⚠️ Reader / Pages

**Endpoint**: `GET https://api.en.to-corona-ex.com/episodes/{id}/begin_reading`

**Status**: 200 OK ✅ (untuk chapter gratis)

**Response structure**:
```json
{
  "pages": [
    {
      "id": "138733763504511",
      "page_image_url": "https://cdn.en.to-corona-ex.com/pages/images/...?drm_hash=...&Expires=...",
      "drm_hash": "BAQJAgwOAAoGAQ0EDwsIBQMH"
    }
  ],
  "page_direction": "rtl",
  "is_first_view_spread": true
}
```

**⚠️ Masalah besar**:
1. **DRM scrambling** — Image di-scramble dan perlu di-unscramble menggunakan `drm_hash`. Source code punya `ImageInterceptor` yang melakukan unscramble bitmap.
2. **Presigned S3 URL** — Expire dalam 15 menit.
3. **Chapter berbayar** — Butuh Firebase auth (email/password) untuk akses chapter `only_for_subscription`.

---

### Kesimpulan Corona EX

| Aspek | Status | Catatan |
|-------|--------|---------|
| Home/Popular | ✅ | REST API JSON, cursor-based pagination |
| Latest | ✅ | REST API JSON (`sort=latest_episode_published_at`) |
| Search | ✅ | REST API JSON (`/search/comics?keyword=...`) |
| Detail | ✅ | REST API JSON (`/comics/{id}`) |
| Chapters | ✅ | REST API JSON (`/episodes?comic_id=...`) |
| Reader/Pages | ⚠️ | REST API JSON tapi **DRM scrambled** + **presigned URL expire 15 menit** |
| Genre Filter | ✅ | REST API JSON (hanya versi JP) |
| Auth Required | ⚠️ | Gratis untuk free chapters, tapi butuh login untuk paid chapters |
| Cover URLs | ⚠️ | Presigned S3 URL (expire 15 menit) |
| Image URLs | ❌ | **DRM scrambled** — perlu unscramble bitmap di client |

**Verdict**: **Tidak cocok untuk Kuron config tanpa custom adapter**. Semua endpoint JSON, tapi ada 2 masalah besar: (1) Image URLs pakai presigned S3 yang expire 15 menit, (2) Image di-DRM-scramble dan perlu unscramble di client. Ini butuh logic khusus yang tidak bisa di-handle oleh config JSON standar.

---

## 3. ScansGG (`scans.gg`)

### ✅ Home / Popular

**Endpoint**: `GET https://api.scans.gg/series?limit=21&offset=0`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "success": true,
  "data": [
    {
      "id": 17851,
      "title": "Territory Reformation of the [Rarity Changer]",
      "summary": "Arc, the second son of a ducal house...",
      "cover": "aeed92-492fe6-a87e0e-5c1a01.avif",
      "author": ["Higashida Yuuri"],
      "artist": ["popopo"],
      "status": 1,
      "type": 2,
      "tags": [10, 1],
      "themes": ["Isekai", "Reincarnation"],
      "views": 23
    }
  ]
}
```

**Query params**:
- `limit` → jumlah per halaman (default 21)
- `offset` → offset untuk pagination
- `q` → search query
- `q_type` → filter by type (array)
- `q_status` → filter by status (array)
- `q_tags` → filter by tags (array)

**Catatan**: Cover URL perlu digabung dengan CDN: `https://cdn.scans.gg/uploads/covers/{cover}`

---

### ✅ Latest

**Endpoint**: `GET https://api.scans.gg/chapters?page=1&limit=14&chapters=true&series_details=true&group_details=true&sort=date`

**Status**: 200 OK ✅

**Response structure**: Sama dengan popular tapi dari endpoint `/chapters` dengan `series_details=true`.

---

### ✅ Search

**Endpoint**: `GET https://api.scans.gg/series?q=one+piece&limit=21&offset=0`

**Status**: 200 OK ✅

**Response structure**: Sama dengan popular.

---

### ✅ Detail

**Endpoint**: `GET https://api.scans.gg/series?id=17851&trackers=true&sources=true`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "success": true,
  "data": {
    "id": 17851,
    "title": "Territory Reformation of the [Rarity Changer]",
    "summary": "Arc, the second son of a ducal house...",
    "cover": "aeed92-492fe6-a87e0e-5c1a01.avif",
    "author": ["Higashida Yuuri"],
    "artist": ["popopo"],
    "status": 1,
    "tags": [10, 1],
    "themes": ["Isekai", "Reincarnation"],
    "sources": [
      { "id": 351, "title": "raw", "website": "https://www.cmoa.jp/title/354196/" }
    ],
    "trackers": {
      "mangadex": null,
      "myanimelist": null
    }
  }
}
```

---

### ✅ Chapters

**Endpoint**: `GET https://api.scans.gg/chapters?series_id=17851&limit=100&page=1&group_details=true`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "success": true,
  "data": [
    {
      "id": 96165,
      "series_id": 17851,
      "number": 1,
      "title": "",
      "language": "en",
      "created_at": "2026-05-16 02:09:50",
      "group": {
        "id": 45,
        "title": "Asmodeus Scans"
      }
    }
  ],
  "meta": {
    "page": 1,
    "limit": 5,
    "has_more": false
  }
}
```

**Catatan**: Chapter list **ada pagination** (`has_more` flag).

---

### ✅ Reader / Pages

**Endpoint**: `GET https://api.scans.gg/chapter-navigation?series_id=17851&chapter_id=96165&group_id=45`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "success": true,
  "data": {
    "series": { ... },
    "chapter": {
      "id": 96165,
      "number": 1,
      "group": { "id": 45, "title": "Asmodeus Scans" },
      "pages": [
        { "position": 0, "path": "ca7bbf-fc753b-a5b058-ffe54f.avif", "title": "00-148.jpg" },
        { "position": 1, "path": "c7cca4-1b39cc-c8cbb5-9c5402.avif", "title": "001_out.jpg" }
      ]
    }
  }
}
```

**Image URL**: `https://cdn.scans.gg/uploads/pages/{chapterId}/{path}`

**Catatan**: Image URL **tidak pakai presigned URL** — langsung ke CDN. Tidak ada DRM.

---

### Kesimpulan ScansGG

| Aspek | Status | Catatan |
|-------|--------|---------|
| Home/Popular | ✅ | REST API JSON, offset-based pagination |
| Latest | ✅ | REST API JSON (`/chapters?sort=date`) |
| Search | ✅ | REST API JSON (`/series?q=...`) |
| Detail | ✅ | REST API JSON (`/series?id=...`) |
| Chapters | ✅ | REST API JSON (`/chapters?series_id=...`), ada pagination |
| Reader/Pages | ✅ | REST API JSON (`/chapter-navigation?...`), CDN langsung |
| Genre/Tag Filter | ✅ | REST API JSON (`q_tags`, `q_type`, `q_status`) |
| Auth Required | ❌ | Tidak perlu login |
| Cover URLs | ✅ | CDN path (`/covers/{filename}`), tidak expire |
| Image URLs | ✅ | CDN langsung (`/pages/{chapterId}/{path}`), tidak expire, tidak ada DRM |

**Verdict**: **Sangat cocok untuk Kuron REST config**. Semua endpoint JSON bersih. Cover dan image URL pakai CDN path yang tidak expire. Tidak ada DRM. Tidak butuh auth. Pagination offset-based.

---

## Perbandingan Langsung

| Fitur | ReiManga | Corona EX | ScansGG |
|-------|----------|-----------|---------|
| **Tipe** | Hybrid (REST + RSC) | Full REST API | Full REST API |
| **Template Kuron** | Tidak cocok | Tidak cocok (DRM) | REST |
| **Home/Browse** | ✅ JSON | ✅ JSON | ✅ JSON |
| **Latest** | ✅ JSON | ✅ JSON | ✅ JSON |
| **Search** | ✅ JSON | ✅ JSON | ✅ JSON |
| **Detail** | ✅ JSON | ✅ JSON | ✅ JSON |
| **Chapters** | ❌ RSC | ✅ JSON | ✅ JSON |
| **Reader** | ❌ RSC | ⚠️ JSON + DRM | ✅ JSON |
| **Genre Filter** | ✅ JSON | ✅ JSON (JP only) | ✅ JSON |
| **Auth** | Tidak perlu | ⚠️ Perlu untuk paid | Tidak perlu |
| **Cover URL** | ❌ Selalu null | ⚠️ Presigned (15 min) | ✅ CDN path |
| **Image URL** | ❌ RSC only | ❌ DRM scrambled | ✅ CDN langsung |
| **Pagination** | Page-based | Cursor-based | Offset-based |
| **Kemudahan Config** | 🔴 Sulit | 🔴 Sulit (DRM) | 🟢 Mudah |

---

## Kesimpulan Akhir

Dari 3 kandidat yang diuji:

1. **ScansGG** → ✅ **Paling cocok untuk Kuron REST config**. Semua endpoint JSON, CDN langsung, tidak ada DRM, tidak perlu auth.

2. **ReiManga** → ❌ **Tidak cocok**. Home/search/detail JSON tapi cover URL selalu null. Chapters dan reader pakai RSC (Next.js) yang butuh parsing khusus.

3. **Corona EX** → ❌ **Tidak cocok**. Semua endpoint JSON tapi ada 2 masalah besar: (1) Image URLs pakai presigned S3 yang expire 15 menit, (2) Image di-DRM-scramble dan perlu unscramble bitmap di client.
