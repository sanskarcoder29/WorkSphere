# WebGL & Spatial Indexing Architecture

## Overview

This manual details the hardware-accelerated rendering pipelines and spatial indexing strategies utilized in the WorkSphere frontend. The architecture leverages WebGL for high-performance rendering and spatial quadtrees for efficient viewport culling and user interactions.

---

## 1. Shader GLSL Source Code Structure

Our shader programs are modularized to separate core rendering logic from projection transformations and material calculations.

- **Vertex Shaders (`.vert`):**
  - Handles coordinate transformations (Model, View, Projection matrices).
  - Passes varying attributes (colors, UV coordinates, normals) to the fragment pipeline.
- **Fragment Shaders (`.frag`):**
  - Computes final pixel colors using interpolated variables.
  - Implements texture sampling and hardware-accelerated anti-aliasing.

## 2. Buffer Allocation Rules

To minimize CPU-to-GPU memory bottlenecks, we enforce strict buffer allocation patterns for our WebGL context.

- **Vertex Buffer Objects (VBOs):**
  - Allocated using `STATIC_DRAW` for static UI/map elements that rarely change.
  - Allocated using `DYNAMIC_DRAW` for highly interactive elements (e.g., active selections, real-time status indicators).
- **Element Array Buffers (EBOs):** Used extensively for indexed drawing to prevent vertex data duplication.
- **Update Strategy:** To prevent memory fragmentation, buffers are pre-allocated in large contiguous chunks. Dynamic updates are executed using `glBufferSubData` rather than reallocating buffers.

## 3. Spatial Quadtree Indexing

To avoid checking every single object on the screen during a render frame, we use a Spatial Quadtree to group objects by their 2D location.

- **Node Splitting:** A region splits into four child quadrants when it exceeds the maximum capacity of `[Insert Max Entities, e.g., 64]` entities.
- **Viewport Culling (Frustum Culling):** Before issuing WebGL draw calls, the current camera viewport (frustum) is checked against the quadtree. Only the quadrants intersecting with the camera view are sent to the GPU to be rendered.

## 4. Performance Benchmarking

Our hardware-accelerated pipeline is continuously profiled to ensure low latency.

| Metric                  | Target   | Actual (Latest Build) |
| :---------------------- | :------- | :-------------------- |
| **Frame Rate (FPS)**    | 60 FPS   | `[Insert FPS]`        |
| **Draw Calls / Frame**  | < 100    | `[Insert Draw Calls]` |
| **GPU Memory Usage**    | < 150 MB | `[Insert Mem Usage]`  |
| **Quadtree Query Time** | < 2 ms   | `[Insert Time]`       |
