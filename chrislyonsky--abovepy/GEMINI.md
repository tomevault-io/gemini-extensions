## abovepy

> This is a Python package for KyFromAbove STAC data access. GPL-3.0 licensed.

# abovepy — Copilot Instructions

This is a Python package for KyFromAbove STAC data access. GPL-3.0 licensed.

Key conventions:
- Python 3.10+, NumPy docstrings, type hints on all public functions
- Product is a parameter to search(), NOT a separate module
- All bbox inputs accept EPSG:4326, convert to EPSG:3089 internally
- No AWS credentials required — public bucket
- Optional deps (pdal, laspy, boto3) must be lazy-imported inside functions
- httpx for HTTP, rasterio for rasters, geopandas for GeoDataFrames
- VRT is the default mosaic output

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrislyonsKY)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chrislyonsKY)
<!-- tomevault:4.0:gemini_md:2026-04-08 -->
