# Pose Estimation on Human Spine

A computer vision pipeline that analyzes spinal posture from standard video recordings — no wearables, no multi-camera rigs.

Built as part of a research project at Kent State University's SCI Laboratory using ~25GB of real lab footage.

---

## What It Does

The system takes raw video, runs Google's MediaPipe Pose on every frame, extracts joint angles (elbow, knee, hip), and classifies each frame as low, medium, or high bending severity. Output lands in a structured CSV — one row per frame, one column per feature.

The spine angle classification uses three posture thresholds tied to hip flexion:
- **Low** — mostly upright
- **Medium** — moderate forward lean or mid-transition
- **High** — deep flexion, the kind that strains things

---

## Pipeline Overview

```
Raw video (MP4)
    ↓
Frame extraction (OpenCV)
    ↓
Landmark detection (MediaPipe Pose — 33 keypoints)
    ↓
Joint angle calculation (3-point vector geometry)
    ↓
Confidence filtering + smoothing
    ↓
Posture classification (Low / Medium / High)
    ↓
Structured CSV export
    ↓
ML classification (Random Forest / XGBoost / SVM)
```

---

## Dataset

- ~25GB of human movement video from Kent State's SCI Laboratory
- Activities: hip flexion, forward bending, sitting transitions, general upper/lower-body movement
- Multiple subjects, varied camera angles, natural lighting conditions

Videos range from hundreds to thousands of frames each. The dataset was organized by activity type, then labeled by bending severity using hip-angle thresholds derived during preprocessing.

No ground-truth frame-level annotations existed — which is why the angle-based approach made more sense than training a supervised classifier from scratch.

---

## Model Approach

### Why angles instead of a deep learning classifier?

Annotating 25GB of video frame-by-frame isn't realistic. More importantly, ergonomic posture analysis already has a language: joint angles, degrees of flexion, torso alignment. An angle-based approach produces numbers that a clinician or ergonomist can actually interpret, not a probability score from a black box.

### Pose estimation

MediaPipe Pose detects 33 body landmarks per frame. It runs on CPU, handles varied lighting and non-frontal angles reasonably well, and outputs confidence scores that the preprocessing step uses to filter unreliable frames.

### Angle calculation

Each joint angle is calculated from three landmarks using vector math:

- **Elbow**: shoulder → elbow → wrist
- **Knee**: hip → knee → ankle
- **Hip/Torso**: torso-to-hip alignment (primary bending indicator)

Additional features include torso lean, shoulder symmetry, and pelvis symmetry.

### SpinePose integration

The project incorporates anatomical framing from [SpinePose](https://saifkhichi96.github.io/research/spinepose/) — specifically its emphasis on mid-torso and spinal curvature landmarks. SpinePose wasn't used for full inference (computational overhead), but its anatomical definitions shaped how the torso/hip angles are interpreted and provide a roadmap for future vertebral tracking.

---

## Results

Three classifiers trained on the extracted angle features (585 samples × 88 features), evaluated with 5-fold cross-validation:

| Model | Accuracy | Macro F1 |
|---|---|---|
| Random Forest | 77% | 0.76 |
| XGBoost | 75% | 0.75 |
| SVM (RBF) | 69% | 0.69 |

Random Forest performed best overall. Group 2 (medium bending) was consistently the hardest to classify — it shares angle characteristics with both ends of the spectrum, which is a known problem in multi-class posture systems.

---

## Comparison with Prior Work

| Approach | Sensors | Limitation |
|---|---|---|
| This project | Single RGB camera | 2D only, no explicit vertebral keypoints |
| IMU + CV hybrid | Dual IMU + RGB | Requires wearable sensors |
| Anatomy-aware 3D pose | RGB camera | Needs curated labeled training data |
| SpinePose | RGB camera | More complex model, heavier compute |
| Multiview 3D | Multiple cameras | Requires synchronized multi-camera setup |
| Fall detection (pose-based) | RGB camera | Good at sequences, not bending angles |

The trade-off here is precision vs. accessibility. This pipeline runs on a laptop with a single camera and produces interpretable output. You give up 3D spinal curvature; you gain something you can actually deploy in a real lab.

---

## Limitations

- MediaPipe gives 2D keypoints — no sagittal-plane curvature, no 3D spine alignment
- Spinal posture is estimated indirectly through hip and torso angles, not vertebral landmarks
- Performance drops with occlusion, harsh lighting, or non-frontal body orientation (confidence filtering reduces but doesn't eliminate this)
- No ground-truth posture labels, so evaluation is limited to cross-validation consistency
- The current classifier doesn't model posture transitions over time — each frame is classified independently

---

## Future Work

- Add SpinePose or anatomy-aware models for explicit vertebral tracking
- Hybrid 2D + IMU approach for 3D posture without multi-camera systems
- Temporal modeling (LSTM, GRU, or Transformer) for transition detection and sustained bending analysis
- Context-aware analysis with object detection (desk, phone, chair)
- Real-time deployment for workplace ergonomics monitoring

---

## Authors

**Mohith Reddy Seelam** — Department of Computer Science & Mathematical Sciences, Kent State University
mseelam1@kent.edu

**JungYoon Kim** — Department of Computer Science, Kent State University
jkim78@kent.edu

---

## References

1. [Upper Body Spine Orientation Estimation](https://doi.org/10.2197/ipsjtcva.7.121)
2. [Wearable 3D Posture Estimation for Lower Back Healthcare](https://ieeexplore.ieee.org/document/9611778)
3. [Anatomy-Aware 3D Human Pose Estimation](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9347537)
4. [SpinePose: Towards Unconstrained 2D Pose Estimation of the Human Spine](https://saifkhichi96.github.io/research/spinepose/)
5. [Multiview 3D Markerless Human Pose Estimation from OpenPose Skeletons](https://link.springer.com/chapter/10.1007/978-3-030-40605-9_15)
6. [Pose Estimation-Based Fall Detection via AI Edge Computing](https://ieeexplore.ieee.org/document/9540831)
7. [Reconstructing Humans with a Biomechanically Accurate Skeleton](https://arxiv.org/pdf/2503.21751)
