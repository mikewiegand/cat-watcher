CatWatcher — Local Cat Recognition & Smart Door 
CatWatcher is a fully local, privacy-preserving cat recognition and door-access system built on **Thor**, **ESP32-S3 camera nodes**, **YOLOv11n detection**, **embedding-based identity**, and **UDLS decision logging**.  
No cloud dependencies — all inference, matching, storage, and policy enforcement run on your LAN.

---

## Features

- **Multiple-cat detection** using YOLOv11n (TensorRT on Thor)
- **Identity embeddings** (ResNet / MobileNet) with cosine similarity matching
- **Authorized cat profiles** stored in MongoDB (Mochi, etc.)
- **Policy-driven door access** (time windows, authorized identities)
- **Multi-cat scene handling** (safe logic for visitor cats)
- **UDLS-backed audit logs** for every detection, match, and door decision
- **ESP32-S3 camera ingest** via simple HTTP endpoint
- **Offline-first** operation — no external services required

---

## Architecture Overview

```mermaid
flowchart TD
  ESP32[ESP32-S3 Camera Node] --> Ingest[Thor Ingest API]
  Ingest --> Detector[YOLOv11n Detector (TRT)]
  Detector --> Embed[Embedding Model]
  Embed --> Match[Similarity Search & Profile Match]
  Match --> UDLS[UDLS Logger]
  Match --> Access[Access Control Policy]
  Access --> Door[Door Controller]
  UDLS --> Mongo[MongoDB Event & Decision Storage]
```
