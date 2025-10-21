# 🌊 Watermark Removal (WebGPU)
[![GitHub Pages](https://github.com/cziter15/watermark-removal-webgpu/actions/workflows/pages.yml/badge.svg)](https://cziter15.github.io/watermark-removal-webgpu/) [![Lines of Code](https://img.shields.io/endpoint?color=blue&url=https://ghloc.vercel.app/api/cziter15/watermark-removal-webgpu/badge?filter=.html$,.js$,.wgsl$)](https://github.com/cziter15/watermark-removal-webgpu)

> **Watermark Removal (WebGPU)** is a lightweight, client-side image restoration tool that removes watermarks while preserving image quality.  
> The core is a neural-network image-to-image model (encoder–decoder with skip connections) exported to ONNX and executed in the browser on WebGPU.

<p align="center">
  <a href="https://cziter15.github.io/watermark-removal-webgpu/" target="_blank">
    <img src="https://img.shields.io/badge/OPEN%20ON%20GITHUB%20PAGES-0078D7?style=for-the-badge&logo=github&logoColor=white" alt="Open on GitHub Pages">
  </a>
</p>

<img src="./public/header.png" alt="header" />

## ✨ What it is
- A neural-network image processing model built to remove watermarks while keeping fine details.
- Network architecture: encoder–decoder with skip connections (preserves texture and edges).
- Exported as ONNX and run in-browser using a WebGPU backend (for example, ONNX Runtime Web with WebGPU).

## 🧩 Usage (overview)
- Use the demo UI to load an image, mark or auto-detect the watermark region, and run removal locally in the browser.
- The app runs the ONNX model on the GPU (WebGPU) — no image data is sent to a server.

## 🔬 How it works (high level)
- Input image is prepared and normalized for the model.
- The ONNX model (encoder–decoder with skip connections) predicts restored pixels for the watermark region.
- The result is composited back onto the original image; small post-processing steps preserve seamless boundaries.
- Execution is performed via ONNX runtime in the browser using a WebGPU backend for GPU acceleration.

```mermaid
graph TD
    Input[Input<br/>3 × 256 × 256]

    Input --> enc1[enc1<br/>conv_block 3→64<br/>e1: 64 × 256 × 256]
    enc1 --> pool1[/MaxPool2d(2) → 64 × 128 × 128/]

    pool1 --> enc2[enc2<br/>conv_block 64→128<br/>e2: 128 × 128 × 128]
    enc2 --> pool2[/MaxPool2d(2) → 128 × 64 × 64/]

    pool2 --> enc3[enc3<br/>conv_block 128→256<br/>e3: 256 × 64 × 64]
    enc3 --> pool3[/MaxPool2d(2) → 256 × 32 × 32/]

    pool3 --> enc4[enc4<br/>conv_block 256→512<br/>e4: 512 × 32 × 32]
    enc4 --> pool4[/MaxPool2d(2) → 512 × 16 × 16/]

    pool4 --> bottleneck[bottleneck<br/>conv_block 512→1024<br/>b: 1024 × 16 × 16]

    %% Decoder path
    bottleneck --> up4[Upsample ×2 → 1024 × 32 × 32]
    up4 --> concat4[Concat (b_up + e4)<br/>1024 + 512 = 1536 × 32 × 32]
    e4 --> concat4
    concat4 --> dec4[dec4<br/>conv_block 1536→512<br/>d4: 512 × 32 × 32]

    dec4 --> up3[Upsample ×2 → 512 × 64 × 64]
    up3 --> concat3[Concat (d4_up + e3)<br/>512 + 256 = 768 × 64 × 64]
    e3 --> concat3
    concat3 --> dec3[dec3<br/>conv_block 768→256<br/>d3: 256 × 64 × 64]

    dec3 --> up2[Upsample ×2 → 256 × 128 × 128]
    up2 --> concat2[Concat (d3_up + e2)<br/>256 + 128 = 384 × 128 × 128]
    e2 --> concat2
    concat2 --> dec2[dec2<br/>conv_block 384→128<br/>d2: 128 × 128 × 128]

    dec2 --> up1[Upsample ×2 → 128 × 256 × 256]
    up1 --> concat1[Concat (d2_up + e1)<br/>128 + 64 = 192 × 256 × 256]
    e1 --> concat1
    concat1 --> dec1[dec1<br/>conv_block 192→64<br/>d1: 64 × 256 × 256]

    dec1 --> final[final_layer<br/>Conv2d 64→3 (1×1) → Output: 3 × 256 × 256]
    final --> Output[Output<br/>3 × 256 × 256]
```

## 📚 Notes on ONNX + WebGPU
- ONNX makes the model portable; ONNX Runtime Web supports a WebGPU execution provider that runs entirely client-side.
- Browser support for WebGPU is evolving — prefer recent Chromium-based builds or Safari TP with WebGPU enabled.
- Performance depends on image size, GPU capacity, and model complexity. Consider resizing images client-side for faster iterations.

## 🐞 Troubleshooting
- "WebGPU not available": ensure browser supports WebGPU and enable required flags if necessary.
- Model load/run errors: check browser console for ONNX runtime messages and verify model input shapes and pre-processing.
- Memory issues on large images: downscale input before running the model.

Enjoy experimenting with ONNX on WebGPU! 🚀