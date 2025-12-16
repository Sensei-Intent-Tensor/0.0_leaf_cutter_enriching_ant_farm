# 🐜 0.0_leaf_cutter_enriching_ant_farm

**Schema-Driven Web Extraction Library**

A collection of declarative `.ant.yaml` schemas that define *what* to extract from web sources—without coupling to *how* the extraction happens. Drop a schema into any compliant engine (Playwright, Puppeteer, requests) and execute.

---

## 🎯 Philosophy

> "The schema is the intent. The engine is the permission. Execution is the collapse."
> — Intent Tensor Theory

**Ants don't think—they follow trails.** These schemas are pheromone trails: deterministic extraction paths that any compliant engine can follow.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    ANT FARM                             │
│            (This Repository)                            │
│                                                         │
│  schemas/                                               │
│  ├── google_maps/business.ant.yaml                      │
│  ├── yelp/business.ant.yaml                             │
│  ├── linkedin/profile.ant.yaml                          │
│  └── ...                                                │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼ (loaded by)
┌─────────────────────────────────────────────────────────┐
│                   QUEEN LOADER                          │
│        (UI dropdown / manifest.yaml reader)             │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼ (injected into)
┌─────────────────────────────────────────────────────────┐
│                   FORAGER ENGINE                        │
│     (Playwright / Puppeteer / requests executor)        │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                  HARVEST CELL                           │
│            (Structured JSON output)                     │
└─────────────────────────────────────────────────────────┘
```

---

## 📁 Repository Structure

```
0.0_leaf_cutter_enriching_ant_farm/
│
├── README.md                    # You are here
├── ANT_SCHEMA_SPEC.md           # Schema format specification
├── manifest.yaml                # Registry index for Queen Loader
│
├── schemas/                     # 🐜 The Ant Farm
│   ├── google_maps/
│   │   └── business.ant.yaml
│   ├── yelp/
│   │   └── business.ant.yaml
│   ├── linkedin/
│   │   └── profile.ant.yaml
│   ├── reddit/
│   │   └── user.ant.yaml
│   ├── amazon/
│   │   └── product.ant.yaml
│   └── ... (extensible)
│
└── engine/                      # Reference implementation (optional)
    └── README.md
```

---

## 🚀 Quick Start

### 1. Find Your Schema

Browse `/schemas/` by platform.

### 2. Load via Manifest

```yaml
# manifest.yaml lists all available ants
ants:
  - id: google_maps.business
    path: schemas/google_maps/business.ant.yaml
    name: "Google Maps Business"
    description: "Extract business listings from Google Maps"
```

### 3. Execute with Any Engine

The schema tells the engine:
- Where to navigate
- What selectors to use
- How to post-process fields
- How to extract nested data (emails from websites)

---

## 🐜 Ant Colony Terminology

| Term | Meaning |
|------|---------|
| **Ant Farm** | This repository—the schema library |
| **Ant** | A single `.ant.yaml` schema |
| **Queen Loader** | Component that selects which ant to deploy |
| **Nest Selector** | UI dropdown for choosing ants |
| **Forager** | The execution engine (Playwright, etc.) |
| **Chamber** | Execution context |
| **Harvest Cell** | Output display |
| **Trophallaxis Stream** | Result sharing/export |
| **Mandible Bridge** | Interface between schema and engine |

---

## 🔧 Control Tiers

Schemas support three levels of user control:

### Tier 1: Intent Only
```yaml
platform: google_maps
entity: business
search: "{industry} near {city}, {state}"
fields: [name, phone, email, website]
# Engine decides all extraction mechanics
```

### Tier 2: Shape Control
```yaml
fields:
  name: { required: true }
  phone: { postprocess: strip_prefix }
  email: { method: extract_from_website }
# User controls output shape, engine fills mechanics
```

### Tier 3: Full Extraction Control
```yaml
extraction:
  engine: playwright
  selectors:
    name:
      selector: 'h1'
      type: xpath
    email:
      method: crawl_footer_links
      depth: 2
      blacklist: [wixpress.com]
# User controls everything declaratively
```

---

## 🔗 Integration

This library integrates with:

- **[Librarian Protocol OS](https://github.com/FunnelFunction/0.0_git_cloned_librarian_and_universal_protocol_os_coupler)** — Via `Keyless Access` protocol channel
- **[AutoWorkspace Schema Plugins](https://github.com/Sensei-Intent-Tensor/0.0_autoworkspace_schema_model_plugins)** — As keyless schema extension
- **[Leaf Cutter Harvester Ants](https://github.com/Sensei-Intent-Tensor/0.0_leaf_cutter_web_enriching_harvester_ants)** — Python execution engine

---

## 📜 License

MIT License - Use freely, build anything.

---

## 🧬 Parent Theory

**Intent Tensor Theory Institute**  
https://intent-tensor-theory.com

*"Schemas are permissioned collapse—execution occurs when intent aligns with state."*
