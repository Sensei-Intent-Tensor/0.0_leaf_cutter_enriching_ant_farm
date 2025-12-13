# 🐜 Ant Schema Specification

**Version:** 1.0  
**Extension:** `.ant.yaml` or `.ant.json`

---

## Overview

An Ant Schema is a declarative definition of web extraction intent. It specifies:

1. **Target** — What platform/entity to extract
2. **Search** — How to navigate to results
3. **Extraction** — What fields to capture and how
4. **Post-processing** — Transformations on raw data
5. **Nested extraction** — Secondary data retrieval (e.g., emails from websites)

---

## Schema Structure

```yaml
# Required metadata
schema_version: "1.0"
platform: string          # e.g., "google_maps", "yelp", "linkedin"
entity: string            # e.g., "business", "profile", "product"
description: string       # Human-readable description

# Search configuration
search:
  input_format: string    # Template: "{industry} near {city}, {state}"
  search_url: string      # Base URL for search
  
# Extraction configuration
extraction:
  engine: string          # "playwright" | "puppeteer" | "requests"
  selectors:
    field_name:
      selector: string    # CSS or XPath selector
      type: string        # "css" | "xpath"
      attribute: string   # Optional: extract attribute instead of text
      postprocess: string # Optional: transformation instruction
      
# Nested extraction (optional)
email_extraction:
  type: string            # "regex" | "api"
  pattern: string         # Regex pattern
  blacklist_domains: []   # Domains to ignore
  disallow_extensions: [] # File extensions to skip
  disallow_starts_with_digit: boolean

# Output metadata
metadata:
  scraped_at:
    type: string
    format: datetime
    auto: true
```

---

## Field Types

### Basic Field
```yaml
name:
  selector: 'h1.business-name'
  type: css
```

### Field with Attribute
```yaml
website:
  selector: 'a.website-link'
  type: css
  attribute: href
```

### Field with Post-processing
```yaml
phone:
  selector: 'button[aria-label^="Phone:"]'
  type: css
  postprocess: strip_prefix:Phone:
```

### Nested Extraction Field
```yaml
email:
  method: extract_email_from_website
  url_field: website      # Use value from 'website' field
```

---

## Post-processors

| Processor | Syntax | Effect |
|-----------|--------|--------|
| `strip_prefix` | `strip_prefix:Phone:` | Removes "Phone:" from start |
| `strip_suffix` | `strip_suffix:)` | Removes ")" from end |
| `regex_extract` | `regex_extract:\d+` | Extracts matching pattern |
| `to_lowercase` | `to_lowercase` | Lowercases entire value |
| `trim` | `trim` | Removes whitespace |

---

## Selector Types

### CSS Selectors
```yaml
type: css
selector: 'button[aria-label^="Phone:"]'
```

### XPath Selectors
```yaml
type: xpath
selector: '//*[@id="QA0Szd"]/div/div/div[1]/div[3]/div/div[1]/div/div/div[2]/div[2]/div/div[1]/div[1]/h1'
```

---

## Email Extraction Configuration

```yaml
email_extraction:
  type: regex
  pattern: "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}"
  blacklist_domains:
    - example.com
    - domain.com
    - wixpress.com
  disallow_extensions:
    - png
    - jpg
    - jpeg
    - pdf
    - js
    - css
    - svg
    - webp
  disallow_starts_with_digit: true
```

---

## Complete Example

```yaml
schema_version: "1.0"
platform: google_maps
entity: business_listing
description: "Extract business data from Google Maps search results"

search:
  input_format: "{industry} near {city}, {state} {country}"
  search_url: "https://www.google.com/maps?hl=en"

extraction:
  engine: playwright
  selectors:
    title:
      selector: '//*[@id="QA0Szd"]/div/div/div[1]/div[3]/div/div[1]/div/div/div[2]/div[2]/div/div[1]/div[1]/h1'
      type: xpath
    category:
      selector: '//*[@id="QA0Szd"]/div/div/div[1]/div[3]/div/div[1]/div/div/div[2]/div[2]/div/div[1]/div[2]/div/div[2]/span[1]/span/button'
      type: xpath
    rating:
      selector: '//*[@id="QA0Szd"]/div/div/div[1]/div[3]/div/div[1]/div/div/div[2]/div[2]/div/div[1]/div[2]/div/div[1]/div[2]/span[1]/span[1]'
      type: xpath
    total_reviews:
      selector: '//*[@id="QA0Szd"]/div/div/div[1]/div[3]/div/div[1]/div/div/div[2]/div[2]/div/div[1]/div[2]/div/div[1]/div[2]/span[2]/span/span'
      type: xpath
      postprocess: strip_chars:()
    address:
      selector: 'button[aria-label^="Address:"]'
      type: css
      postprocess: strip_prefix:Address:
    phone:
      selector: 'button[aria-label^="Phone:"]'
      type: css
      postprocess: strip_prefix:Phone:
    website:
      selector: 'a.CsEnBe'
      type: css
      attribute: href
    email:
      method: extract_email_from_website
      url_field: website

email_extraction:
  type: regex
  pattern: "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}"
  blacklist_domains:
    - example.com
    - domain.com
    - wixpress.com
  disallow_extensions:
    - png
    - jpg
    - pdf
  disallow_starts_with_digit: true

metadata:
  scraped_at:
    type: string
    format: datetime
    auto: true
```

---

## Engine Requirements

A compliant Forager engine MUST:

1. Parse `.ant.yaml` files
2. Substitute `{variables}` in `input_format`
3. Navigate to `search_url` with query
4. Apply CSS/XPath selectors as specified
5. Execute post-processors in order
6. Handle nested extraction methods
7. Return structured JSON matching field names

---

## Output Format

```json
[
  {
    "title": "Dallas Tattoo Co",
    "category": "Tattoo Shop",
    "rating": "4.8",
    "total_reviews": "234",
    "address": "123 Main St, Dallas, TX",
    "phone": "+1 214-555-1234",
    "website": "https://dallastattoo.com",
    "email": "info@dallastattoo.com",
    "scraped_at": "2025-12-13T06:15:00Z"
  }
]
```

---

## Versioning

Schema versions follow semver:
- **1.x** — Current stable
- Breaking changes increment major version
- New optional fields increment minor version
