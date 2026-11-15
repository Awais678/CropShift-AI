# 🌾 CropShift AI: Climate-Resilient Crop Rotation Intelligence

**An Earth-Observation-Powered Deep Learning Decision Support System for Smallholder Farmers**

## 📖 Overview

**CropShift AI** bridges satellite telemetry with multi-objective climate adaptation analytics to help smallholder farmers optimize crop rotations and build climate resilience. Demonstrated in the agricultural belt of **Hyderabad and Jamshoro, Sindh, Pakistan (MGRS Tile T43RDR)**, the platform moves beyond static crop health monitoring to answer a critical question: *What crop should be planted next, and why?*

By combining 30-meter multispectral satellite imagery, agroclimatological records, and soil moisture context, CropShift AI evaluates alternative rotation scenarios (such as introducing nitrogen-fixing mung beans or drought-resilient sorghum alongside cotton and wheat) against water availability, extreme heat exposure, soil health, and economic priorities.

## 🎯 The Challenge

Agricultural regions in Sindh face converging environmental and agronomic pressures:

* **Extreme Thermal Stress:** Peak summer temperatures regularly exceed $39^\circ\text{C}$, threatening flowering and boll retention in traditional crops.

* **Water Scarcity & Rationing:** High irrigation demand during Kharif and Rabi seasons collides with constrained canal water allocations.

* **Soil Nitrogen Depletion:** Continuous cotton monoculture degrades soil organic matter and nitrogen reserves, increasing reliance on costly synthetic fertilizers.

## 🛰️ NASA & Partner Data Sources

CropShift AI fuses multi-scale Earth observation datasets to capture both field-level crop dynamics and regional climate constraints:

| Dataset | Source / Mission | Resolution | Role in CropShift AI | 
| ----- | ----- | ----- | ----- | 
| **Copernicus Sentinel-2 (`HLSS30`)** | NASA Harmonized Landsat Sentinel-2 (LPCLOUD) | $30\text{ m}$ | Surface reflectance (Red, NIR, SWIR1) used to compute $\text{NDVI}$ and $\text{NDMI}$ vegetation/moisture indices. | 
| **NASA POWER** | Agroclimatology & Meteorological Data | Regional Daily | Historical daily temperature ($T_{\text{max}}$, $T_{\text{mean}}$), precipitation, and agro-meteorological profiles. | 
| **SMAP** | Soil Moisture Active Passive | $\sim 9\text{ km}$ | Root-zone soil wetness context to identify drought stress and water infiltration limits. | 

> **Spatial Scale Transparency:** All dashboard visualizations explicitly disclose the spatial resolution mismatch between $30\text{ m}$ HLS field-level segmentation and $\sim 9\text{ km}$ SMAP regional soil moisture estimates.

## 🧠 Technical Architecture

The end-to-end pipeline integrates deep learning spatial-temporal modeling with interpretable gradient-boosted decision scoring:

1. **GeoTIFF Ingestion & Preprocessing (`data_pipeline.py`):**
   * Ingests real NASA `HLSS30` GeoTIFF bands (`B04` Red, `B8A`/`B08` NIR, `B11` SWIR1) or generates realistic synthetic regional patches for rapid prototyping.
   * Applies scale factors, masks clouds/nodata values, and computes normalized indices:
     
     $$\text{NDVI} = \frac{\text{NIR} - \text{Red}}{\text{NIR} + \text{Red}}, \qquad \text{NDMI} = \frac{\text{NIR} -    \text{SWIR1}}{\text{NIR} + \text{SWIR1}}
     $$

2. **Spatial Crop Segmentation (`CropSegFormer`):**
   * Transformer-based semantic segmentation backbone classifying $30\text{ m}$ pixels into active agricultural classes (Cotton, Wheat, Legumes/Fodder, Bare Soil/Fallow).

3. **Spatiotemporal Stress Detection (`SeasonalStressConvLSTM`):**
   * Convolutional LSTM network analyzing multi-temporal vegetation and moisture sequences across growing stages to map canopy thermal and water stress probabilities.

4. **Multi-Objective Rotation Scoring (`XGBoost` + Agronomic Rules):**
   * Evaluates candidate rotations (*Continuous Cotton*, *Cotton* $\rightarrow$ *Wheat*, *Cotton* $\rightarrow$ *Mung Bean*, *Cotton* $\rightarrow$ *Sorghum*) using farmer-adjustable priority weights for **Water Efficiency**, **Soil Health**, **Thermal Resilience**, and **Net Profit (PKR/acre)**.

## 🚀 Running in Google Colab with Ngrok

To launch the **CropShift AI** interactive Streamlit dashboard directly in your browser via Google Colab, execute the following steps:

1. **Install Dependencies:**
   In your Colab cell, install the required geospatial and machine learning libraries:
   ```python
   !pip install -q streamlit folium streamlit-folium plotly xgboost rasterio pyproj transformers pyngrok
   ```

2. **Clone the Repository & Start Streamlit via Ngrok:**
   Configure your free Ngrok authtoken and launch the app:
   ```python
   !git clone https://github.com/cropshift-ai/cropshift-sindh-prototype.git
   %cd cropshift-sindh-prototype/cropshift_ai

   from pyngrok import ngrok
   
   # Replace with your actual authtoken from dashboard.ngrok.com
   ngrok.set_auth_token("YOUR_NGROK_AUTHTOKEN_HERE")
   
   # Start Streamlit in the background
   !streamlit run app.py --server.port 8501 &>/content/streamlit.log &
   
   # Expose port 8501 via Ngrok and print public URL
   public_url = ngrok.connect(8501)
   print("Open this secure URL in your browser:", public_url)
   ```

> **Note on Security:** Never commit your personal `ngrok` authtoken directly to a public GitHub repository. Use Colab Secrets (`google.colab.userdata`) when sharing notebooks publicly.

## 🖥️ Dashboard Features

* **Dual Data Modes:** Switch seamlessly between **Synthetic Simulation Mode** (calibrated to Hyderabad/Jamshoro MGRS `T43RDR` conditions) and **Real NASA HLS GeoTIFF Mode** (via local directory scan or browser file upload).
* **Interactive Priority Sliders:** Customize weights for water conservation, soil nitrogen recovery, heat resilience, and profit maximization.
* **Side-by-Side Spatial Maps:** Inspect late-season $\text{NDVI}$, `CropSegFormer` land-cover masks, and `ConvLSTM` stress probability heatmaps.
* **Explainable Agronomic Trade-Offs:** View clear, plain-language explanations detailing water requirements ($\text{mm}$), nitrogen balance ($\text{kg/ha}$), heat tolerance, and estimated profit ($\text{PKR/acre}$) for each rotation pathway.

## 🌍 Impact & Future Roadmap

* **Empowering Smallholders:** Translates complex satellite observations into actionable, transparent planting decisions that reduce input costs and protect long-term soil fertility.
* **Next Steps:**
  * Integrate live NASA Earthdata CMR API queries for automated granule fetching.
  * Expand localized crop calendars across additional districts in the Indus Basin.
  * Add Urdu and Sindhi language localization for direct farmer accessibility.
