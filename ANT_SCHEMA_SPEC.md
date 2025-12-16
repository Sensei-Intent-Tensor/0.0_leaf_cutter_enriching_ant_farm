# 🐜 Ant Schema Specification

**Version:** 2.0  
**Extension:** `.ant.yaml` or `.ant.json`

---

## Overview

An Ant Schema is a declarative definition of web extraction intent. It specifies:

1. **Target** — What platform/entity to extract
2. **Search** — How to navigate to results
3. **Pagination** — How to get ALL results (not just first page)
4. **Extraction** — What fields to capture and how
5. **Post-processing** — Transformations on raw data
6. **Nested extraction** — Secondary data retrieval (e.g., emails from websites)
7. **Output** — Column definitions for UI rendering

---

## Schema Structure

```yaml
# Required metadata
schema_version: "2.0"
platform: string          # e.g., "google_maps", "yelp", "linkedin"
entity: string            # e.g., "business", "profile", "product"
description: string       # Human-readable description

# Search configuration
search:
  input_format: string    # Template: "{industry} near {city}, {state}"
  search_url: string      # Base URL for search
  inputs: []              # UI input field definitions

# ═══════════════════════════════════════════════════════════════
# PAGINATION CONFIGURATION (NEW in v2.0)
# ═══════════════════════════════════════════════════════════════
pagination:
  type: string            # "url_param" | "infinite_scroll" | "load_more" | "none"
  results_per_page: int   # How many results per page (platform-specific)
  
  # For URL parameter pagination (Yelp, LinkedIn, Indeed, etc.)
  url_param:
    name: string          # Parameter name: "start", "page", "offset"
    style: string         # "offset" (0,10,20) or "page" (1,2,3)
    
  # For infinite scroll (Google Maps)
  infinite_scroll:
    container: string     # CSS selector for scrollable container
    end_marker: string    # Text or selector indicating end of results
    scroll_delay: int     # ms to wait between scrolls
    max_scrolls: int      # Safety limit on scroll attempts
    
  # User-controllable inputs for UI
  inputs:
    - name: max_results
      label: "Max Results"
      type: number
      default: 20
      min: 1
      max: 500
      description: "Total results to fetch across all pages"
      
# Extraction configuration
extraction:
  engine: string          # "playwright" | "puppeteer" | "requests"
  selectors: {}           # Field extraction selectors
      
# Output configuration
output:
  format: string          # "json" | "csv"
  fields: []              # Column definitions for UI
```

---

## Pagination Types

### Type 1: URL Parameter (`url_param`)

Used by: **Yelp, LinkedIn, Indeed, Yellow Pages, most traditional sites**

```yaml
pagination:
  type: url_param
  results_per_page: 10
  
  url_param:
    name: start           # The URL parameter name
    style: offset         # "offset" = 0,10,20 | "page" = 1,2,3
    
  # How to build paginated URLs:
  # Page 1: ?find_desc=pizza&find_loc=Dallas
  # Page 2: ?find_desc=pizza&find_loc=Dallas&start=10
  # Page 3: ?find_desc=pizza&find_loc=Dallas&start=20
```

**Yelp Example:**
```yaml
pagination:
  type: url_param
  results_per_page: 10
  url_param:
    name: start
    style: offset    # start=0, start=10, start=20...
```

**Yellow Pages Example:**
```yaml
pagination:
  type: url_param
  results_per_page: 30
  url_param:
    name: page
    style: page      # page=1, page=2, page=3...
```

### Type 2: Infinite Scroll (`infinite_scroll`)

Used by: **Google Maps, social media feeds**

```yaml
pagination:
  type: infinite_scroll
  results_per_page: 20    # Approximate results per scroll
  
  infinite_scroll:
    container: 'div[role="feed"]'           # What to scroll
    end_marker: "You've reached the end"    # Stop when this appears
    scroll_delay: 2000                       # Wait 2s between scrolls
    max_scrolls: 50                          # Safety limit
```

### Type 3: Load More Button (`load_more`)

Used by: **Some modern sites, lazy-loading feeds**

```yaml
pagination:
  type: load_more
  results_per_page: 20
  
  load_more:
    button_selector: 'button.load-more'
    max_clicks: 20
    click_delay: 1500
```

### Type 4: None (`none`)

Used when results fit on one page or pagination not needed.

```yaml
pagination:
  type: none
```

---

## Pagination Inputs for UI

The schema declares what pagination controls appear in the Leaf Cutter UI:

```yaml
pagination:
  type: url_param
  results_per_page: 10
  
  # These become input fields in the UI
  inputs:
    - name: max_results
      label: "Max Results"
      type: number
      default: 20
      min: 1
      max: 500
      description: "How many total results to fetch"
      
    - name: max_pages
      label: "Max Pages"  
      type: number
      default: 5
      min: 1
      max: 50
      description: "Maximum pages to scrape (alternative to max_results)"
```

**The Forager engine calculates:**
- If `max_results = 100` and `results_per_page = 10` → scrape 10 pages
- If `max_pages = 5` and `results_per_page = 10` → scrape up to 50 results

---

## Complete Platform Examples

### Yelp Business Schema (v2.0)

```yaml
schema_version: "2.0"
platform: yelp
entity: business_listing
description: "Extract business listings from Yelp search results"

search:
  input_format: "{industry} {city}, {state}"
  search_url: "https://www.yelp.com/search"
  
  inputs:
    - name: industry
      label: "Business Type"
      required: true
    - name: city
      label: "City"
      required: true
    - name: state
      label: "State"
      required: true

pagination:
  type: url_param
  results_per_page: 10
  
  url_param:
    name: start
    style: offset
    
  inputs:
    - name: max_results
      label: "Max Results"
      type: number
      default: 30
      min: 10
      max: 200

extraction:
  engine: playwright
  # ... selectors ...

output:
  fields:
    - name: title
      label: "Business Name"
    - name: rating
      label: "Rating"
    # ... etc ...
```

### Google Maps Business Schema (v2.0)

```yaml
schema_version: "2.0"
platform: google_maps
entity: business_listing
description: "Extract business listings from Google Maps"

search:
  input_format: "{industry} near {city}, {state} {country}"
  search_url: "https://www.google.com/maps"
  
  inputs:
    - name: industry
      label: "Industry/Business Type"
      required: true
    - name: city
      label: "City"
      required: true
    - name: state
      label: "State"
      required: true
    - name: country
      label: "Country"
      default: "USA"

pagination:
  type: infinite_scroll
  results_per_page: 20
  
  infinite_scroll:
    container: 'div[role="feed"]'
    end_marker: "You've reached the end of the list"
    scroll_delay: 2000
    max_scrolls: 25
    
  inputs:
    - name: max_results
      label: "Max Results"
      type: number
      default: 20
      min: 5
      max: 100
      description: "Google Maps limits ~120 results per search"

extraction:
  engine: playwright
  # ... selectors ...

output:
  fields:
    - name: title
      label: "Business Name"
    - name: rating
      label: "Rating"
    # ... etc ...
```

---

## Engine Requirements (v2.0)

A compliant Forager engine MUST:

1. Parse `.ant.yaml` files including pagination config
2. Substitute `{variables}` in `input_format`
3. Navigate to `search_url` with query
4. **Handle pagination according to `pagination.type`:**
   - `url_param`: Loop through pages by incrementing parameter
   - `infinite_scroll`: Scroll container until end_marker or max_scrolls
   - `load_more`: Click button until hidden or max_clicks
5. Respect `max_results` or `max_pages` user input
6. Apply CSS/XPath selectors as specified
7. Execute post-processors in order
8. Return structured JSON matching field names

---

## Output Format

```json
{
  "success": true,
  "query": "pizza near Dallas, TX",
  "pagination": {
    "pages_scraped": 5,
    "results_requested": 50,
    "results_returned": 47
  },
  "results": [
    {
      "title": "Best Pizza Co",
      "rating": "4.8",
      "address": "123 Main St",
      ...
    }
  ]
}
```

---

## Versioning

- **1.x** — Original spec (no pagination)
- **2.0** — Added pagination configuration
- Breaking changes increment major version
- New optional fields increment minor version
