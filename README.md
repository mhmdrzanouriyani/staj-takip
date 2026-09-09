# SpikeEdge — Industrial Digital Twin & AI Anomaly Detection

Real-time telemetry, Digital Twin, and AI-based anomaly detection prototype.

## Development Progress

| Gün | Çalışma | Durum |
|:---:|---|:---:|
| 01 | Proje altyapısı ve temel sistem | ☑️ |
| 02 | Telemetry veri yapısı | ☑️ |
| 03 | Simulator ve veri üretimi | ☑️ |
| 04 | WebSocket telemetry bağlantısı | ☑️ |
| 05 | Dashboard altyapısı | ☑️ |
| 06 | Live telemetry gösterimi | ☑️ |
| 07 | Digital Twin 3D model altyapısı | ☑️ |
| 08 | Telemetry ve fault testleri | ☑️ |
| 09 | Normal telemetry dataset | ☑️ |
| 10 | İstatistik ve analiz yardımcıları | ☑️ |
| 11 | Live WebSocket telemetry pipeline | ☑️ |
| 12 | Fault simulation / F1–F5 | ☑️ |
| 13 | Sistem validation ve entegrasyon | ☑️ |
| 14 | Fixed threshold + hysteresis | ☑️ |
| 15 | Frozen anomaly evaluation | ☑️ |
| 16 | Z-score + 64-frame window | ☑️ |
| 17 | Keras Autoencoder eğitimi | ☑️ |
| 18 | TensorFlow.js model export | ☑️ |
| 19 | Browser-side TF.js inference | ☑️ |
| 20 | Live telemetry + Reconstruction MSE | ☑️ |
| 21 | MSE threshold ve anomaly boundary | ⬜ |

## Current Architecture

```text
Telemetry
    ↓
WebSocket
    ↓
Ring Buffer
    ├── Threshold Detector
    └── 64-frame Window
            ↓
        Frozen μ/σ
            ↓
      TensorFlow.js
        Autoencoder
            ↓
      Reconstruction MSE
            ↓
        AI Monitor
```

## Current Status

**Days 01–20 completed.**

Current focus: **Autoencoder-based anomaly detection and threshold selection.**

Detailed daily reports are available inside:

```text
Gun_01/
Gun_02/
...
Gun_20/
```
