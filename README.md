## Fourier Boundary Reconstruction — Research Prototype (MATLAB)

A deterministic research prototype for boundary-based shape modeling using Fourier Shape Descriptors.

This project investigates how spectral truncation of contour representations governs geometric reconstruction fidelity. It provides an interactive framework for analyzing how low-frequency Fourier components encode global structure while high-frequency components control boundary detail.

The application enables real-time reconstruction by adjusting the number of retained Fourier coefficients.

---

## Concept

A 2D object boundary can be represented as a complex sequence:
```text
Applying the Discrete Fourier Transform:

Z(m) = ∑ z(k) e^{-i 2πmk/N}
```
Applying the Discrete Fourier Transform:
```text
Z(m) = FFT(z(k))
```
Truncating high-frequency components produces a spectrally compressed boundary representation.

This project visualizes that compression–fidelity trade-off interactively.

---

## What This Prototype Demonstrates
- Complex contour encoding
- Spectral shape representation
- Low-frequency dominance in geometric structure
- High-frequency contribution to edge sharpness
- Real-time inverse reconstruction
- Deterministic classical vision pipeline

This is a signal-processing–driven shape modeling system — not a machine learning model.

---

## Processing Pipeline
1. Image Input
  - JPG / PNG / BMP / TIF
2. Preprocessing
	- Grayscale conversion
  - Morphological dilation (strel('disk',5))
	- Edge emphasis: edges = dilated - original
	- Global binarization
3. Boundary Extraction
  - bwboundaries
	- First detected contour selected
4. Fourier Descriptor Computation
	- Complex boundary normalization
	- fft + fftshift
	- Low-frequency truncation (controlled by slider)
5. Reconstruction
	- Zeroing high-frequency coefficients
	- ifft
	- Spatial remapping
	- Binary boundary rendering
6. Descriptor Export
	- Saves reconstructed boundary coordinates

---

## Interactive Control

Slider parameter r controls compression:
```bash
n ≈ round(N / r)
```
Where:
- N = number of boundary samples
- n = retained Fourier coefficients

Lower r → higher fidelity
Higher r → smoother reconstruction

---

## Repository Structure
```text
matlab-fourier-boundary-reconstruction/
├── src/
│   └── FourierShapeDescriptorApp.mlapp
├── examples/
│   ├── chromosome_example.png
│   ├── reconstruction_r5.png
│   ├── reconstruction_r50.png
│   └── ui_overview.png
├── docs/
│   ├── technical_notes.md
│   └── architecture_overview.md
├── README.md
├── LICENSE
└── .gitignore
```
---

## How to Run
1.	Open FourierShapeDescriptorApp.mlapp in MATLAB App Designer.
2.	Run the application.
3.	Load an image.
4.	Adjust slider to vary spectral truncation.
5.	Optionally export descriptor output.

---

## Design Properties
- Fully deterministic
- Classical signal-processing implementation
- Explicit frequency-domain manipulation
- No learned components
- Real-time reconstruction visualization
- Research-oriented parameter exposure

---

## Limitations
- Uses boundary{1} only (not necessarily largest object)
- Global thresholding may fail under non-uniform illumination
- No rotation-invariant normalization
- No contour resampling for descriptor stability
- Full pipeline recomputed on each slider update
- Exports reconstructed boundary points, not raw Fourier coefficients

---

## Research Focus

This work explores:
- Spectral encoding of geometric structure
- Information loss under Fourier truncation
- Relationship between contour frequency content and spatial smoothness
- Controlled experimental visualization of boundary compression

The system functions as a classical spectral shape modeling framework for computer vision research.

---

## Requirements

- MATLAB
- Image Processing Toolbox
- App Designer

---

## Author

**Abdul Rafay Mohd**  
Artificial Intelligence | Medical AI | Computer Vision 

---

## Contributors

- Abdul Rafay Mohd — Algorithm design, MATLAB implementation, UI architecture, Fourier reconstruction logic
- Ziya Mubeen Ahmed Mohammed — Analytical evaluation and experimentation
- Abdul Sohail Mohammed — Report writing and presentation development

---

## Citation

If referencing this work, please cite:

```bibtex
@software{rafay2025fourier,
  author  = {Abdul Rafay Mohd and Ziya Mubeen Ahmed Mohammed and Abdul Sohail Mohammed},
  title   = {Fourier Boundary Reconstruction: Interactive Spectral Shape Modeling},
  year    = {2025},
  url     = {https://github.com/Mohd-Abdul-Rafay/matlab-fourier-boundary-reconstruction}
}
```

---

## License

This project is licensed under the terms of the [MIT License](LICENSE).
