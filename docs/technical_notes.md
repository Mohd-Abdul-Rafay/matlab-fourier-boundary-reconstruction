# Technical Notes — Fourier Boundary Reconstruction (MATLAB)

This document records the technical decisions, algorithmic details, and known limitations for the Fourier Shape Descriptor reconstruction workflow implemented in `src/FourierShapeDescriptorApp.mlapp`.

## 1) Problem Statement

Given an input image containing a single dominant object (e.g., chromosome), we:

1. Extract a boundary from a binarized edge map.
2. Represent the boundary as a complex sequence.
3. Compute Fourier descriptors via the DFT.
4. Truncate descriptors to control compression.
5. Reconstruct an approximate boundary using the inverse DFT.
6. Visualize the reconstructed shape and optionally export descriptors.

This is a classical, interpretable pipeline intended for study and controlled experimentation (not a production segmentation system).

---

## 2) Preprocessing Pipeline (Edge → Binary)

### Morphological edge emphasis
The app uses a morphological edge-like operation:

- Dilate grayscale image: `I_d = imdilate(I, strel('disk', 5))`
- Edge evidence: `E = I_d - I`

This increases boundary contrast where dilation expands bright structures beyond their original support.

### Binarization
- `B = imbinarize(E, "global")`

This is intentionally simple and deterministic. It is sensitive to illumination and object scale.

**Assumption:** The binarized image contains one dominant foreground object with a clean boundary.

---

## 3) Boundary Extraction

- `bwboundaries(B)` returns boundary point sequences.
- Current implementation selects `boundary{1}`.

**Important limitation:** `boundary{1}` is not guaranteed to be the *largest* or *correct* boundary if multiple components exist.

Recommended robust alternative (future work):
- Select boundary with maximum length or maximum enclosed area.

---

## 4) Fourier Descriptor Formulation

Let boundary points be:
- \( x_k, y_k \) for \( k = 0, 1, \dots, N-1 \)

Normalize to reduce translation and scale sensitivity:
- \( \tilde{x}_k = \frac{x_k - \mu_x}{\sigma_x} \)
- \( \tilde{y}_k = \frac{y_k - \mu_y}{\sigma_y} \)

Represent as complex sequence:
- \( z_k = \tilde{x}_k + i\tilde{y}_k \)

Fourier descriptors:
- \( Z_m = \text{DFT}(z_k) \)

Implementation uses:
- `fftshift(fft(z))` for centered spectrum.

---

## 5) Descriptor Truncation (Compression Control)

The slider controls a parameter `r` that defines how many coefficients are preserved.

If \(N\) is number of boundary points:
- Kept coefficients \( n \approx \text{round}(N / r) \)

Coefficients retained symmetrically around DC in the shifted spectrum to preserve low-frequency structure.

Reconstruction:
- Zero all discarded coefficients.
- \( \hat{z}_k = \text{IDFT}(\hat{Z}_m) \)

---

## 6) Reprojection to Image Space

The reconstructed complex sequence is mapped back to pixel coordinates using the original boundary statistics:

- \( \hat{x}_k = \mu_x + \Re(\hat{z}_k)\sigma_x \)
- \( \hat{y}_k = \mu_y + \Im(\hat{z}_k)\sigma_y \)

A sparse binary image is then created by setting pixels at \((\hat{x}_k, \hat{y}_k)\) to 1.

**Note:** This produces a boundary trace, not a filled region. If you need a filled shape, apply `imfill` on the boundary mask after closing gaps.

---

## 7) Exported Descriptors

The “Save Descriptors” action exports a 1D descriptor representation to a text file.

Current app behavior writes the reconstructed complex boundary coordinates (complex-valued sequence) rather than the Fourier coefficients.

If the goal is to export **Fourier descriptors**, the exported vector should be the truncated coefficient set \( \hat{Z}_m \) (or magnitudes/phases), not boundary points.

This is a definitional choice—documented here so users understand what is being saved.

---

## 8) Complexity and Performance

Brute complexity:
- FFT/IDFT: \( O(N\log N) \)
- Boundary extraction: depends on contour length.
- Slider interaction: recomputes pipeline each change → can be heavy for large images.

Practical guidance:
- Resize input images to a consistent scale for smooth UI responsiveness.
- Reduce boundary points via contour resampling if needed.

---

## 9) Known Failure Modes (Honest)

1. Multiple objects → wrong boundary chosen.
2. Weak edges / low contrast → binarization produces fragmented contours.
3. Boundary self-intersections → reconstruction may fold.
4. Very small `r` (too aggressive truncation) → oversmoothing and shape collapse.
5. Coordinate rounding may introduce gaps → boundary appears broken.

---

## 10) Suggested Improvements (Future Work)

- Largest-component boundary selection.
- Adaptive thresholding (`imbinarize(..., 'adaptive')`) or Otsu on edges.
- Contour resampling to fixed \(N\) for consistent descriptor comparison.
- Export true Fourier coefficients + metadata (N, r, normalization).
- Fill and post-process reconstruction for region-level masks.

---

## Reproducibility

This repo is designed as a reproducible classical baseline:
- deterministic operations
- explicit parameters
- examples included under `examples/`

See `docs/architecture_overview.md` for a system-level view.
