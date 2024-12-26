# Texture-Segmentation-App


![image](https://github.com/user-attachments/assets/bb125021-b80e-4e4c-aaf8-66801e9b439e)

> This task involves texture segmentation of grayscale images from the Prague dataset. It is part of a Computer Vision course at my university. As a student, I find it quite challenging to fully understand this area, so I decided to replicate an algorithm published in this article - [A DATA-CENTRIC APPROACH TO UNSUPERVISED TEXTURE SEGMENTATION USING PRINCIPLE REPRESENTATIVE PATTERNS](https://www.researchgate.net/publication/331731944_A_DATA-CENTRIC_APPROACH_TO_UNSUPERVISED_TEXTURE_SEGMENTATION_USING_PRINCIPLE_REPRESENTATIVE_PATTERNS?enrichId=rgreq-7c9d63d6e208de4d9a21b1faf1fb37b1-XXX&enrichSource=Y292ZXJQYWdlOzMzMTczMTk0NDtBUzo3NDk0NTY4ODI0MjE3NjBAMTU1NTY5NTg1MzQzNg%3D%3D&el=1_x_3&_esc=publicationCoverPdf).
I would be incredibly grateful if a professional in this field could help identify and correct my mistakes, as well as provide recommendations for improvement.
My code is available [here on GitHub](https://github.com/dmytro-varich/Texture-Segmentation-App/blob/main/segmentation_main.ipynb).

I will implement a texture segmentation algorithm using the principle of representative patterns (PRP). The basic idea is to extract texture features based on local and global image patches. However, I am having difficulties with the segmentation quality, especially in the patch classification stage and subsequent segmentation.

## **Approach Description:**
### 1. Image Preprocessing  
**Method:** `preprocess_image`  
**Also Class:** `ImagePreprocessingMixin`  

In the first step, I preprocess the image using various methods to preserve essential information required for the subsequent patch clustering stage. By experimenting with different parameters, I aim to enhance the representation of relevant features. The preprocessing methods include:  
- **Gaussian Blur** (`_apply_gaussian_blur`): Smooths the image and reduces minor noise.  
- **Wavelet Transform** (`_apply_wavelet_transform`): Highlights local frequency features.  
- **Fourier Transform** (`_apply_fourier_transform`): Captures global frequency characteristics of the image.  

---

### 2. Patch Extraction  
**Method:** `extract_patches`  

Next, I divide the image into fixed-size square patches (e.g., 16x16 pixels). These patches are extracted from each image in the dataset to analyze local texture features.  

- **Overlapping or Non-Overlapping Patches**: The patch extraction process depends on the `patch_step` parameter, which determines the step size between patches.  

---

### 3. PRP (Principle Representative Patterns)  
**Method:** `cluster_patches`  

All the extracted patches are combined into a single array and clustered using the **K-Means** algorithm. The resulting cluster centroids are interpreted as **PRPs** (Principle Representative Patterns).  

- **The Core Idea of PRP**:  
  PRPs provide a compact representation of textures by leveraging the **self-similarity** and **quasi-periodicity** properties of textures.  

---

### 4. Texture Feature Calculation Based on PRP  
**Method:** `compute_texture_features`  

In this step, I compute texture features for each image:  
- **Similarity Between Patches and PRPs**:  
  The similarity of each patch to the PRPs is measured using a weighted Gaussian function, which evaluates how strongly a patch corresponds to each PRP.  
- **Result**:  
  The output is interpreted as a probability distribution or energy spectrum, representing the degree to which each patch belongs to a particular PRP.  

Gaussian Formula of atricle:
[Gaussian Formula](https://i.sstatic.net/ORhT4s18.png) 

---

### 5. Segmentation Using the Mumford-Shah Algorithm  
**Method:** `segmentation`  

Based on the computed texture features, a **feature map** is generated for the entire image. This feature map is fed into the Mumford-Shah segmentation algorithm (I used an open-source implementation and integrated the corresponding class into my code).  

- The algorithm divides the image into regions with homogeneous texture features.  

---

### 6. Post-Processing  
**Method:** `postprocess_segmentation`  

For refining the segmentation boundaries, I use **CRF** (Conditional Random Fields) from the `pydensecrf` library.  
- **Purpose of Post-Processing**:  
  - Remove minor noise.  
  - Refine the boundaries between segmented regions.  
---
## Questions:

1. What preprocessing methods are best suited for extracting meaningful texture features from an image?  

2. Is my implementation of patch comparison in the `compute_texture_features` method correct?  

3. What can be done to improve the algorithm? Would using a different clustering algorithm be more effective? If so, which one would you recommend?  

Thanks for you answer!


