# CSS/OSS Measurement System for Gyratory Crushers

## Overview

This project provides an automated system for measuring **Close Side Setting (CSS)** and **Open Side Setting (OSS)** of gyratory crushers using LiDAR point cloud data. These measurements are critical for optimizing crusher performance and maintaining consistent product size.

### What is CSS/OSS?

- **CSS (Close Side Setting)**: The minimum gap between the mantle (rotating cone) and the concave (stationary wall) during the crusher's gyrating motion.
- **OSS (Open Side Setting)**: The maximum gap between these surfaces.
- **Eccentricity**: The difference between OSS and CSS, indicating the offset of the mantle's rotation axis.

## Key Features

- **Automatic Processing**: No manual parameter tuning required for each scan
- **Absolute Height Wall Detection**: Robust detection method that ignores floor noise
- **Interactive 3D Visualization**: Plotly-based HTML visualizations
- **Batch Processing**: Process multiple scans at once with a unified dashboard
- **CSV Export**: Results exported for further analysis

## Algorithm (v9 - Final)

The algorithm uses a surface-based approach with Digital Elevation Modeling (DEM):

1. **Data Loading**: Read XYZ coordinates from CSV files
2. **Axis Alignment**: 90° Y-rotation and origin normalization
3. **DEM Construction**: Interpolate point cloud onto regular grid
4. **Peak Detection**: Automatic identification of mantle apex
5. **Floor Detection**: Histogram-based floor level identification
6. **Mantle Edge Detection**: Gradient-based ray casting from peak
7. **Wall Detection**: Height threshold method (key innovation)
8. **Contour Smoothing**: Circle fitting + Savitzky-Golay filter
9. **CSS/OSS Calculation**: Find minimum and maximum gaps

### Key Innovation: Height Threshold Wall Detection

Traditional gradient-based wall detection can be fooled by floor noise. Our approach detects the wall where the surface height exceeds a configurable threshold (default: 35cm), providing robust detection regardless of floor conditions.

## Installation

### Prerequisites

- Python 3.8 or higher
- pip package manager

### Setup

```bash
# Clone or download the project
cd "Final Folder with dashboard"

# Create virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate  # Linux/macOS
# venv\Scripts\activate   # Windows

# Install dependencies
pip install -r requirements.txt
```

## Usage

### Single File Analysis

To analyze a single LiDAR scan:

```bash
# Edit test2.py to set your CSV file path (line ~95)
python test2.py
```

This will:
- Process the specified CSV file
- Display interactive 3D visualizations
- Print CSS/OSS measurements to console

### Batch Processing

To process multiple files and generate a dashboard:

```bash
# Ensure all Test*.csv files are in the current directory
python batch_analyzer2.py
```

This will:
- Process all `Test*.csv` files in the directory
- Generate individual HTML visualizations in `visualizations/`
- Create an interactive dashboard at `visualizations/index.html`
- Export results to `batch_analysis_results_v9.csv`

### Viewing the Dashboard

After running batch_analyzer2.py:

```bash
cd visualizations
python3 -m http.server 8080
```

Then open http://localhost:8080 in your web browser.

## File Structure

```
Final Folder with dashboard/
├── test2.py                    # Single file analysis (v9 algorithm)
├── batch_analyzer2.py          # Batch processing with dashboard
├── requirements.txt            # Python dependencies
├── README.md                   # This file
├── Test1.csv ... Test30.csv    # LiDAR scan data files
├── Test1.pcd ... Test30.pcd    # Point cloud files (optional)
├── batch_analysis_results_v9.csv  # Batch processing results
└── visualizations/
    ├── index.html              # Interactive dashboard
    └── Test*_visualization.html # Individual 3D views
```

## Data Format

### Input CSV Format

CSV files should contain at minimum:

```csv
X,Y,Z
0.123,0.456,0.789
0.234,0.567,0.890
...
```

Additional columns (e.g., reflectivity) are ignored.

### Output Format

The batch analysis CSV contains:

| Column | Description |
|--------|-------------|
| filename | Source CSV file |
| num_points | Number of points in scan |
| css | Close Side Setting (cm) |
| oss | Open Side Setting (cm) |
| mean_gap | Average gap (cm) |
| std_gap | Standard deviation of gaps (cm) |
| eccentricity | OSS - CSS (cm) |
| coverage | Percentage of valid wall points |
| peak_height | Mantle peak height (cm) |
| floor_z | Detected floor level (cm) |
| visualization | Path to HTML visualization |

## Configuration

### Wall Height Threshold

The primary configurable parameter is `WALL_HEIGHT_THRESHOLD` in both scripts:

```python
WALL_HEIGHT_THRESHOLD = 0.35  # meters (35 cm)
```

- **Lower value**: Wall detected closer to mantle due to the noise in our prototype 
- **Higher value**: Wall detected further from mantle

Adjust this based on your crusher's geometry and the height of the concave wall.

## Visualization Features

### Dashboard (index.html)

- Summary statistics (average CSS/OSS, total points)
- Searchable file list
- Interactive 3D viewer with embedded Plotly figures
- Keyboard navigation (arrow keys)

### Individual Visualizations

- 3D surface plot of the DEM
- Mantle and wall contours
- CSS (minimum gap) and OSS (maximum gap) indicator lines
- Gap measurement lines at 15° intervals
- Polar plots for contour and gap distribution

## Troubleshooting

### No wall points detected

If you see "No wall points detected", try:
- Lowering `WALL_HEIGHT_THRESHOLD`
- Verify the LiDAR scan captures the full crusher geometry

### Poor mantle detection

Ensure:
- The mantle apex is within the scan area
- There are sufficient points around the mantle surface

### Visualization not loading

- Check that all HTML files are in the `visualizations/` folder
- Ensure you're running a local web server (see Usage section)

## Technical Details

### Dependencies

| Package | Purpose |
|---------|---------|
| numpy | Array operations, linear algebra |
| pandas | CSV reading, data manipulation |
| scipy | Interpolation, Gaussian filtering, Sobel operator, Savitzky-Golay filter |
| plotly | Interactive 3D visualization |


## Authors

CSS/OSS Measurement Project Team
Behlah Katleriwala, Anjali Singh, Mayank Mehan, Yaash Singh


## License

This project is provided for educational and research purposes.

