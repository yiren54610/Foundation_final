
# Data Diary: Color Patterns in the Fast-Fashion Industry

## **Introduction**  
This project aims to track color usage patterns in the fashion industry, focusing primarily on the color choices of women’s new arrivals from fast-fashion brands. The final output is an auto-updating website (`index.html`) displaying color grids with corresponding RGB values, updated weekly. The website also archives the previous version for comparison, serving as a source of inspiration for designers and manufacturers.

---

## **Chronological Log**

### **Project Structure**  
The project consists of three main parts:  
1. **Data Extraction**  
2. **Analysis**  
3. **Visualization**  

### **Data Collection**  
Data was collected using the Playwright library to scrape the official websites of four fashion brands. Media links for new arrivals were extracted, and the corresponding images were downloaded and saved in brand-specific directories.  

An image segmentation model from Hugging Face, `mattmdjaga/segformer_b2_clothes`, was employed to crop out the background and body parts of model images, isolating only the clothing.  

**Code Example:**  
```python
from transformers import pipeline
segmenter = pipeline("image-segmentation", model="mattmdjaga/segformer_b2_clothes")
```

The output from this step consisted of images containing only the clothing sections.  

### **Analysis**  
The segmented images underwent color extraction using the KMeans clustering algorithm. A color palette was generated for each brand using Matplotlib.  

The results were compiled into a color wheel format, with the most frequently occurring color recorded using Python's `collections.Counter`.  

### **Visualization**  
The final product was saved as a color grid, with RGB values displayed across the grid in `color_grids.html`. This section was dynamically integrated into `index.html` using JavaScript for seamless updates.

---

## **Challenges**  
1. **Scraping Blocks:**  
   - Repeated scraping using Playwright led to agent blocks.  
   - The headless mode of Playwright was detected as a bot. To overcome this, a virtual browser (Xvfb) was set up to enable "headed mode" without displaying the browser interface.  

   **Setup Command Example:**  
   ```bash
   - name: Set up Xvfb for non-headless Playwright
     run: |
       sudo apt-get update
       sudo apt-get install -y xvfb
       Xvfb :99 -screen 0 1920x1080x24 &
       export DISPLAY=:99
   ```

2. **HTML Conversion:**  
   To allow notebooks to output HTML files:  
   ```bash
   - name: Convert Notebook to HTML
     run: |
       jupyter nbconvert --to html Model__building.ipynb --output color_grids.html
   ```

---

## **Unresolved Issues**  
1. **Incomplete Dataset:**  
   - Data for Forever 21 could not be included due to a persistent `403 Forbidden` error.  
2. **Color Limitations:**  
   - KMeans clustering could not perfectly capture all exact color shades. To address this, the website includes a link to the original webpage for reference.  

---

## **Findings**  
The analysis revealed a strikingly similar pattern in color use across brands, with a shared preference for darker tones. While each brand maintains some classic colors, the overall palette showed limited diversity.  

---

## **Next Steps**  
1. Revamp the layout of `index.html` to give greater prominence to the color grids.  
2. Expand the dataset by including additional brands.  
3. Incorporate **best-seller** data, as it may offer deeper insights compared to new arrivals.  

--- 

This document provides a transparent and replicable overview of the project's process and outcomes. Further enhancements and datasets will continue to improve its scope and value.  
