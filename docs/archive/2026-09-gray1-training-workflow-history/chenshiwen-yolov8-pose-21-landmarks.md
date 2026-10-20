# chenshiwen 21-landmark YOLOv8-Pose Training Notes

Checked at: 2026-09-08 14:51 CST  
Remote access used: `ssh H200-chenshiwen`  
Remote user: `ai-i-chenshiwen`  
Remote host: `user-Rack-Server`  
Scope: read-only inspection; no files or configuration under chenshiwen paths were changed.

## Main Training Project

The active 21-landmark YOLOv8-Pose training project is:

```text
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine
```

Important files:

```text
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/train_yolov8x_p6_pose.py
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/test_yolov8x_p6_pose.py
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/ultralytics/
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/ultralytics/yolov8x-pose-p6.pt
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/logs/
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/runs/
```

The training script imports the local vendored Ultralytics checkout instead of the global Python package:

```text
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/ultralytics
```

The training log reports:

```text
Ultralytics 8.3.98
Python 3.12.12
torch 2.8.0+cu128
CUDA devices used: 0,1,3
GPU model: NVIDIA H200 NVL
```

## Python Environment

The environment that matches the successful training log is:

```text
/home/ai-i-chenshiwen/miniconda3/envs/leo
```

Observed package checks from that environment:

```text
python: 3.12.12
torch: 2.8.0+cu128
cv2: 4.11.0
numpy: 1.26.4
albumentations: 2.0.8
```

There is also an `emlandmark` environment at:

```text
/home/ai-i-chenshiwen/miniconda3/envs/emlandmark
```

But it did not import `torch`, `cv2`, `numpy`, or `albumentations` during inspection, so it does not appear to be the training environment for this YOLOv8-Pose run.

## Dataset

The dataset YAML used by the run is:

```text
/data2/ai-i-chenshiwen/auto_landmark/Dataset/yolo_full/dataset.yaml
```

Observed content:

```yaml
path: /data2/ai-i-chenshiwen/auto_landmark/Dataset/yolo_full
train: images/train
val: images/val
test: images/test
kpt_shape: [2, 3]
flip_idx: [0, 1]
r_channel_only: true
online_augmentations: false
names:
  - agaier
  - aipuha
  - aituosha
  - ansainusa
  - aoerhong
  - chapala
  - dibulong
  - didikaka
  - kanjialu
  - lupate
  - qinghai
  - rineiwa
  - sading
  - saersaer
  - saleikamei
  - taihu
  - taiwan
  - tana
  - tekesi
  - wennibeige
  - moon
```

This is a 21-class landmark dataset. The pose keypoint shape is `[2, 3]`, meaning each object has 2 keypoints, each with x/y/visibility.

## Dataset Scale and Storage

Read-only counts observed for:

```text
/data2/ai-i-chenshiwen/auto_landmark/Dataset/yolo_full
```

Image and label counts:

| split | images | labels | non-empty labels | empty labels |
|---|---:|---:|---:|---:|
| train | 52,772 | 52,772 | 45,466 | 7,306 |
| val | 6,634 | 6,634 | 5,581 | 1,053 |
| test | 6,600 | 6,600 | 5,533 | 1,067 |
| total | 66,006 | 66,006 | 56,580 | 9,426 |

Storage usage:

| path | observed size |
|---|---:|
| full dataset | 824G |
| images/train | 661G |
| labels/train | 180M |
| images/val | 83G |
| labels/val | 23M |
| images/test | 81G |
| labels/test | 22M |

Approximate image+label split totals:

| split | size |
|---|---:|
| train | 660.73 GiB |
| val | 82.04 GiB |
| test | 80.72 GiB |

The labels are tiny compared with the images; storage is dominated by PNG image files.

## Image Inventory

A per-image inventory was generated locally from read-only remote file header scans:

```text
E:\codex\quantitize\docs\train\yolo_full_image_inventory.csv
```

CSV columns:

```text
split, relative_path, size_bytes, width, height, error
```

Summary:

| item | value |
|---|---:|
| total images | 66,006 |
| read errors | 0 |
| image format | all `.png` |
| 4096x4096 images | 64,640 |
| 2048x2048 images | 1,366 |

Observed file-size distribution:

| split | images | total size | avg size | min size | max size |
|---|---:|---:|---:|---:|---:|
| train | 52,772 | 660.72 GiB | 12.82 MiB | 16.01 KiB | 22.03 MiB |
| val | 6,634 | 82.04 GiB | 12.66 MiB | 16.29 KiB | 21.99 MiB |
| test | 6,600 | 80.72 GiB | 12.52 MiB | 19.11 KiB | 22.00 MiB |

Current interpretation:

- Earth landmark images are effectively the 4096x4096 group.
- Moon images are effectively the 2048x2048 group.
- Both groups are square PNGs, so the training resize path does not need asymmetric letterbox padding for these images.

## Preprocessing Before YOLO

The training run uses:

```yaml
imgsz: 1024
multi_scale: false
online_augmentations: false
r_channel_only: true
cache: false
```

The effective image tensor entering the YOLO model is:

```text
shape: [B, 3, 1024, 1024]
dtype: float32
range: 0.0 - 1.0
channel semantic: RGB tensor with only R containing grayscale intensity; G/B are zero
```

The preprocessing chain in the local Ultralytics checkout is:

```text
PNG file
-> cv2.imread(...), loaded as BGR
-> load_image(): resize longest side to imgsz=1024
   4096x4096 -> 1024x1024
   2048x2048 -> 1024x1024
-> LetterBox(new_shape=(1024, 1024))
   no extra padding for the current square images
-> RChannelOnly()
   B=0, G=0, keep the existing R channel as grayscale intensity
-> Format()
   HWC -> CHW and channel flip BGR -> RGB
-> trainer preprocess_batch()
   move to GPU, float32, divide by 255
-> YOLOv8x-pose-p6 model
```

Relevant local modifications in the vendored Ultralytics checkout:

```text
ultralytics/ultralytics/data/dataset.py
  if self.augment and not self.data.get("online_augmentations", True):
      transforms = Compose([LetterBox(new_shape=(self.imgsz, self.imgsz))])
  ...
  if self.data.get("r_channel_only", False):
      transforms.append(RChannelOnly())
```

```text
ultralytics/ultralytics/data/augment.py
  class RChannelOnly:
      img[..., :2] = 0  # B=0, G=0; existing R channel is grayscale intensity.
```

Because `imgsz=1024` and `multi_scale=false`, both 4096x4096 landmark images and 2048x2048 moon images are fed to the model as 1024x1024 tensors during this training run.

## Known Training Run

Main run directory:

```text
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/runs/landmarks_yolov8x_p6_pose_pretrained_1024_no_online_aug_b64_run2
```

Training arguments observed from `args.yaml`:

```yaml
task: pose
mode: train
model: Yolo_fine/ultralytics/yolov8x-pose-p6.pt
data: /data2/ai-i-chenshiwen/auto_landmark/Dataset/yolo_full/dataset.yaml
epochs: 150
batch: 192
imgsz: 1024
device: 0,1,3
workers: 8
project: /data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/runs
name: landmarks_yolov8x_p6_pose_pretrained_1024_no_online_aug_b64_run2
save_period: 10
amp: true
cache: false
online augmentations: disabled in script and dataset config
```

Weights found in the run:

```text
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/runs/landmarks_yolov8x_p6_pose_pretrained_1024_no_online_aug_b64_run2/weights/best.pt
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/runs/landmarks_yolov8x_p6_pose_pretrained_1024_no_online_aug_b64_run2/weights/last.pt
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/runs/landmarks_yolov8x_p6_pose_pretrained_1024_no_online_aug_b64_run2/weights/epoch*.pt
```

`best.pt` and `last.pt` were both observed with timestamp `2026-08-25 19:58` and size `199053642` bytes.

## Evaluation Artifacts

Evaluation logs and outputs found:

```text
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/logs/test_landmarks_yolov8x_p6_pose_best_gpu0.log
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/logs/test_landmarks_yolov8x_p6_pose_best_sample100_gpu0.log
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/runs/test_landmarks_yolov8x_p6_pose_best
/data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/runs/test_landmarks_yolov8x_p6_pose_best_sample100
```

The full test log reported 6600 selected test images and 5533 labeled instances. Summary line from the observed log:

```text
all 6600 images, 5533 instances, Box mAP50 0.957, Box mAP50-95 0.901, Pose mAP50 0.963, Pose mAP50-95 0.958
```

The sample100 detailed test log reported:

```text
detection_precision_at_match_iou: 0.9572852396772663
detection_recall_at_match_iou: 0.9604761904761905
keypoint_prediction_recall: 0.9604761904761905
keypoint_mae_px_on_predicted: 21.716809797398543
pck@0.05: 0.9178571428571428
pck@0.1: 0.9488095238095238
pck@0.2: 0.9604761904761905
```

## Related Candidate Paths

Additional related paths were found but appear secondary or older compared with `Yolo_fine`:

```text
/data1/chenshiwen/landmarks_detect
/data1/chenshiwen/landmarks_detect/landmarks2_aituosha_moon
/data1/chenshiwen/landmarks_detect/Yolo_Sam2
/data2/ai-i-chenshiwen/auto_landmark/Yolo_Sam2
/data2/ai-i-chenshiwen/auto_landmark/Albumentations
/data2/ai-i-chenshiwen/auto_landmark/备份/Yolo_fine
```

Notable older/smaller candidate:

```text
/data1/chenshiwen/landmarks_detect/landmarks2_aituosha_moon
```

It contains a local `ultralytics_v8.3.98`, `yolov8x-pose-p6.pt`, and pose runs, but its observed `data.yaml` is for a single `moon` class with `kpt_shape: [2, 3]`, so it is not the 21-class training project.

## Reproduction Pointer

To reproduce from the identified project, the likely entry shape is:

```bash
ssh H200-chenshiwen
conda activate leo
cd /data2/ai-i-chenshiwen/auto_landmark
python Yolo_fine/train_yolov8x_p6_pose.py \
  --data /data2/ai-i-chenshiwen/auto_landmark/Dataset/yolo_full/dataset.yaml \
  --weights /data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/ultralytics/yolov8x-pose-p6.pt \
  --epochs 150 \
  --imgsz 1024 \
  --batch 192 \
  --device 0,1,3 \
  --workers 8 \
  --project /data2/ai-i-chenshiwen/auto_landmark/Yolo_fine/runs \
  --name landmarks_yolov8x_p6_pose_pretrained_1024_no_online_aug_b64_run2 \
  --save-period 10
```

Before actually running this, check GPU ownership/utilization and confirm authorization, because H200 is a shared server and the original run used three GPUs.
