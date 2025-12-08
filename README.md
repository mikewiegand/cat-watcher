 CatWatcher — Local Cat Recognition & Smart Door System

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

```
ESP32-S3 Camera → Thor Ingest API → YOLOv11n Detector (TRT)
                → Embedding Model → Similarity Search → Profile Match
                → UDLS Logs → Access Control → Door Controller
```

Thor manages the entire inference and decision pipeline.  
Embedding vectors and event logs are stored in local MongoDB collections.

For multi-cat frames, each detected cat is processed separately, and policy rules determine whether door access is allowed.

---

## UDLS (Unified Decision Log Specification)

All important decisions — detection, identity resolution, access control, drift detection — are recorded following the **UDLS v4** specification.

📘 **UDLS Repository:**  
https://github.com/<your-user>/unified-decision-log-spec

This provides structured, replayable audit trails for:
- `cat_frame_classified`
- `cat_identity_resolved`
- `door_access_cat`
- `unknown_cat_handled`
- `model_drift_detected`

---

## Repo Structure (Proposed)

```
catwatcher/
  docs/
    thor-cat-model-design-v1.md
    use-cases.md
    system-architecture.md

  server/
    api/
      main.py
    cat_model/
      cat_model.py
      yolov11n.plan
      embedding_model.onnx

  esp32/
    camera/
      main.py

  schemas/
    mongo/
      cat_profiles.schema.json
      cat_events.schema.json

  udls/
    catwatcher-udls-extensions-v1.md
```

---

## Status

Active development.  
Initial components include the YOLOv11n detector pipeline, embedding engine, multi-cat recognition logic, access control policy, and UDLS integration.

---

## License

MIT (or your preferred license)
