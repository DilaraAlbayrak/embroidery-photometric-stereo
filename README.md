# Photometric Stereo for Embroidery Digitisation

A photometric stereo pipeline for 3D surface reconstruction of embroidered textiles, using synthetic captures generated in Unity under parametric spherical lighting. This study was developed as a proof of concept exploring how computer vision techniques can support **the digitisation of textile cultural heritage**.

This is a project sitting at the intersection of computer vision and textile digitisation. It was developed to investigate whether classical photometric stereo, combined with a controlled synthetic capture setup, could recover meaningful surface normals from embroidered textile samples whose three-dimensional structure (stitch height, thread weave) is difficult to capture with conventional flat scanning.

## Pipeline Overview

<img width="1543" height="678" alt="photometric stereo" src="https://github.com/user-attachments/assets/548d6894-0216-4380-a0c0-64d574751c63" />

The pipeline consists of three stages:

1. **Synthetic capture in Unity:** A textile sample is rendered under a configurable set of point light sources distributed on a sphere around the object. Lighting positions are parameterised in spherical coordinates (roll and yaw axes), and the camera is fixed. For each light configuration, a grayscale image is rendered.
2. **Photometric stereo solve:** The captured images and their associated light direction vectors are fed into a classical photometric stereo solver based on the Lambertian reflectance model. Surface normals are recovered via least-squares (pseudo-inverse).
3. **Normal map output:** Recovered normals are encoded as an RGB normal map, which can be used downstream for relighting, visualisation, or further geometric processing.

## Capture Setup

Captures were generated in Unity with the following parametric setup:

- A virtual hemisphere/sphere is constructed around the target textile sample.
- Lighting positions are placed at user-defined intervals along the **roll axis (red)** and **yaw axis (yellow)** (see pipeline diagram).
- The user specifies the number of lights and their angular spacing; the system enables only the configured subset for each capture pass.
- Each rendered image is saved with the corresponding light direction encoded in the filename as `(x, y, z)`, which the solver parses at load time.

This setup is deliberately synthetic. Working in Unity allowed full control over lighting direction, camera position, and surface material, which is difficult to achieve with a physical photometric stereo rig without specialist equipment.

## Repository Structure

```
embroidery-photometric-stereo/
├── ps.ipynb                      # Grayscale photometric stereo solver
├── ps_rgb.ipynb                  # RGB-channel photometric stereo variant
├── config.yaml                   # Capture and processing configuration
├── unity-capture-samples/        # Example synthetic captures from Unity
│   └── ...
└── README.md
```

## Method Notes

The solver implements the standard formulation of photometric stereo under the Lambertian reflectance assumption:

```
I = L · N
```

where `I` is the per-pixel intensity vector across captures, `L` is the matrix of light direction vectors, and `N` is the unknown surface normal. The solution is obtained via the pseudo-inverse:

```
N = (LᵀL)⁻¹ Lᵀ I
```

Normals are then normalised to unit length and remapped to the `[0, 255]` range for visualisation as a normal map.

The RGB variant runs the same solve on each colour channel independently, which can offer modest improvements on textiles with strong colour variation but is sensitive to channel-wise specularity.

## Notes and Limitations

This is an early-stage exploratory implementation. Known limitations include:

- The Lambertian assumption breaks down on shiny or metallic threads (e.g. gold thread), introducing artefacts in the recovered normals.
- The current synthetic capture setup does not model self-shadowing between stitches, which would be present in physical captures.
- The pipeline has not been validated against ground-truth geometry; results are visual rather than quantitative.

Despite these limitations, the recovered normal maps preserve enough fine-scale surface detail to be visually informative for embroidery characterisation, and the modular structure (Unity capture → light parsing → solver) is straightforward to extend with more advanced reflectance models or real-world captures.
