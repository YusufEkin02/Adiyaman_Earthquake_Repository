# Multi-Temporal Assessment of Earthquake-Induced Land Cover Transformation and Vegetation Dynamics in Adıyaman, Türkiye (2022–2026)

**Author:** Yusuf Ekin Özdemir  
**Location:** Adıyaman, Türkiye  
**Tools & Stack:** Google Earth Engine (JavaScript API), Python (`pandas`, `numpy`, `scipy`, `matplotlib`, `seaborn`, `openpyxl`), QGIS, Sentinel-2 MSI Level-2A, SRTM DEM

---

## Overview

This repository contains the geospatial processing scripts, statistical analysis code, summary data tables, 300 DPI cartographic outputs, and full research manuscript for the study:

> **Özdemir, Y. E. (2026).** *Multi-Temporal Assessment of Earthquake-Induced Land Cover Transformation and Vegetation Dynamics in Adıyaman, Türkiye Using Google Earth Engine and Machine Learning.*

Following the February 6, 2023 Kahramanmaraş earthquake sequence ($M_w$ 7.7 and $M_w$ 7.6), Adıyaman Province experienced widespread urban collapse, debris clearance, temporary container settlement deployment, and large-scale post-disaster mass-housing (TOKİ) reconstruction. This project quantifies both immediate (**2022–2023**) and medium-term (**2022–2026**) Land Use/Land Cover (LULC) transitions and vegetation dynamics across all nine administrative districts of Adıyaman (**750,143.09 ha**).

---

## Methodology

1. **Phenologically Matched Satellite Composites (GEE):**
   * **Dataset:** Atmospherically corrected Copernicus Sentinel-2 Level-2A surface reflectance (`COPERNICUS/S2_SR_HARMONIZED`) at **10 m** spatial resolution (`EPSG:32637`).
   * **Temporal Windows:** Peak vegetative window (**May 1 – June 30**) for **2022** (pre-earthquake baseline), **2023** (post-earthquake year 1), and **2026** (reconstruction phase), masked for clouds and shadows via `QA60` and Scene Classification Layer (`SCL`).
2. **16-Feature Classification Stack:**
   * **9 Spectral Bands:** Blue (`B2`), Green (`B3`), Red (`B4`), Red-Edge 1–3 (`B5`, `B6`, `B7`), Near-Infrared (`B8`), and SWIR 1–2 (`B11`, `B12`).
   * **5 Spectral Indices:** `NDVI`, `SAVI`, `NDBI`, `MNDWI`, and `BSI`.
   * **2 Topographic Ancillary Variables:** Elevation (`ELEV`) and Terrain Slope (`SLOPE`) derived from the 30 m SRTM Digital Elevation Model (`USGS/SRTMGL1_003`).
3. **Machine Learning & Accuracy Assessment:**
   * Trained a **150-tree Random Forest** classifier (`ee.Classifier.smileRandomForest`) across six LULC classes (*Water Bodies*, *Forest & Orchards*, *Cropland*, *Rangeland / Steppe*, *Built-up Area*, and *Bare Soil / Debris*) using stratified sampling (70% training, `n = 1,052`; 30% independent validation, `n = 448`; seed `42`).
4. **Post-Classification & Statistical Analysis (Python & QGIS):**
   * Computed pixel-level cross-tabulation transition matrices (hectares) for `2022–2023` and `2022–2026`.
   * Evaluated zonal spectral index shifts across all nine districts (*Merkez, Besni, Çelikhan, Gerger, Gölbaşı, Kahta, Samsat, Sincik, Tut*) using paired-sample $t$-tests and Wilcoxon signed-rank tests ($\alpha = 0.05$).

---

## Key Findings

* **High Classification Performance:** The 16-feature Random Forest model achieved an **Overall Accuracy of 89.06%** (399/448 validation pixels) and a **Cohen's Kappa coefficient of 0.8687**.
* **Two-Stage Built-up Area Trajectory:** Net *Built-up Area* decreased by **14,470.09 ha** (-16.15%, from `89,619.11 ha` in 2022 to `75,149.02 ha` in 2023) due to building collapse and demolition, before rebounding by **+6,989.32 ha** to `82,138.34 ha` in 2026 during post-disaster reconstruction.
* **Agricultural Encroachment During Shelter & Housing Expansion:** Cross-tabulation matrices revealed that **12,344.03 ha** of *Cropland* transitioned into *Built-up Area* in `2022–2023` (emergency container cities and staging hubs), expanding to **18,610.17 ha** by `2026` (alongside **5,309.06 ha** from *Forest & Orchards*) due to permanent mass-housing projects.
* **Reservoir Drawdown & Shoreline Colonization:** *Water Bodies* (primarily the Atatürk Dam reservoir) contracted by **11,727.12 ha** (`2022–2023`) and **39,060.30 ha** (`2022–2026`), exposing **38,111.80 ha** of shoreline mudflats colonized by herbaceous vegetation and seasonal crops.
* **Urban Epicenter Suppression vs. Regional Greening:** While regional spring moisture and reservoir drawdown elevated province-wide mean NDVI from `0.2314` (2022) to `0.2627` (2023, $p = 0.00043$) and `0.3034` (2026, $p = 0.00007$), **Merkez District**—the epicenter of urban devastation—recorded the lowest immediate post-earthquake NDVI gain (**+0.0042**, from `0.2178` to `0.2221`). All four indices (`NDVI`, `SAVI`, `NDBI`, `BSI`) showed statistically significant shifts ($p < 0.012$).

---

## Repository Structure

```text
├── gee/                 # Google Earth Engine JavaScript code (compositing, RF classification, transitions)
├── python/              # Python scripts for statistical testing (paired t-test, Wilcoxon) & 300 DPI plots
├── qgis/                # 300 DPI cartographic layouts and figures (Fig1_LULC_TimeSeries.png – Fig5_LULC_Gecis_Matrisi_Heatmap.png)
├── tables/              # Summary tables, confusion matrix, transition matrices, and district statistics (Makale_Analiz_Tablolari.xlsx)
├── paper/               # Full research manuscript (Adiyaman_Earthquake_LULC_Article.pdf)
├── LICENSE              # MIT License
└── README.md            # Project documentation
