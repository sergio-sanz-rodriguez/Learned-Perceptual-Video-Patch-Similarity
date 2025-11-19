# Learned Perceptual Video Patch Similarity (LPVPS) Metric

LPVPS is an extension of the original [Learned Perceptual Image Patch Similarity (LPIPS)](https://github.com/richzhang/PerceptualSimilarity/tree/master) metric, adapted to operate on video files instead of individual images.

LPIPS provides a learned, human-aligned measure of perceptual difference between two images. 
```
@inproceedings{zhang2018perceptual,
  title={The Unreasonable Effectiveness of Deep Features as a Perceptual Metric},
  author={Zhang, Richard and Isola, Phillip and Efros, Alexei A and Shechtman, Eli and Wang, Oliver},
  booktitle={CVPR},
  year={2018}
}
```

This project generalizes the idea to video by computing LPIPS on corresponding frame pairs and aggregating the results into a single similarity score.

## How LPVPS Works

Given two videos of equal length (reference and distorted):

1. The videos are decoded frame-by-frame (supporting any YUV/RGB-FFmpeg format, including 8-bit and 10-bit raw YUV) and converted to RGB.
2. LPIPS is computed for each pair of corresponding RGB frames.
3. Each LPIPS distance is transformed into a similarity score:

<p align="center">
  <img src="https://latex.codecogs.com/png.latex?s_i%20%3D%201%20-%20%5Ctext%7BLPIPS%7D%28f_i%5E%7Bref%7D%2C%20f_i%5E%7Bdist%7D%29)" />
</p>

4. The final LPVPS score is the average over all N frames:

<p align="center">
  <img src="https://latex.codecogs.com/png.latex?%5Ctext%7BLPVPS%7D%20%3D%20%5Cfrac%7B1%7D%7BN%7D%20%5Csum_%7Bi%3D1%7D%5EN%20s_i" />
</p>

## Interpretation

LPVPS is designed to behave similarly to SSIM, but using the learned perceptual sensitivity of LPIPS.

* LPVPS = 1.0 → Videos are perceptually identical
* LPVPS ≈ 0.8 → Very high similarity
* LPVPS ≈ 0.5 → Moderate differences visible
* LPVPS → 0 → Strong perceptual degradation

This contrasts with the original LPIPS score, where higher means more different.
LPVPS flips the interpretation to be more intuitive for video-quality assessment.

## Why LPVPS?

* Works directly on raw YUV formats (via FFmpeg)
* Supports 10-bit and 8-bit content
* No temporary PNG files required (memory-efficient streaming mode)
* GPU-accelerated perceptual scoring via PyTorch (with CPU fallback)
* Intuitive, similarity-oriented scale (compatible with SSIM/VMAF-style interpretation)

## Example Usage

```
python lpvps.py \
  --ref ref_video.yuv \
  --dist distorted_video.yuv \
  --size 1920x1080 \
  --in_pix_fmt yuv420p10le \
  --fps 60 \
  --use_gpu \
  --per_frame
```
Output:
```
...
597: LPVPS=0.801551
598: LPVPS=0.806944
599: LPVPS=0.803296
600: LPVPS=0.805074
===========================
Number of frame pairs: 600
Mean LPVPS: 0.806206
```

## Notes
* LPVPS does not model temporal coherence (like flicker or judder) directly. It is strictly a per-frame perceptual similarity averaged over time.
* For a more complete video-quality assessment, LPVPS can be combined with temporal metrics or motion-aware models.
* This project remains interoperable with the original LPIPS model weights and networks (AlexNet, VGG, SqueezeNet).

## Dependencies/Setup

### Installation
* Install PyTorch 1.0+ and torchvision fom http://pytorch.org

```bash
pip install -r requirements.txt
```

* Download and install FFmpeg from https://www.ffmpeg.org/download.html

* Clone this repo:
```bash
git clone https://github.com/sergio-sanz-rodriguez/PerceptualSimilarityVideo
cd PerceptualSimilarityVideo
```