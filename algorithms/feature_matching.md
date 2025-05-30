# Feature Matching and Image Alignment Evaluation

This repository provides a comprehensive framework for evaluating various feature detectors and matching algorithms on a subset of the UBC IMW2020 dataset. In our experiments, we focus on the **Brandenburg tur images** subset (comprising 100 images) and compare detectors such as **SIFT**, **ORB**, **AKAZE**, **BRISK**, and **KAZE** using both **Brute-Force (BF)** and **FLANN-based** matchers.

Our evaluation metrics include:
- **Number of Matches:** The number of valid correspondences detected between image pairs.
- **Reprojection Error:** The average error (in pixels) when mapping keypoints using the computed homography.

---

## Table of Contents

- [Introduction](#introduction)
- [Definitions and Formulas](#definitions-and-formulas)
- [Methodology](#methodology)
  - [Feature Detectors](#feature-detectors)
  - [Feature Matchers](#feature-matchers)
  - [Homography and Reprojection Error](#homography-and-reprojection-error)
  - [Handling Unknown Overlap](#handling-unknown-overlap)
  - [Preprocessing Steps](#preprocessing-steps)
  - [Evaluation Strategy](#evaluation-strategy)
- [Notebook Experimental Results](#notebook-experimental-results)
- [Experimental Results](#experimental-results)
  - [Average Results](#average-results)
  - [Conclusion](#conclusion)
- [Usage](#usage)
  - [Script Execution](#script-execution)
  - [Data Samples and Links](#data-samples-and-links)
- [References](#references)

---

## Introduction

In many computer vision tasks such as panoramic stitching, 3D reconstruction, and object recognition, robust feature matching and image alignment are critical. However, when the exact overlap between images is unknown, a dynamic and robust evaluation method is required. This project evaluates several popular feature detectors and matchers on a subset of the UBC IMW2020 dataset—specifically, the **Brandenburg tur images** subset (100 images)—to determine the optimal combination for reliable image alignment.

---

## Definitions and Formulas

### Key Terms

- **Feature Detector:** An algorithm used to identify salient points (keypoints) in an image. Examples include SIFT, ORB, AKAZE, BRISK, and KAZE.
- **Descriptor:** A vector that encodes the local appearance around a keypoint.
- **Matcher:** An algorithm that compares descriptors between images to establish correspondences.
- **Homography:** A 3×3 matrix $$H$$ that maps points from one plane (image) to another, typically used to align two images.
- **Reprojection Error:** A quantitative measure of alignment accuracy, calculated as the average distance between keypoints in the second image and the projected keypoints from the first image using the homography $$H$$.

### Formula for Reprojection Error

Given $$N$$ inlier matches, where $$\mathbf{p}_i$$ are keypoints in the first image and $$\mathbf{p}'_i$$ are their corresponding points in the second image, the average reprojection error is defined as:

$$
\text{Average Error} = \frac{1}{N} \sum_{i=1}^{N} \left\| \mathbf{p}'_i - H \mathbf{p}_i \right\|
$$

---

## Methodology

### Feature Detectors

1. **SIFT (Scale-Invariant Feature Transform):**
   - *Description:* Detects keypoints invariant to scale, rotation, and moderate illumination changes. Based on [Lowe, 2004](https://doi.org/10.1023/B:VISI.0000029664.99615.94).
   - *Advantages:* High robustness and repeatability; widely used in many applications.
   - *Disadvantages:* Computationally intensive.

2. **ORB (Oriented FAST and Rotated BRIEF):**
   - *Description:* Combines the FAST detector with a modified BRIEF descriptor (incorporating orientation) as proposed by [Rublee et al., 2011](https://ieeexplore.ieee.org/document/6126544).
   - *Advantages:* Fast and computationally efficient; free and open-source.
   - *Disadvantages:* May yield fewer or less distinctive matches.

3. **AKAZE:**
   - *Description:* Utilizes nonlinear diffusion filtering for keypoint detection and computes binary descriptors, introduced in [Alcantarilla et al., 2013](https://ieeexplore.ieee.org/document/6620643).
   - *Advantages:* Robust to noise and offers good performance with binary descriptors.
   - *Disadvantages:* Results can be variable across different datasets.

4. **BRISK (Binary Robust Invariant Scalable Keypoints):**
   - *Description:* Detects keypoints and computes binary descriptors that are invariant to scale and rotation, as detailed in [Leutenegger et al., 2011](https://ieeexplore.ieee.org/document/6126542).
   - *Advantages:* Very high match density and speed.
   - *Disadvantages:* Matching precision can be lower than with methods like SIFT.

5. **KAZE:**
   - *Description:* Operates in nonlinear scale space to detect keypoints and compute descriptors, similar to AKAZE but without linear approximations [Alcantarilla et al., 2012](https://ieeexplore.ieee.org/document/6460711).
   - *Advantages:* Good balance between match density and alignment accuracy.
   - *Disadvantages:* More computationally expensive than binary methods.

### Feature Matchers

- **Brute-Force (BF) Matcher:**  
  - *Description:* Matches each descriptor in one image with all descriptors in another using a cross-check for symmetry. Proposed in [Lowe, 2004](https://doi.org/10.1023/B:VISI.0000029664.99615.94).
  - *Advantages:* Simple and typically yields a high number of matches.  
  - *Disadvantages:* Slower for very large descriptor sets.

- **FLANN (Fast Library for Approximate Nearest Neighbors):**  
  - *Description:* Uses an approximate nearest neighbor search combined with Lowe's ratio test to filter matches [Muja and Lowe, 2009](https://link.springer.com/article/10.1007/s11263-009-0273-4).
  - *Advantages:* Faster than BF on large datasets.  
  - *Disadvantages:* For binary descriptors, FLANN may be overly conservative, sometimes resulting in no valid matches.

### Homography and Reprojection Error

After matching, the keypoint correspondences are used to compute a homography $$H$$ using the RANSAC algorithm. The reprojection error, as defined above, quantifies the alignment accuracy [Fischler and Bolles, 1981](https://doi.org/10.1145/358669.358692).

### Handling Unknown Overlap

Since the exact overlap between images is unknown, the evaluation is performed on every consecutive image pair from the subset. By checking the number of matches and reprojection error for each pair, we can:
- Determine whether the overlap is sufficient.
- Filter out pairs with inadequate overlap.
- Adjust thresholds dynamically based on actual data.

### Preprocessing Steps

Before feature detection, the images undergo preprocessing to enhance robustness:
- **Grayscale Conversion:** Simplifies processing by reducing the image to a single channel.
- **Histogram Equalization:** Improves contrast and feature visibility.
- **Gaussian Blurring:** Reduces noise that could lead to spurious keypoints.

### Evaluation Strategy

- **Dynamic Thresholding:** Adaptively sets match quality thresholds based on initial evaluations.
- **Cross-Validation:** Uses image pairs with known overlap to validate performance.
- **Visualization:** Automatically generates match visualizations to qualitatively assess performance.

---

## Notebook Experimental Results

The experimental evaluation results presented below are directly obtained from running the `feature_matching.ipynb` notebook. The notebook processes a subset of the UBC IMW2020 dataset—specifically, the **Brandenburg tur images** subset (100 images)—and evaluates the performance of different feature detector–matcher combinations. Key insights from the updated notebook results include:

- **BF matching and FLANN matching provide different trade-offs:**  
  Depending on the feature detector used, BF and FLANN may produce varying match counts and reprojection errors.
- **SIFT exhibits relatively high match counts** with both BF and FLANN, but at the cost of higher reprojection errors.
- **ORB, AKAZE, BRISK, and KAZE show lower match counts overall,** with notable differences between BF and FLANN strategies.

These observations have been integrated into the table and discussion in the next section.

---

## Experimental Results

### Average Results

The updated average experimental results obtained from the notebook are summarized in the table below:

| Detector | Matcher | Avg. Matches | Avg. Reprojection Error (pixels) |
|----------|---------|--------------:|---------------------------------:|
| SIFT     | BF      |       104.84  |                         271.76   |
| SIFT     | FLANN   |       114.00  |                         388.43   |
| ORB      | BF      |        10.54  |                         290.86   |
| ORB      | FLANN   |        11.92  |                         116.28   |
| AKAZE    | BF      |        22.24  |                         155.79   |
| AKAZE    | FLANN   |        27.22  |                         312.40   |
| BRISK    | BF      |        16.70  |                         149.32   |
| BRISK    | FLANN   |        22.86  |                         244.32   |
| KAZE     | BF      |        71.88  |                         163.95   |
| KAZE     | FLANN   |        79.68  |                         251.68   |

*Note:* The differences in performance between BF and FLANN matchers vary with the detector type. For instance, while SIFT shows a modest increase in matches with FLANN, the reprojection error is considerably higher. Conversely, ORB yields a lower error with FLANN despite low match counts.

---

### Conclusion

- **BF vs. FLANN:**  
  The results demonstrate that the choice between BF and FLANN matchers can lead to different trade-offs in terms of the number of matches and the reprojection error. The optimal choice depends on the specific requirements of the application.
  
- **Detector Observations:**  
  - **SIFT:** Provides high match counts; however, the associated reprojection errors suggest a trade-off in precision.
  - **ORB:** Consistently low match counts with BF and FLANN; though FLANN produces a lower error, the overall number of matches is very low.
  - **AKAZE and BRISK:** Show moderate match counts with relatively low reprojection errors, making them viable candidates when balanced performance is desired.
  - **KAZE:** Delivers the highest number of matches among binary detectors, with acceptable reprojection error, particularly favoring BF matching.

**Overall Recommendation:**  
The choice of detector and matcher should be guided by the specific alignment accuracy and match density required by the application. For high-precision applications, the lower reprojection error of certain FLANN configurations (e.g., ORB) might be desirable, while for robust feature density, BF matching with SIFT, AKAZE, or KAZE may be preferable.

---

## Usage

### Script Execution

The main processing script is available at:  
[feature_matching](https://github.com/tamer017/Multi-Angular-Photogrammetry/blob/master/scripts/feature_matching.ipynb)

To reproduce the experiments:
1. Open `feature_matching.ipynb` in Jupyter Notebook.
2. Download and extract the dataset from [UBC IMW2020 Data](https://www.cs.ubc.ca/~kmyi/imw2020/data.html).
3. Use the **Brandenburg tur images** subset (100 images) from the extracted data.
4. Run all cells to perform feature detection, matching, and evaluation.
5. Visualizations and updated experimental results will be generated automatically.

### Data Samples and Links

The dataset used in this evaluation is a subset of the UBC IMW2020 dataset. Specifically, the **Brandenburg tur images** subset (100 images) was used.  
- Complete dataset: [UBC IMW2020 Data](https://www.cs.ubc.ca/~kmyi/imw2020/data.html)  
- Brandenburg tur images subset: [Brandenburg tur images subset](https://github.com/tamer017/Multi-Angular-Photogrammetry/tree/master/data/brandenburg_gate_images)

---

## References

1. Lowe, D. G. "Distinctive Image Features from Scale-Invariant Keypoints." *International Journal of Computer Vision*, 2004.
2. Rublee, E., Rabaud, V., Konolige, K., & Bradski, G. "ORB: an efficient alternative to SIFT or SURF." 2011.
3. Alcantarilla, P. F., Nuevo, J., & Bartoli, A. "Fast explicit diffusion for accelerated features in nonlinear scale spaces." BMVC, 2012.
4. Brown, M., & Lowe, D. G. "Automatic panoramic image stitching using invariant features." *International Journal of Computer Vision*, 2007.
5. Fischler, M. A., & Bolles, R. C. "Random sample consensus: a paradigm for model fitting with applications to image analysis and automated cartography." *Communications of the ACM*, 1981.
