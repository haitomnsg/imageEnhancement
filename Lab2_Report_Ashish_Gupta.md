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
- To study the effect of different transformations on brightness, contrast, and visibility.
- To perform **histogram-based** enhancement, including histogram equalization.
- To apply contrast stretching and binary thresholding and compare them with the original image.

---

## 2. Tools and Environment

- **Language:** Python 3
- **Libraries:** OpenCV (`cv2`), NumPy, Matplotlib, scikit-image, Pillow
- **Platform:** Jupyter Notebook / Google Colab
- **Test images (different characteristics):** a bright high-contrast photograph (*cat*), a standard mid-contrast image (*cameraman*), and a dark low-contrast image (*moon*).

---

## 3. Theory

Image **enhancement** improves the visual quality or interpretability of an image. It does not add new information — it only redistributes or emphasizes the information already present so that important features are easier to perceive. Techniques are grouped into the **spatial domain** (operating directly on pixel values) and the **frequency domain** (operating on the Fourier transform). This lab uses the spatial domain, specifically **point processing** and **histogram processing**.

### 3.1 Point Processing

In point processing (or *intensity transformation*), each output pixel depends **only on the corresponding input pixel**, with no influence from its neighbours:

> **g(x, y) = T[ f(x, y) ]**

where *f* is the input, *g* the output, and *T* the transformation function. The whole operation is described by a single **transformation curve** of output intensity *s* against input intensity *r*. For an 8-bit image, *r* and *s* lie in [0, 255] (0 = black, 255 = white).

**(a) Image Negative** — reverses the intensity scale:

> **s = 255 − r**

Dark pixels become bright and vice-versa; useful for revealing detail hidden in dark regions (e.g. X-ray/medical images).

**(b) Log Transformation:**

> **s = c · log(1 + r)**,  with **c = 255 / log(1 + r_max)**

It maps a narrow range of low intensities to a wider output range while compressing high intensities — i.e. it **expands dark regions and compresses bright ones**, bringing out shadow detail in high-dynamic-range images.

**(c) Inverse Log (Exponential) Transformation** — the opposite mapping:

> **s = c · (baseʳ − 1)**

It **compresses dark regions and expands bright ones**, making the image darker overall.

**(d) Power-Law (Gamma) Transformation:**

> **s = c · rᵞ**

γ < 1 brightens the image and reveals dark detail; γ > 1 darkens it and boosts contrast in bright areas; γ = 1 leaves it unchanged. This is the *gamma correction* used to compensate for the non-linear response of displays and cameras.

**(e) Contrast Stretching** — a **linear** rescaling of the existing range to fill [0, 255]:

> **s = (r − r_min) / (r_max − r_min) · 255**

It stretches the histogram without changing its shape, giving a natural-looking result. A robust variant clips at low/high percentiles (e.g. 2nd–98th) to ignore outlier pixels.

**(f) Binary Thresholding** — converts a grayscale image to two levels:

> **g(x, y) = 255 if f(x, y) > T, else 0**

**Otsu's method** picks *T* automatically by minimizing the intra-class variance, and **adaptive thresholding** uses a different *T* per region to handle uneven lighting. Thresholding is mainly a **segmentation** tool: it separates foreground from background but discards all texture detail.

### 3.2 Histogram Processing

The **histogram** *h(r)* records how many pixels have each intensity *r*. Its shape reflects the image: values clustered near 0 indicate a **dark** image, values near 255 a **washed-out** one, a narrow spread means **low contrast**, and an even spread means **good contrast**.

**Histogram Equalization** redistributes intensities to use the full range as uniformly as possible, maximizing global contrast. It uses the normalized **Cumulative Distribution Function (CDF)** as the mapping:

> **s = round( (cdf(r) − cdf_min) / (M·N − cdf_min) · 255 )**

where *M·N* is the total number of pixels. Because frequent intensities get spread apart, contrast in low-contrast images improves greatly. Being global and automatic, it can over-amplify noise — a refinement, **CLAHE**, equalizes small tiles independently and clips the histogram to limit this. In short, equalization is a *non-linear* reshaping of the histogram, whereas contrast stretching is a gentler *linear* rescaling.

---

## 4. Conclusion

In this lab, a complete set of spatial-domain point-processing and histogram-based enhancement techniques was studied and implemented in Python, and each was applied to images of differing characteristics — bright, mid-contrast, and dark/low-contrast.

The most important conclusion is that **there is no single "best" enhancement technique**; the right method is dictated by the characteristics of the input image and by the goal of the enhancement:

- For **dark or under-exposed** images, the **log transform** and **gamma correction with γ < 1** are most effective, because their curves map a narrow band of dark intensities onto a wide output range and thereby open up shadow detail.
- For **low-contrast** images, **histogram equalization** gives the strongest, most automatic improvement by redistributing intensities across the full range, while **contrast stretching** offers a gentler, more natural-looking alternative when a linear rescaling is sufficient.
- **Binary thresholding** is best understood not as an enhancement method but as a **segmentation** tool: it cleanly separates foreground from background (especially with Otsu's automatically chosen threshold) but necessarily discards all internal detail.

The exercise also clarified the practical trade-offs involved. Point operations are extremely fast and intuitive because they depend on a single transformation curve, yet this same locality is a limitation — a method such as gamma correction applies one uniform curve to the whole image, so brightening the shadows inevitably compresses the highlights, and a single global threshold fails under uneven lighting. Histogram-based methods are more data-driven and powerful, but their automatic, global nature can amplify noise, which motivates locally adaptive refinements such as adaptive thresholding and CLAHE.

Finally, implementing histogram equalization **from scratch** using the CDF and confirming that its output exactly matched OpenCV's built-in `cv2.equalizeHist()` reinforced confidence in both the code and the underlying theory. Overall, the lab provided a clear, working understanding of how simple mathematical transformations of pixel intensities can meaningfully improve the visibility and interpretability of an image, and of how to select an appropriate technique for a given image.

---

*Submitted as part of AICL 311 — Image Processing. The complete, executed code with all output images and histograms is provided in the accompanying notebook `Lab2_Ashish_Gupta.ipynb`.*
