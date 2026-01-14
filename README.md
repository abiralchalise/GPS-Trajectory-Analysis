# GPS-Trajectory-Analysis
Python-based analysis of travel time reliability using GPS data, incorporating statistical distribution fitting, reliability indices.
**Lagankhel ↔ Budhanilkantha Route – Kathmandu**

This project processes raw GPS trajectory data from public buses to support GIS mapping, extract reliable travel times, perform rigorous statistical distribution fitting, analyze segment-level performance, and visualize temporal reliability using the **Planning Time Index (PTI)**.

Developed as part of my final-year project, it demonstrates a complete data science workflow: data preparation, spatial analysis (QGIS), algorithmic trip detection, advanced statistical modeling (distribution fitting & goodness-of-fit), and clear visualization.

## Project Pipeline Overview

1. **Data Preparation**  
   Convert multi-sheet Excel GPS logs (header starting at row 8) → clean CSV files suitable for GIS import.

2. **GIS Spatial Processing** (QGIS)  
   - Create point layers from longitude/latitude (EPSG:4326)  
   - Spatial join with route buffer → flag points `Inside_Route`  
   - Spatial join with segment/stop buffers → assign `Description` (stop name) & `Segment_point` (segment ID)  
   Output: enriched CSVs ready for temporal analysis

3. **Full Trip & Segment Travel Time Extraction**  
   - Identify **complete one-way trips** (terminal-to-terminal: segment 1 → 8 or 8 → 1)  
   - Daytime filter (05:00–22:00)  
   - State-based detection logic for start/end conditions  
   - Separate outputs for each direction (Excel files with departure, arrival, duration)

4. **Segment-level & Half-hourly Analysis**  
   - Extract travel times between adjacent segments  
   - Aggregate trips into 30-minute bins (06:00–20:00)

5. **Statistical Distribution Fitting & Goodness-of-Fit**  
   **Critical step before computing reliability indices**  
   Urban travel times are typically right-skewed → we compare four common skewed distributions:  
   - Log-Normal  
   - Gamma  
   - Weibull  
   - Exponential  

   For each 30-min interval and segment:  
   - Fit parameters using maximum likelihood  
   - Compute empirical mean, std, percentiles (15th, 50th, 95th)  
   - Perform **Anderson–Darling (A-D) goodness-of-fit test** (sensitive to tail behavior)  
   - Determine pass/fail at α = 0.05  
   - Generate visual comparison: histogram + overlaid PDF curves of all four distributions  

   **Output per segment/direction**:  
   - `distribution_fitting_results.xlsx` (parameters, percentiles, A-D stats, pass/fail)  
   - `Histograms/` folder with one PNG per time bin showing actual data vs fitted curves  

   This rigorous analysis **justifies** which distribution (most often Log-Normal) can be reliably used for parametric 95th percentile estimation.

6. **Final Distribution Comparison Summary**  
   Aggregate results across all time intervals for each segment:  
   - % of intervals where distribution **passes** A-D test (acceptable fit)  
   - % of intervals where distribution has the **lowest A-D statistic** (best numerical fit)  

   Output: `distribution_analysis.xlsx` with clear summary table:  
   | Distribution | Cases Pass (%) | Cases 1st (%) |  
   |--------------|----------------|---------------|  
   | Log-Normal   | e.g. 96.4%     | e.g. 78.6%    |  
   | Gamma        | ...            | ...           |  
   | Weibull      | ...            | ...           |  
   | Exponential  | ...            | ...           |  

   → Log-Normal is usually the most consistent and frequently best-fitting model → used for final PTI calculation.

7. **Planning Time Index (PTI) Calculation & Visualization**  
   - PTI = 95th percentile travel time / median travel time  
   - 95th percentile derived from the best-fitting distribution (justified in step 5)  
   - Temporal heatmap showing PTI variation across segments and time intervals  
   - Custom colormap: white (reliable) → green → blue → pink → dark red (unreliable)

## Key Technologies & Tools

- **Python**: pandas, numpy, scipy.stats (distribution fitting & A-D test), glob, os  
- **Visualization**: seaborn, matplotlib (custom colormap + histogram+PDF overlays)  
- **GIS Integration**: QGIS processing algorithms  
- **Environment**: Jupyter Notebook, Python 3.12

## How to View the Work

The notebook is fully rendered on GitHub — including code, detailed explanations, distribution fitting results, goodness-of-fit tables, histogram+PDF plots, and the final PTI heatmap.

→ Open interactive version: [GPS_data_analysis.ipynb](GPS_data_analysis.ipynb)

For quicker reading without clicking through cells, I have also exported the complete notebook (with all outputs rendered) as a PDF:

→ Static PDF: [GPS_data_analysis.pdf](GPS_data_analysis.pdf)


## GIS Preprocessing Requirements

**Important note on QGIS steps:**

The spatial analysis section relies on two **pre-existing layers** in QGIS that must be prepared manually **before** running the notebook:

- `Dissolvedbuffer` (route buffer layer) → located in `day1.gpkg`
- `segment_buffer` (segment/stop buffers layer) → located in `day5.gpkg`

These are **not generated by the code** — they need to be created and dissolved/buffered manually in QGIS using the route shapefile and bus stop/segment points.

Steps summary (done once):
1. Load the bus route shapefile
2. Create a buffer around the route (e.g., 20–50 m) → dissolve to single feature → save as `Dissolvedbuffer`
3. Create individual buffers around each segment/stop → save as `segment_buffer`
4. Place them in the respective GeoPackages as shown in the notebook paths.

Once these layers exist, the notebook performs:
- Point creation from table
- Spatial joins to add `Inside_Route`, `Description`, and `Segment_point`

**The QGIS spatial processing part can be skipped** if you only want to focus on travel time extraction, distribution fitting, PTI calculation, and visualization (using already processed CSVs).

## Notes for Reproducibility

- Paths are currently local → adjust as needed  
- Full raw dataset not included (size/privacy) → use your own files or request sample structure  
- QGIS preprocessing requirements
- Minor dtype warnings can be fixed with explicit dtypes in `pd.read_csv`


## Contact

Questions or suggestions? Open an issue or reach out!

[Abiral Chalise](https://www.linkedin.com/in/abiral-chalise-123704330/)
