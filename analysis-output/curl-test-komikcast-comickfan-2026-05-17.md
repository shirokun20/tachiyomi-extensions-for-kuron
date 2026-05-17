# Uji Curl: Komik Cast vs ComicK Fan

**Tanggal**: 2026-05-17
**Tujuan**: Verifikasi endpoint API real dari 2 kandidat Tier 1

---

## 1. Komik Cast (`be.komikcast.cc`)

### ✅ Home / Popular / Latest

**Konsep**: Di Kuron config, "home" bukan literal homepage website, tapi **endpoint yang return list manga untuk browse**. Bisa endpoint yang sama dengan parameter sorting berbeda.

**Komik Cast**: Satu endpoint `/series` untuk semua — beda `sort` param:

| Mode | Endpoint |
|------|----------|
| Popular | `GET /series?sort=popularity&sortOrder=desc&take=12&page=1` |
| Latest | `GET /series?sort=latest&sortOrder=desc&take=12&page=1` |
| Search | `GET /series?title=one+piece&take=12&page=1` |
| Genre | `GET /series?filter=genreIds=19&sort=popularity&take=12&page=1` |

**Status**: 200 OK ✅

**Response structure** (semua mode sama):
```json
{
  "status": 200,
  "message": "Series retrieved successfully",
  "data": [
    {
      "id": 10029,
      "data": {
        "title": "Otome Game no Mob Desura Naindaga",
        "nativeTitle": "乙女ゲーのモブですらないんだが",
        "slug": "otome-game-no-mob-desura-naindaga",
        "coverImage": "https://minio.imgkc1.my.id/prod/series/.../cover/amyl.webp?X-Amz-...",
        "author": "Gyokuro",
        "rating": 7,
        "status": "ongoing",
        "format": "manga",
        "type": "mirror",
        "genreIds": [22, 28, 6, 26, 14],
        "isHot": false,
        "totalChapters": "1"
      },
      "dataMetadata": {
        "ranking": 5831,
        "dailyViews": 778,
        "bookmarkCount": 32
      }
    }
  ],
  "meta": {
    "total": 10005,
    "page": 1,
    "lastPage": 1001
  }
}
```

**Mapping ke Kuron**:
- `list.items` → `$.data[*]`
- `list.fields.id` → `$.id`
- `list.fields.title` → `$.data.title`
- `list.fields.coverUrl` → `$.data.coverImage`
- `list.fields.description` → `$.data.synopsis` (ada di detail)
- `list.fields.status` → `$.data.status`
- `list.fields.tags` → `$.data.genreIds` (perlu lookup ke genre list)
- `pagination.offsetMode` → `false` (page-based)
- `pagination.total` → `$.meta.lastPage`
- `pagination.pageSize` → `$.meta.total`

**Headers yang dibutuhkan**:
```
Accept: application/json
Origin: https://v2.komikcast.fit
Referer: https://v2.komikcast.fit/
```

**Catatan**: Cover image URL menggunakan **presigned URL S3** (berisi signature yang expire). Ini bisa jadi masalah jika Kuron tidak handle URL yang expire. Perlu cek apakah ada URL tanpa signature.

---

### ✅ Search

**Endpoint**: Sama dengan home, tambah param `title`:
`GET https://be.komikcast.cc/series?page=1&title=one+piece&includeMeta=true&take=12`

**Status**: 200 OK ✅

**Response structure**: Sama persis dengan home.

**Query params**:
- `title` → search query (backend pakai `like` filter)
- `page` → halaman
- `take` → jumlah per halaman (default 12)
- `includeMeta` → `true` untuk dapat pagination info
- `filter` → filter lanjutan, contoh: `genreIds=19`

---

### ✅ Detail

**Endpoint**: `GET https://be.komikcast.cc/series/one-piece`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "status": 200,
  "message": "Series retrieved successfully",
  "data": {
    "id": 5800,
    "data": {
      "slug": "one-piece",
      "title": "One Piece",
      "nativeTitle": "ONE PIECE",
      "author": "Oda Eiichiro",
      "format": "manga",
      "rating": 9,
      "status": "ongoing",
      "synopsis": "Bercerita tentang seorang laki-laki bernama Monkey D. Luffy...",
      "releaseDate": "1997",
      "totalChapters": "1268",
      "coverImage": "https://minio.imgkc1.my.id/.../wanpis113.webp?X-Amz-...",
      "backgroundImage": "https://minio.imgkc1.my.id/.../bg-one-piece.webp?X-Amz-...",
      "folderCdn": "One_Piece",
      "genreIds": [19, 16, 22, 28, 46, 39, 30, 24],
      "genres": [
        { "id": 19, "data": { "name": "Action", "description": "..." } },
        { "id": 16, "data": { "name": "Adventure", "description": "..." } },
        ...
      ]
    },
    "dataMetadata": {
      "ranking": 13,
      "bookmarkCount": 992,
      "totalViewsComputed": 257452
    }
  }
}
```

**Mapping ke Kuron**:
- `detail.fields.title` → `$.data.title`
- `detail.fields.description` → `$.data.synopsis`
- `detail.fields.coverUrl` → `$.data.coverImage`
- `detail.fields.status` → `$.data.status`
- `detail.fields.tags` → `$.data.genres[*].data.name`
- `detail.fields.artists` → `$.data.author`
- `detail.chapters.endpoint` → `/series/{slug}/chapters`

---

### ✅ Chapters

**Endpoint**: `GET https://be.komikcast.cc/series/one-piece/chapters`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "status": 200,
  "message": "Chapters for the series retrieved successfully",
  "data": [
    {
      "id": 398714,
      "data": {
        "slug": null,
        "title": "FIX",
        "index": 1182,
        "isDraft": false,
        "seriesId": 5800,
        "thumbnail": null
      },
      "createdAt": "2026-05-09T11:27:54.154+07:00",
      "views": { "total": 15459 }
    },
    ...
  ]
}
```

**Mapping ke Kuron**:
- `chapters.items` → `$.data[*]`
- `chapters.fields.id` → `$.data.index` (chapter number)
- `chapters.fields.chapterNum` → `$.data.index`
- `chapters.fields.title` → `$.data.title`
- `chapters.fields.date` → `$.createdAt`
- `chapters.fields.url` → `$.data.index` (digabung dengan slug)

**Catatan**: Chapter list **tidak ada pagination** — semua chapter langsung dikembalikan sekaligus. Untuk One Piece ada 1268 chapter dalam satu response.

---

### ✅ Reader / Pages

**Endpoint**: `GET https://be.komikcast.cc/series/one-piece/chapters/1182`

**Status**: 200 OK ✅

**Response structure**:
```json
{
  "status": 200,
  "message": "Chapter retrieved successfully",
  "data": {
    "id": 398714,
    "data": {
      "title": "FIX",
      "images": [
        "https://sv1.imgkc1.my.id/wp-content/img/O/One_Piece/1182FIX/001.jpg",
        "https://sv1.imgkc1.my.id/wp-content/img/O/One_Piece/1182FIX/002.jpg",
        "https://sv1.imgkc1.my.id/wp-content/img/O/One_Piece/1182FIX/003.jpg",
        ...
        "https://sv1.imgkc1.my.id/wp-content/img/O/One_Piece/1182FIX/Marathon.jpg"
      ]
    },
    "chapterIndex": 1182
  }
}
```

**Mapping ke Kuron**:
- `images.mode` → `direct` (array URL langsung)
- `images.selector` → `$.data.images[*]`

**Catatan**: Image URL **tidak pakai presigned URL** — langsung ke CDN `sv1.imgkc1.my.id`. Ini bagus, tidak ada masalah expire.

---

### ✅ Genre / Tag Clicked

**Endpoint**: Sama dengan home, tambah param `filter`:
`GET https://be.komikcast.cc/series?page=1&sort=popularity&sortOrder=desc&take=12&filter=genreIds=19`

**Status**: 200 OK ✅

**Response structure**: Sama persis dengan home/search.

**Catatan**: Genre filter menggunakan **genre ID** (angka), bukan slug. Perlu mapping genre ID → nama genre. Dari response detail, genre ID 19 = "Action".

---

### Kesimpulan Komik Cast

| Aspek | Status | Catatan |
|-------|--------|---------|
| Home/Browse | ✅ | `GET /series?sort=popularity` — REST bersih, pagination page-based |
| Latest | ✅ | `GET /series?sort=latest` — endpoint sama, beda sort param |
| Search | ✅ | `GET /series?title=...` — endpoint sama, tambah title param |
| Detail | ✅ | `GET /series/{slug}` — lengkap: synopsis, genres, author, status |
| Chapters | ✅ | `GET /series/{slug}/chapters` — semua chapter sekaligus (no pagination) |
| Reader/Pages | ✅ | `GET /series/{slug}/chapters/{index}` — array URL langsung, CDN terpisah |
| Genre Filter | ✅ | `GET /series?filter=genreIds=19` — endpoint sama, tambah filter param |
| Auth Required | ❌ | Tidak perlu login |
| Image URLs | ✅ | CDN langsung (`sv1.imgkc1.my.id`), tidak expire |
| Cover URLs | ⚠️ | Presigned S3 URL (expire 24 jam) |

**Satu endpoint `/series` untuk semua mode** — tinggal beda query params (`sort`, `title`, `filter`). Ini ideal untuk Kuron config.

---

## 2. ComicK Fan (`comickfan.com`)

### ❌ Home / Popular / Latest

**Konsep**: Di Kuron config, "home" = endpoint yang return list manga untuk browse. ComicK Fan **tidak punya endpoint JSON** untuk ini.

**Yang ada**:
- `/advanced-search` → **HTML**, bukan JSON
- `/api/comics/{slug}/chapter-list` → JSON, tapi hanya untuk chapter list satu manga tertentu

**Verdict**: Tidak ada endpoint JSON yang bisa dipakai sebagai `allGalleries` / browse entry point di Kuron config. Semua manga listing (home, popular, latest, search, genre) harus di-scrape dari HTML.

---

### ❌ Search

**Endpoint**: `GET https://comickfan.com/advanced-search?name=one+piece&page=1`

**Status**: 200 OK (tapi **HTML**, bukan JSON)

Tidak ada endpoint JSON untuk search manga. Harus scrape dari HTML.

---

### ❌ Detail

**Endpoint**: `GET https://comickfan.com/manga/one-piece`

**Status**: 200 OK (tapi **HTML**, bukan JSON)

Tidak ada endpoint JSON untuk detail manga. Harus scrape dari HTML.

---

### ✅ Chapters

**Endpoint**: `GET https://comickfan.com/api/comics/one-piece/chapter-list?translation_group_id=`

**Status**: 200 OK ✅ (JSON!)

**Response structure**:
```json
{
  "data": [
    {
      "id": 9973052,
      "hash_id": "fCoNWTo1Kw",
      "chapter": "1182",
      "title": "",
      "volume": null,
      "language": "en",
      "published_at": null,
      "group_names": ["Mangakakalot"],
      "chapter_groups": [
        { "group": { "title": "Mangakakalot", "slug": "mangakakalot" } }
      ],
      "upvotes": 1,
      "downvotes": 0,
      "is_final_chapter": false,
      "created_at": "2026-05-08T19:31:11.000000Z",
      "updated_at": "2026-05-16T17:00:57.000000Z"
    },
    ...
  ]
}
```

**Mapping ke Kuron**:
- `chapters.items` → `$.data[*]`
- `chapters.fields.id` → `$.hash_id`
- `chapters.fields.chapterNum` → `$.chapter`
- `chapters.fields.date` → `$.created_at`
- `chapters.fields.url` → `$.hash_id` (digabung dengan slug)

**Catatan**: Ini **satu-satunya endpoint JSON** di ComicK Fan. Yang lain semua HTML scraping.

---

### ❌ Reader / Pages

**Endpoint**: `GET https://comickfan.com/manga/one-piece/chapter-1182-fCoNWTo1Kw`

**Status**: 200 OK (tapi **HTML**, bukan JSON)

Tidak ada endpoint JSON untuk reader pages. Harus scrape dari HTML.

**Image URLs** (contoh real dari HTML):
```
https://meo.cdncmk.com/RwNQRA4gAhoAPDEaF00zHlYGUEQZMgAHCnoxWBJdMR8cXw1RDnZcTA0.webp
https://meo.cdncmk.com/WhhQAgssQh4BICFfC1M2QV4eRgYfNwAeG3g7RAFTIgUeH1AHGSAGWAA6f1QMUSZBRwQYCgIsBBQcNH9ACxU3DVcKGAEDMQ4GGyczGhxAOxQeGEAbH2gbFAMwf1kNFTcfRgBUHEUmAAMLJyEYUAghDgVaDFo.webp
```

**Catatan**: Image URL dari CDN `meo.cdncmk.com`, langsung bisa diakses.

---

### ❌ Genre / Tag Clicked

**Endpoint**: `GET https://comickfan.com/advanced-search?genres=action&sort=latest&page=1`

**Status**: 200 OK (tapi **HTML**, bukan JSON)

Tidak ada endpoint JSON untuk filter genre. Harus scrape dari HTML.

---

### Kesimpulan ComicK Fan

| Aspek | Status | Catatan |
|-------|--------|---------|
| Home/Popular/Latest | ❌ | **Tidak ada endpoint JSON** untuk manga listing |
| Search | ❌ | HTML scraping, bukan JSON |
| Detail | ❌ | HTML scraping, bukan JSON |
| Chapters | ✅ | REST API JSON (`/api/comics/{slug}/chapter-list`) |
| Reader/Pages | ❌ | HTML scraping, bukan JSON |
| Genre Filter | ❌ | HTML scraping, bukan JSON |
| Auth Required | ❌ | Tidak perlu login |
| Image URLs | ✅ | CDN langsung, tidak expire |

**Verdict**: **Tidak cocok untuk REST config**. ComicK Fan hanya punya **1 endpoint JSON** (chapter list), sisanya HTML scraping. Cocok untuk `source-config-scraper-template.json`, bukan REST template.

---

## Perbandingan Langsung

| Fitur | Komik Cast | ComicK Fan |
|-------|-----------|------------|
| **Tipe** | Full REST API | Hybrid (1 API + scraping) |
| **Template Kuron** | REST | Scraper |
| **Home/Browse** | JSON ✅ (`/series?sort=popularity`) | ❌ Tidak ada endpoint JSON |
| **Latest** | JSON ✅ (`/series?sort=latest`) | ❌ Tidak ada endpoint JSON |
| **Search** | JSON ✅ (`/series?title=...`) | ❌ HTML scraping |
| **Detail** | JSON ✅ (`/series/{slug}`) | ❌ HTML scraping |
| **Chapters** | JSON ✅ (`/series/{slug}/chapters`) | JSON ✅ (`/api/comics/{slug}/chapter-list`) |
| **Reader** | JSON ✅ (`/series/{slug}/chapters/{index}`) | ❌ HTML scraping |
| **Genre Filter** | JSON ✅ (`/series?filter=genreIds=...`) | ❌ HTML scraping |
| **Cover URL** | Presigned S3 (expire 24 jam) | HTML scraping |
| **Image URL** | CDN langsung (`sv1.imgkc1.my.id`) | CDN langsung (`meo.cdncmk.com`) |
| **Pagination** | Page-based (`page`, `lastPage`) | HTML pagination |
| **Auth** | Tidak perlu | Tidak perlu |
| **Kemudahan Config** | 🟢 Mudah | 🟡 Sedang |

**Intinya**: Komik Cast punya **satu endpoint `/series`** yang bisa dipakai untuk semua mode (home, latest, search, genre) — tinggal beda query params. ComicK Fan tidak punya endpoint JSON untuk manga listing sama sekali.

---

## Rekomendasi

1. **Komik Cast** → Langsung bisa dikonversi ke `source-config-rest-template.json`. Semua endpoint JSON bersih. Satu endpoint `/series` untuk semua mode (home, latest, search, genre) — tinggal beda query params.

2. **ComicK Fan** → Butuh `source-config-scraper-template.json` karena mayoritas endpoint adalah HTML scraping. Hanya chapter list yang JSON. Tidak ada endpoint JSON untuk manga listing (home/browse/search/genre).
