# SMBUD — MongoDB & Elasticsearch Analytics
## Overview
Two focused analytics tracks on unstructured data:

1. **High‑Profile Travel Planner (MongoDB, Yelp ~150k)** — decision‑oriented pipelines (open‑now, rating thresholds, tie‑breakers): dentists on Sunday, ≥4.5★ restaurants by state, best pub per state, shopping cities, late‑hour auto repair, pet/kid‑friendly lunch, luxury dog‑friendly hotels.
2. **Anime Streaming Analytics (Elasticsearch, ~24.9k)** — full‑text + aggregations for catalog strategy: studio × status averages, seasonal/year trends (SQL‑on‑ES), yearly top‑10s (Score ≥7), completion ratios, and theme slices (synopsis search).

**Why MongoDB + Elasticsearch?** MongoDB fits flexible JSON + fast pipelines; Elasticsearch excels at text relevance and scalable aggregations.

---

## Datasets
- **Yelp businesses** JSON subset (businesses only). Keep raw under `data/` and out of VCS.
- **Anime dataset** (e.g., MyAnimeList snapshot) as CSV/NDJSON.

> Datasets are **not** redistributed. Place them locally before running.

---

## Tech Stack
- **Databases:** MongoDB, Elasticsearch (8.x), Kibana (optional)
- **Querying:** MongoDB Aggregation Framework; Elasticsearch DSL & SQL endpoint
- **Artifacts:** JSON/NDJSON, `.js`/`.json` query specs, Markdown report/plots

---

## Quickstart

### Prerequisites
- MongoDB (local/Atlas)
- Elasticsearch (8.x) + Kibana (optional)
- Node/CLI or preferred runner for `.js` query files

### 1) MongoDB setup (Yelp)
1. **Import** businesses JSON
   ```bash
   mongoimport --db smbud --collection businesses \
     --file yelp_academic_dataset_business.json --jsonArray
   ```
2. **Indexes** (examples)
   ```js
   db.businesses.createIndex({ city: 1, state: 1 })
   db.businesses.createIndex({ categories: "text" })
   db.businesses.createIndex({ "hours.Sunday": 1, is_open: 1, stars: 1 })
   db.businesses.createIndex({ stars: -1, review_count: -1 })
   ```
3. **Run aggregations**
   Use `.js` pipelines under `mongodb/queries/` (dentists Sunday; restaurants ≥4.5★ by state; best pub per state; late‑hour auto repair).

### 2) Elasticsearch setup (Anime)
1. **Create index + mapping (sketch)**
   ```json
   PUT /anime
   {
     "mappings": {
       "properties": {
         "anime_id":   { "type": "long"   },
         "name":       { "type": "text"   },
         "english_name": { "type": "text" },
         "other_name": { "type": "keyword"},
         "score":      { "type": "double" },
         "genres":     { "type": "text"   },
         "synopsis":   { "type": "text"   },
         "type":       { "type": "keyword"},
         "episodes":   { "type": "keyword"},
         "aired":      { "type": "text"   },
         "premiered":  { "type": "keyword"},
         "status":     { "type": "keyword"},
         "producers":  { "type": "text"   },
         "licensors":  { "type": "text"   },
         "studios":    { "type": "keyword"},
         "source":     { "type": "keyword"},
         "duration":   { "type": "keyword"},
         "rating":     { "type": "keyword"},
         "rank":       { "type": "keyword"},
         "popularity": { "type": "long"   },
         "favourites": { "type": "long"   },
         "scored_by":  { "type": "double" },
         "members":    { "type": "long"   },
         "image_url":  { "type": "keyword"}
       }
     }
   }
   ```
2. **Ingest CSV/NDJSON** (Logstash, `_bulk`, or Kibana). Ensure field types match.
3. **Run analytics**: DSL aggregations for studio × status; SQL endpoint for season/year trends; top‑hits per year (Score ≥7); completion ratios; synopsis `match` filters.

---

## Example Queries

**MongoDB — Restaurants ≥4.5★ by state (open)**
```js
db.businesses.aggregate([
  { $match: { categories: { $regex: /restaurant/i }, is_open: 1, stars: { $gte: 4.5 } } },
  { $group: { _id: "$state", restaurant_count: { $sum: 1 }, average_reviews: { $avg: "$review_count" } } },
  { $sort: { restaurant_count: -1 } },
  { $limit: 3 }
])
```

**Elasticsearch — Avg score by studio × status**
```json
GET /anime/_search
{
  "size": 0,
  "aggs": {
    "by_studio": {
      "terms": { "field": "studios" },
      "aggs": {
        "by_status": {
          "terms": { "field": "status" },
          "aggs": { "avg_score": { "avg": { "field": "score" } } }
        }
      }
    }
  }
}
```

---

## Repository Structure (suggested)
```
.
├─ mongodb/
│  ├─ queries/            # .js aggregation pipelines
│  └─ notebooks/          # optional EDA/plots
├─ elasticsearch/
│  ├─ mappings/           # index mappings
│  ├─ queries/            # DSL & SQL examples
│  └─ notebooks/          # optional EDA/plots
├─ docs/
│  └─ report.pdf          # course deliverable (optional)
└─ README.md
```

---

## Reproducibility & Testing
- Parametrize date/time filters and thresholds (env or small config files).
- Keep queries idempotent and versioned (one query = one file).
- Add lightweight checks (counts, nulls, type validation) pre/post pipeline.

---

## Authors & Roles
**Patrizio Acquadro, Tito Nicola Drugman, David Gadiaga**
MSc Computer Science - Artificial Intelligence (T2i)
Milan, Politecnico di Milano

---

## License
Course deliverable — All rights reserved.
