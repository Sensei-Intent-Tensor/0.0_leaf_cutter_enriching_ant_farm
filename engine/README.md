# 🐜 Forager Engine

**Reference Implementation Placeholder**

This folder is reserved for reference engine implementations that can execute `.ant.yaml` schemas.

## Supported Engines

The Ant Farm schemas are designed to work with:

- **Playwright** (Node.js / Python)
- **Puppeteer** (Node.js)
- **Selenium** (Multi-language)
- **requests + BeautifulSoup** (Python, for static pages)

## Integration Points

### Librarian Protocol OS
The primary integration is via the [Librarian Protocol OS Coupler](https://github.com/FunnelFunction/0.0_git_cloned_librarian_and_universal_protocol_os_coupler), which provides:

- Schema loading via dropdown (Queen Loader)
- Execution UI (Chamber)
- Result display (Harvest Cell)

### Standalone CLI
```bash
# Future: standalone runner
ant-forager run --schema google_maps.business --input keywords.json
```

### API Service
```bash
# Future: hosted API
POST /api/harvest
{
  "schema": "google_maps.business",
  "inputs": {
    "industry": "tattoo",
    "city": "Dallas",
    "state": "TX"
  }
}
```

## Engine Contract

A compliant Forager engine MUST:

1. Parse `.ant.yaml` schema files
2. Substitute `{variables}` in search patterns
3. Navigate to target URLs
4. Apply CSS/XPath selectors as specified
5. Execute post-processors
6. Handle nested extraction (email from website)
7. Return structured JSON matching schema fields

See [ANT_SCHEMA_SPEC.md](../ANT_SCHEMA_SPEC.md) for full specification.
