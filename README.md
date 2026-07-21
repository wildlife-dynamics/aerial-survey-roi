# Aerial Survey ROI Workflow

This workflow generates aerial survey transect lines over a user-defined Region of Interest (ROI). It takes a geospatial boundary file as input and outputs:

- An interactive HTML ecomap showing the survey lines overlaid on the ROI
- The survey lines exported as a GeoParquet (`.parquet`)
- A dashboard widget displaying the map

> **Note:** This workflow only supports **polygon** geometry. Point and line geometry types are not supported.

## Requirements

- A geospatial boundary file (GeoPackage, GeoJSON, or GeoParquet) containing **polygon** or **multipolygon** geometry, reachable by URL or local path
- No EarthRanger or Google Earth Engine connection is required for this workflow

---

## 1. Load the Workflow

Navigate to **Workflow Templates** in the top navigation bar and click **Add Workflow Template**. Paste this repository's URL into the **Github Link** field, then click **Add Template**:

```
https://github.com/wildlife-dynamics/aerial-survey-roi.git
```

Once added, it appears under **Workflow Templates** as `aerial_survey_roi`. Click on the card to open and configure it.

---

## 2. Configure the Workflow

The **Configure New Workflow** page has a left-side navigation panel with several sections.

### Workflow Details

Provide a name and optional description to identify this workflow run. This label will appear in the output dashboard.

| Field | Description |
|-------|-------------|
| Workflow Name *(required)* | e.g., `Mara North Conservancy Aerial Survey` |
| Workflow Description *(optional)* | A short description of the survey run |

### Time Range

Select the timezone and the start/end time for the analysis period. This does not filter the ROI or transect geometry — it's recorded for traceability against a specific survey window.

| Field | Description |
|-------|-------------|
| Timezone | e.g., `Africa/Nairobi (UTC+03:00)` |
| Since / Until | The date and time window to record against this run |

### Basemap Layers

The **Configure base map layers** step sets the two stacked background tile layers on the ecomap. Pre-filled with sensible defaults, but the URL, opacity, and max zoom of each layer are editable.

| Layer | Default Opacity | Max Zoom |
|-------|------------------|----------|
| ESRI World Hillshade | `1.0` | `20` |
| ESRI World Street Map | `0.15` | `20` |

### Define Region of Interest

Upload or link to the boundary file for the area to be surveyed.

> **Important:** The input file must contain **polygon** geometry. Point or line files will not work.

| Input Method | Example |
|---|---|
| Download from URL | `https://www.dropbox.com/scl/fi/14rcy4lkwp7xgewj3xf7k/mnc_conservancy.gpkg?...&dl=0` |
| Local path | `/data/inputs/my_conservancy.gpkg` |

- **Input Method**: Select `Download from URL` or upload a local file
- **URL** *(if downloading)*: Paste the direct URL to your `.gpkg`, `.geojson`, or `.geoparquet` file

### Draw Aerial Survey Lines

Configure how the transect lines are generated across the ROI.

| Parameter | Description | Example |
|---|---|---|
| **Survey Direction (Degrees)** | Compass bearing the survey lines will run from the ROI (0° = North, 90° = East) | `North South` (0°) or `East West` (90°) |
| **Line Spacing (m)** | Distance in metres between adjacent survey lines | `500` |

The transect lines are automatically clipped to the ROI polygon boundary. If the spacing is too large relative to the ROI extent, no lines are generated — see [Troubleshooting](#troubleshooting).

Click **Submit** to run the workflow.

---

## 3. Run the Workflow

Once submitted, the runner will:

1. Load the ROI boundary file into a GeoDataFrame and style it as a map layer (olive green, 25% opacity).
2. Generate parallel transect lines across the ROI's bounding box at the configured direction and spacing, then clip them to the ROI polygon.
3. Reproject both the ROI and the transect lines to EPSG:4326 for display.
4. Render the interactive ecomap (ROI boundary + survey lines over the basemap) and assemble the dashboard widget.
5. Save all outputs to the directory specified by `ECOSCOPE_WORKFLOWS_RESULTS`.

### Output Files

| File | Format | Description |
|---|---|---|
| `survey_lines_<hash>.parquet` | GeoParquet | Survey transect lines in cloud-native format (a content hash is appended to the filename to avoid overwriting previous runs) |
| `aerial_survey.html` | HTML | Interactive ecomap with ROI boundary and survey lines |

The dashboard will display a single map widget titled **"Aerial Survey Lines"** showing:
- The ROI boundary (olive green fill, 25% opacity)
- The generated survey transects (yellow lines, 55% opacity)

---

## Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Workflow fails on ROI input | Input file contains non-polygon geometry | Ensure your file contains only polygon or multipolygon features |
| No lines generated | ROI too small relative to line spacing | Reduce the **Line Spacing** value |
| URL field validation error | URL is not a direct download link | Use a direct file URL (not a preview link) |
| CRS error on load | Source file has no CRS assigned | Set an EPSG code on the file before uploading |

---

## More Help

- **Technical Guide:** [technical_guide/aerial_survey_roi_technical_guide.pdf](technical_guide/aerial_survey_roi_technical_guide.pdf) — pipeline internals and task-by-task reference
- **Issues:** [github.com/wildlife-dynamics/aerial-survey-roi/issues](https://github.com/wildlife-dynamics/aerial-survey-roi/issues)

## Development

This workflow's code (`ecoscope-workflows-aerial-survey-roi-workflow/`) is generated from [`spec.yaml`](spec.yaml) and [`test-cases.yaml`](test-cases.yaml). After editing either file, recompile and commit the generated changes:

```
pixi run --manifest-path pixi.toml --locked bash -c "./dev/recompile.sh --update"
```

## License

[BSD 3-Clause](LICENSE)
