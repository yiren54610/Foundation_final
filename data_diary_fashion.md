
# Data Diary: Color Patterns in the Fast-Fashion Industry

## **Introduction**  
This project aims to track color usage patterns in the fashion industry, focusing primarily on the color choices of women’s new arrivals from fast-fashion brands. The final output is an auto-updating website (`index.html`) displaying color grids with corresponding RGB values, updated weekly. The website also archives the previous version for comparison, serving as a source of inspiration for designers and manufacturers.

---

## **Chronological Log**

### **Project Structure**  
The project consists of three main parts: 
1. **Model Building**
2. **Data Extraction**  
3. **Analysis**  
4. **Visualization**  

### **Model building & Testing**
In the first week:
I mainly leanred how to use segmentation model.
An image segmentation model from Hugging Face, `mattmdjaga/segformer_b2_clothes`, was employed to crop out the background and body parts of model images, isolating only the clothing.  

**Code Example:**  
```python
from transformers import pipeline
segmenter = pipeline("image-segmentation", model="mattmdjaga/segformer_b2_clothes")

# Initialize segmentation pipeline
def segment_clothing(img, clothes= ["Upper-clothes", "Pants", "Belt"]):
    # Segment image
    segments = segmenter(img)

    # Create list of masks
    mask_list = []
    for s in segments:
        if(s['label'] in clothes):
            mask_list.append(s['mask'])


    # Paste all masks on top of eachother 
    final_mask = np.array(mask_list[0])
    for mask in mask_list:
        current_mask = np.array(mask)
        final_mask = final_mask + current_mask
            
    # Convert final mask from np array to PIL image
    final_mask = Image.fromarray(final_mask)

    # Apply mask to original image
    img.putalpha(final_mask)

    # display the image
    return img
def batch_segment_clothing(img_dir, out_dir, clothes= ["Hat", "Upper-clothes", "Skirt", "Pants", "Dress", "Belt", "Left-shoe", "Right-shoe", "Scarf"]):
    # Create output directory if it doesn't exist
    if not os.path.exists(out_dir):
        os.makedirs(out_dir)

    # Iterate through each file in the input directory
    for filename in os.listdir(img_dir):
        if filename.endswith(".jpg") or filename.endswith(".JPG") or filename.endswith(".png") or filename.endswith(".PNG"):
            try:
                # Load image
                img_path = os.path.join(img_dir, filename)
                img = Image.open(img_path).convert("RGBA")

                # Segment clothing
                segmented_img = segment_clothing(img, clothes)

                # Save segmented image to output directory as PNG
                out_path = os.path.join(out_dir, filename.split('.')[0] + ".png")
                segmented_img.save(out_path)

                print(f"Segmented {filename} successfully.")

            except Exception as e:
                print(f"Error processing {filename}: {e}")

        else:
            print(f"Skipping {filename} as it is not a supported image file.")

```


### **Data Collection**  
Started from November 25,
I was working on collecting data and gettng it to work throughout the whole project in order to get as much details as possible.
Data was collected using the Playwright library to scrape the official websites of four fashion brands. Media links for new arrivals were extracted, and the corresponding images were downloaded and saved in brand-specific directories.  
The output from this step consisted of images containing only the clothing sections.  

On December 4th, 
I realized that webiste like Zara required playwright to scroll down to the bottom of the page to get all the results, so I made this:

```python
async def scroll_to_bottom(page):
    
    await page.evaluate("window.scrollTo(0, document.body.scrollHeight);")


```

### **Analysis**  

The segmented images underwent color extraction using the KMeans clustering algorithm. A color palette was generated for each brand using Matplotlib.  

The results were compiled into a color wheel format, with the most frequently occurring color recorded using Python's `collections.Counter`.  

```python

from collections import Counter
color_counts = Counter(final_colors)
most_common_color, _ = color_counts.most_common(1)[0]

fig2, ax2 = plt.subplots(subplot_kw={'projection': 'polar'}, figsize=(4, 4))
ax2.bar(angles, np.ones(num_colors), color=[most_common_color] * num_colors, width=2 * np.pi / num_colors)


ax2.set_yticks([])
ax2.set_xticks([])
ax2.spines['polar'].set_visible(False)

plt.savefig('most_common_color_wheel.png', transparent=True)

```



### **Visualization**  
The final product was saved as a color grid, with RGB values displayed across the grid in `color_grids.html`. This section was dynamically integrated into `index.html` using JavaScript for seamless updates.

### **Autoscrapper**
On December 9th, I got my autoscraper to work for the first time, but realized that headless mode blocked three of my webiste.

On December 11th, I added Xvfb in my yaml file and finally get the workflow to run properly. 

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
