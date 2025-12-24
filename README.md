# Milwaukee Parking Ticket Data Analysis

A data analysis project exploring patterns in Milwaukee parking ticket issuance through geocoding, visualization, and exploratory analysis. This project processes historical parking ticket data from 2012 to understand spatial and temporal patterns of parking enforcement across the city.

## About

This project analyzes parking ticket data from Milwaukee, Wisconsin, with a focus on:
- **Data Processing**: Geocoding ticket locations using Milwaukee's address point data from ArcGIS
- **Exploratory Analysis**: Identifying spatial and temporal patterns in ticket issuance
- **Visualization**: Creating interactive maps and time series visualizations to reveal enforcement patterns
- **Future Work**: Time series prediction and modeling of parking enforcement behavior

### Background

This project utilizes two primary parking ticket datasets:
- **2012 Data**: Available from [MKE_2012_Parking_Violations](https://github.com/milwaukeedata/MKE_2012_Parking_Violations), originally compiled by Duncan Solutions (the city's former debt collector and current parking IT infrastructure manager). This data represents the pre-LPR (License Plate Recognition) era.
- **2014-2022 Data**: Obtained via a Freedom of Information Act request filed with the City of Milwaukee in fall 2022, covering the post-LPR era after Milwaukee's Department of Public Works introduced automated plate recognition systems around early 2013.

## Data Sources

- **Parking Tickets**: 
  - [2012 dataset](https://github.com/milwaukeedata/MKE_2012_Parking_Violations) (~750k records)
  - 2014-2022 dataset (FOIA request, fall 2022)
- **Geocoding Reference**: [Milwaukee Address Points](https://gis-mclio.opendata.arcgis.com/datasets/MCLIO::address-points/about) from ArcGIS Open Data
- **Street Network**: [Milwaukee Street Data](https://data.milwaukee.gov/dataset/streets) from Milwaukee Open Data Portal
- **Map Visualization**: Mapbox API for interactive mapping

## Project Structure

```
Milwaukee-Ticket-Data-Analysis/
├── Parking Initial Proccessing (Rosie).ipynb  # Data cleaning and geocoding pipeline
├── Parking Initial Exploration (Rosie).ipynb  # Exploratory analysis and visualizations
├── data/
│   ├── street/                                 # Milwaukee street shapefiles
│   └── tickets_csv/                            # Processed ticket data
└── README.md
```

## Key Findings & Visualizations

### Data Processing Pipeline
- **Geocoding**: Matched ~50% of ticket addresses to coordinates using Milwaukee's address point data
- **Challenge**: Significant address mismatch (~375k unmatched records) suggests address schema changes since 2012
- **Solution**: Dictionary-based geocoding using hash maps for efficient coordinate lookup

### Visualizations Created
1. **3D Spatiotemporal Plot**: Tickets plotted across latitude, longitude, and time
2. **Animated Map**: Daily ticket issuance across the city throughout 2012 (exported as GIF)
3. **Density Heatmap**: Concentration of parking enforcement activity

## Technical Approach

### Technologies Used
- **Python**: Core analysis language
- **Pandas**: Data manipulation and time series handling
- **GeoPandas**: Spatial data processing
- **Plotly**: Interactive map visualizations
- **Matplotlib**: 3D plotting
- **Pandarallel**: Parallel processing for large-scale geocoding

### Key Processing Steps
1. Load and parse ticket data with datetime conversion
2. Geocode addresses using Milwaukee address point data
3. Convert to GeoDataFrame for spatial analysis
4. Generate time series visualizations
5. Create animated maps showing temporal patterns

## Research Context

This project explores approaches to parking ticket prediction, considering:
- **Spatial Factors**: Street segments, blocks, and enforcement routes
- **Temporal Patterns**: Day of week, month, time of day
- **Novelty Detection**: Treating ticket prediction as a one-class classification problem due to extreme class imbalance

### Related Work
- Random Forest approaches from Madison/NYC deep learning studies
- RNN-based time series prediction for Thessaloniki, Greece parking data

## Future Work

### Planned Analyses
- **Time Series Modeling**: Predict ticket likelihood given location and time
- **Feature Engineering**: Extract temporal features (day of week, month, holiday status)
- **Equity Analysis**
- **Advanced Models for Spatio-Temporal ticket prediction**: 
  - One-class SVM/SVDD for novelty detection
  - Spatio-Temporal graph neural networks
  - Generative density estimation methods

### Data Needs
- Vehicle capacity estimates per street segment
- Parking regulation data (time restrictions, permit zones)
- Weather data for correlation analysis with enforcement patterns
- Socioeconomic data for equity analysis

## Installation & Usage

```bash
# Clone the repository
git clone https://github.com/johneckberg/Milwaukee-Ticket-Data-Analysis.git
cd Milwaukee-Ticket-Data-Analysis

# Install required packages
pip install pandas geopandas plotly pandarallel matplotlib kaleido pillow

# Run notebooks
jupyter notebook
```

**Note**: Mapbox token required for interactive map visualizations (set in notebook).

## Data Limitations

- Current analysis focuses on 2012 data; 2014-2022 dataset pending full integration
- ~50% geocoding success rate for 2012 data due to address schema changes
- Missing parking regulation metadata (time restrictions, permit zones)
- Class imbalance: ~1:800 ratio for ticket vs. no-ticket at given locations
- Gap in 2013 data (transition year to LPR systems)

## About

**John Eckberg**  
*Conflict of Interest Declaration*: The author declares a personal bias against parking enforcement.

## Links

- [Milwaukee Street Data](https://data.milwaukee.gov/dataset/streets)


