# Architecture Overview — Fourier Boundary Reconstruction App (MATLAB)

This document describes the actual system architecture implemented in `Project_Group10.mlapp`.

The application is a MATLAB App Designer–based interactive tool that:
- Loads an image
- Extracts a boundary using morphological operations
- Computes truncated Fourier shape descriptors
- Reconstructs the shape interactively using a slider
- Exports descriptor output to a text file

This is a classical signal-processing demonstration tool, not a production segmentation system.

---

## 1. High-Level Architecture

The system follows a layered interactive architecture:
+––––––––––+
|      UI Layer      |
| (Buttons, Slider)  |
+––––––––––+
|
v
+––––––––––+
|  Processing Layer  |
| (Morphology + FFT) |
+––––––––––+
|
v
+––––––––––+
| Reconstruction &   |
| Visualization      |
+––––––––––+
|
v
+––––––––––+
| Descriptor Export  |
|   (Optional)       |
+––––––––––+

Each slider interaction triggers a full recomputation of the processing layer.

---

## 2. Component Breakdown

### A) UI Layer (App Designer)

Components:
- Load Image Button
- Slider (controls truncation parameter r)
- Save Descriptors Button
- Image (original image display)
- Image3 (reconstructed output display)

The slider triggers real-time recomputation using `ValueChangingFcn`.

---

### B) Processing Pipeline

Each slider interaction performs the full processing pipeline:

1. Load image from UI display
2. Convert to grayscale (if RGB)
3. Morphological edge enhancement
   - `imdilate` using `strel('disk',5)`
   - Edge extraction: `edges = dilated - original`
4. Global binarization: `imbinarize(edges, "global")`
5. Boundary extraction: `bwboundaries(binary_img)`
6. Fourier descriptor computation
7. Descriptor truncation
8. Inverse reconstruction
9. Pixel remapping
10. Render reconstructed boundary

The pipeline is recomputed fully on each slider movement.

---

## 3. Fourier Descriptor Workflow

Boundary coordinates:
- Extracted using `bwboundaries`
- First boundary (`boundary{1}`) is selected

Coordinates are normalized:
- Mean-centered
- Standard deviation scaled

Complex boundary representation:
z = x + i y

Fourier transform:
F = fftshift(fft(z))

Truncation:
- n ≈ round(N / r)
- Only low-frequency centered coefficients retained

Reconstruction:
- Zero out high-frequency coefficients
- Apply inverse FFT
- Map back to spatial domain

---

## 4. Descriptor Export Behavior

Important:

The Save button does NOT export Fourier coefficients.

It exports reconstructed boundary coordinates:
```matlab
shape = fourier_shape_x + 1i * fourier_shape_y;
```
These complex values are written to file as floating-point numbers.

This is a design choice and should be clearly stated in documentation.

---

## 5. Design Characteristics

- Deterministic processing
- No randomness
- Full recomputation on slider change
- No caching
- Single-boundary assumption
- Real-time interactive feedback

---

## 6. Limitations (Structural)

- Always uses boundary{1}, not necessarily largest object
- Global thresholding may fail on low contrast
- Recomputes entire pipeline on every slider movement
- No multi-object handling
- No boundary resampling for descriptor stability
- Exported descriptors are reconstructed boundary points, not Fourier coefficients

---

## 7. Intended Use

This application is designed for:
- Demonstrating Fourier shape descriptors
- Visualizing compression vs fidelity
- Educational use in digital image processing courses

It is not optimized for:
- Robust segmentation
- Multi-object analysis
- Large-scale datasets
