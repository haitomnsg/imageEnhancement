# Lab 2 — Image Enhancement using Point Processing and Histogram Techniques

**Kathmandu University — School of Engineering**
Department of Artificial Intelligence

| | |
|---|---|
| **Course** | AICL 311 — Image Processing |
| **Lab Title** | Image Enhancement using Point Processing and Histogram Techniques |
| **Student Name** | Ashish Gupta |
| **Roll Number** | 7 |
| **Exam Roll Number** | 31675 |

---

## 1. Objectives

- To implement basic **point processing** techniques for image enhancement.
- To apply intensity transformations on grayscale images using Python.
- To analyze the effect of different transformations on brightness, contrast, and visibility.
- To perform **histogram-based** enhancement, including histogram equalization.
- To apply contrast stretching and binary thresholding and compare the results with the original images.

---

## 2. Tools and Environment

- **Language:** Python 3
- **Libraries:** OpenCV (`cv2`), NumPy, Matplotlib, scikit-image, Pillow
- **Platform:** Jupyter Notebook / Google Colab
- **Test images (different characteristics):**
  - *Cat* — a bright, high-contrast photograph (`cat.jpg`)
  - *Cameraman* — a standard mid-contrast image (`skimage.data.camera()`)
  - *Moon* — a dark, low-contrast image (`skimage.data.moon()`)

---

## 3. Theory

In **spatial-domain** enhancement, every output pixel is computed directly from the input pixels. The simplest family is **point processing**, where the output at a pixel depends only on the input value at that same pixel:

> **g(x, y) = T[ f(x, y) ]**

where *f* is the input image, *g* is the output, and *T* is the intensity-transformation function. Because each pixel is treated independently, point operations are fast and easy to reason about using a *transformation curve* (output intensity vs. input intensity).

### 3.1 Point Processing Transformations

For an 8-bit image, intensities range from 0 (black) to 255 (white). Let *r* be the input intensity and *s* the output intensity.

| Technique | Formula | Effect |
|---|---|---|
| **Image Negative** | s = 255 − r | Inverts intensities; dark ↔ bright |
| **Log Transform** | s = c · log(1 + r) | Expands dark regions, compresses bright regions |
| **Inverse Log (Exponential)** | s = c · (baseʳ − 1) | Compresses dark regions, expands bright regions |
| **Power-Law (Gamma)** | s = c · rᵞ | γ < 1 brightens, γ > 1 darkens |
| **Contrast Stretching** | s = (r − r_min) / (r_max − r_min) · 255 | Linearly rescales intensities to full [0, 255] |
| **Binary Thresholding** | s = 255 if r > T, else 0 | Converts to a black/white (binary) image |

The scaling constant *c* (e.g. for the log transform, **c = 255 / log(1 + r_max)**) maps the result back into the valid [0, 255] display range.

### 3.2 Histogram Processing

The **histogram** h(r) counts how many pixels have each intensity *r*. It is a quick diagnostic: a histogram clustered to the left means a dark image; clustered to the right means a washed-out image; a well-spread histogram usually indicates good contrast.

**Histogram Equalization** redistributes intensities so the full range is used more uniformly. It uses the normalized **Cumulative Distribution Function (CDF)** as the mapping:

> **s = round( (cdf(r) − cdf_min) / (M·N − cdf_min) · 255 )**

where *M·N* is the total number of pixels. This non-linear mapping stretches the most frequent intensities apart, improving **global contrast**.

**Contrast Stretching**, by contrast, is a *linear* remap of the existing range to [0, 255]; it improves visibility without changing the relative shape of the histogram, giving a more natural look.

---

## 4. Methodology

Each technique was implemented as a small, well-commented function and applied to the test images, displaying the original and processed images (and transformation curves / histograms where relevant) side by side. Representative implementation snippets:

**Image Negative**
```python
img_negative = 255 - img
```

**Log Transform** (constant `c` keeps the output in range; cast to float to avoid uint8 overflow)
```python
c = 255.0 / np.log(1.0 + np.max(img).astype(float))
img_log = np.uint8(c * np.log(1 + img.astype(np.float64)))
```

**Gamma Correction**
```python
def gamma_transform(image, gamma):
    norm = image / 255.0
    return np.uint8(np.power(norm, gamma) * 255)
```

**Histogram Equalization** (from scratch via the CDF)
```python
hist, _ = np.histogram(image.ravel(), 256, (0, 256))
cdf = hist.cumsum()
cdf_min = cdf[cdf > 0].min()
mapping = np.round((cdf - cdf_min) / (image.size - cdf_min) * 255).astype(np.uint8)
equalized = mapping[image]          # identical to cv2.equalizeHist()
```

**Binary Thresholding** (manual T, plus Otsu's automatic threshold)
```python
img_thresh = np.where(img > 127, 255, 0).astype(np.uint8)
T_otsu, img_otsu = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
```

The manual histogram-equalization output was verified to be **identical** to OpenCV's `cv2.equalizeHist()`, confirming the implementation is correct.

---

## 5. Results and Observations

| Technique | Observed effect |
|---|---|
| **Image Negative** | Intensities reversed — dark background became bright and vice-versa. Useful for revealing detail hidden in dark regions (e.g. medical/X-ray imaging). |
| **Log Transform** | Shadow detail was brought out by expanding low intensities and compressing highlights. Effective for dark, high-dynamic-range images. |
| **Inverse Log** | Opposite effect — the image darkened overall as dark tones were compressed and bright tones expanded. |
| **Gamma Correction** | γ = 0.4 brightened the image and revealed dark detail; γ = 2.5 darkened it and increased contrast in bright areas. More controllable than the log transform. |
| **Histogram Equalization** | Strongly increased global contrast on the low-contrast image by flattening the histogram and spreading the CDF, though it slightly amplified background noise. |
| **Contrast Stretching** | Expanded the intensity range linearly, improving visibility while preserving a natural look. The percentile (2–98%) variant was more robust to outlier pixels than min–max. |
| **Binary Thresholding** | Produced a clean black/white image. Otsu's method automatically chose a near-optimal threshold and separated foreground from background, but all internal texture was lost. |

> *(The corresponding side-by-side images, transformation curves, and before/after histograms are shown in the accompanying notebook `Lab2_Ashish_Gupta.ipynb`.)*

In addition to the required techniques, the notebook also demonstrates **spatial-domain filtering** — smoothing (box, Gaussian, median), sharpening (Laplacian, unsharp masking, high-boost) — and applies histogram equalization / CLAHE to a colour image in the LAB colour space.

---

## 6. Analysis and Discussion

**Which method improves visibility the most?**
For low-contrast or poorly illuminated images, **histogram equalization** generally gives the largest visibility improvement because it redistributes intensities across the whole range. **Contrast stretching** is a more controlled, natural-looking alternative when the intensities already span a reasonable range.

**Which techniques work best for dark / low-contrast images?**
- **Dark images:** the **log transform** and **gamma correction (γ < 1)** work best, as they map a narrow band of dark inputs onto a wide range of outputs, opening up shadow detail.
- **Low-contrast images:** **contrast stretching** is ideal when intensities occupy a narrow band; **histogram equalization** is preferred when the distribution is highly concentrated and a stronger, non-linear boost is needed.

**Limitations of thresholding and gamma correction**
- **Binary thresholding** discards all intensity information except two levels, so texture and gradients are lost. A single global threshold also fails under uneven lighting — which is why adaptive (local) thresholding exists.
- **Gamma correction** applies one uniform non-linear curve to the whole image. Brightening shadows (γ < 1) simultaneously compresses highlights, which can wash out bright detail, and the γ value must be tuned manually for each image.

---

## 7. Conclusion

All required point-processing and histogram techniques were implemented and analysed successfully. The key takeaway is that **no single technique is universally best** — the right choice depends on the image's characteristics:

- **Log** and **gamma (γ < 1)** transforms are best for **dark / under-exposed** images.
- **Histogram equalization** and **contrast stretching** are best for **low-contrast** images.
- **Thresholding** is best suited to **segmentation** tasks rather than general enhancement.

Comparing the manual histogram-equalization implementation against OpenCV's built-in function confirmed the correctness of the from-scratch approach and reinforced the underlying CDF-based theory.

---

*Submitted as part of AICL 311 — Image Processing. The complete, executed code with all output images and histograms is provided in `Lab2_Ashish_Gupta.ipynb`.*
