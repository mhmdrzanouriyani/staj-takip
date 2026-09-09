# Gün 20 — Günlük Çalışma Raporu

**Tarih:** 22 Ağustos 2026  
**Konu:** Canlı Telemetri ile TensorFlow.js Autoencoder Inference ve Reconstruction MSE

---

## 1. Günün amacı

Bugünkü amaç, Day 19'da tarayıcı ortamında çalıştırılabilir hale getirilen TensorFlow.js Autoencoder modelini mevcut canlı telemetri akışına bağlamaktı.

Day 19'da model yükleme ve inference altyapısı hazırlanmıştı. Day 20'de bu altyapı canlı telemetry window üzerinden çalışacak şekilde sisteme eklendi.

Temel akış:

```text
WebSocket
    ↓
TelemetryClient
    ↓
Ring Buffer
    ↓
Son 64 frame
    ↓
Frozen Day 16 μ/σ
    ↓
64 × 6 = 384 değer
    ↓
TensorFlow.js Autoencoder
    ↓
Reconstruction
    ↓
MSE
    ↓
AI Monitor
```

Bugün henüz anomaly threshold veya NORMAL / ANOMALY sınıflandırması eklenmedi.

---

## 2. Mevcut telemetry pipeline korundu

Yeni bir simulator veya ikinci bir telemetry kaynağı oluşturulmadı.

Mevcut yapı aynen kullanıldı:

```text
WebSocket
    ↓
TelemetryClient
    ↓
Ring Buffer
```

Bu akıştan iki ayrı analiz katmanı çalışıyor:

```text
                    ┌── Day 14 ThresholdDetector
Telemetry ──────────┤
                    └── Day 20 Autoencoder
```

Day 14 ThresholdDetector değiştirilmedi ve Autoencoder onun yerine geçirilmedi.

---

## 3. 64 frame telemetry window

Autoencoder tek bir frame ile çalışmıyor.

Day 16'dan beri kullanılan yapı korundu:

```text
64 frames × 6 channels = 384 values
```

Kanal sırası:

```text
1. temp_core
2. temp_ambient
3. voltage_in
4. current_draw
5. fan_rpm
6. cpu_load
```

Flattening kuralı:

```text
index = timestep × 6 + channelIndex
```

En yeni 64 frame, mevcut ring buffer snapshot'ından zaman sırası korunarak alınıyor.

64 frame henüz oluşmadıysa inference yapılmıyor ve AI Monitor collecting durumunu gösteriyor.

---

## 4. Frozen normalization

Day 16'da oluşturulan normal-only μ/σ değerleri kullanılmaya devam ediyor.

Kaynak:

```text
ml/normal_stats.json
```

Runtime tarafındaki frozen karşılığı:

```text
src/lib/ml/frozenNormalStats.ts
```

İstatistikler runtime'da yeniden hesaplanmıyor.

Fault verileri normalization için kullanılmıyor.

Bu sayede Day 16 preprocessing'i ile Day 20 inference preprocessing'i aynı kalıyor.

---

## 5. Autoencoder inference

Day 19'da hazırlanan Autoencoder loader ve inference API yeniden kullanıldı.

Model yeniden yüklenmiyor:

```text
loadAutoencoder()
        ↓
cached model
        ↓
runAutoencoderInference()
```

Model:

```text
Input: 384
Latent: 16
Output: 384
```

olarak değişmeden kaldı.

Model yeniden eğitilmedi, ağırlıklar değiştirilmedi ve TensorFlow.js export yeniden oluşturulmadı.

---

## 6. Inference scheduling

Telemetri yaklaşık 10 Hz hızında geliyor.

Her render döngüsünde inference çalıştırmak yerine inference telemetry frame sequence üzerinden kontrol edildi.

Aynı `frame.seq` ikinci kez skorlanmıyor.

Aynı anda bir inference devam ederken yeni frame'ler gelirse gereksiz paralel prediction başlatılmıyor; en güncel window korunarak işlem devam ediyor.

Bu yapı duplicate inference'ı ve gereksiz UI yükünü azaltıyor.

---

## 7. Reconstruction MSE

Autoencoder'ın reconstruction çıktısı ile giriş arasındaki hata Mean Squared Error ile hesaplanıyor:

```text
MSE = mean((input - reconstruction)²)
```

Day 20'de bu değer yalnızca **Reconstruction MSE** olarak ele alınıyor.

Henüz:

```text
MSE → NORMAL
MSE → ANOMALY
```

şeklinde bir sınıflandırma yapılmadı.

Bunun nedeni anomaly threshold'un henüz veriye dayalı olarak belirlenmemiş olmasıdır.

---

## 8. AI Monitor

AI Monitor'a Reconstruction Error bölümü eklendi.

Gösterilen bilgiler:

```text
Reconstruction MSE
Current MSE
Window: 64 / 384
Model: Autoencoder v1
Status: Collecting / Inference / Ready / Error
```

Ayrıca son **90 MSE sample** için bounded bir canlı grafik eklendi.

Grafik sonsuza kadar büyümüyor ve başlangıçta yeterli veri yoksa boş durum düzgün şekilde gösteriliyor.

Anomaly Score ve Detection Status henüz aktif edilmedi.

---

## 9. Python ↔ Browser parity

Day 19'da oluşturulan reference probe değerleri Day 20'de tekrar doğrulandı.

Kabul edilen tolerans:

```text
|Python MSE − Browser MSE| ≤ 1e-4
```

Sonuçlar:

| Window | Python MSE | Browser MSE | |Δ| | Result |
|:------:|-----------:|------------:|------:|:------:|
| 321 | 0.02067520 | 0.02067520 | 7.025e-9 | PASS |
| 120 | 0.01639086 | 0.01639085 | 1.233e-8 | PASS |
| 209 | 0.02538610 | 0.02538609 | 1.901e-9 | PASS |

Tüm sonuçlar `1e-4` toleransının çok altında kaldı.

---

## 10. Performance

Day 20 validation çalışmasındaki inference süreleri:

| Window | Time |
|:------:|-----:|
| 321 | 16.54 ms |
| 120 | 2.25 ms |
| 209 | 1.57 ms |

Bu değerler **Node CPU** üzerinde `@tensorflow/tfjs` kullanılarak ölçüldü.

İlk window warm-up içeriyor.

Gerçek browser WebGL/WASM performansı bu çalışmada ölçülmedi ve bu nedenle browser latency için herhangi bir değer iddia edilmedi.

---

## 11. Hata yönetimi

Canlı inference dashboard'un geri kalanını bozmamalı.

Ele alınan durumlar:

- 64 frame'den az veri
- geçersiz telemetry değeri
- NaN
- Infinity
- model loading failure
- inference failure

Geçersiz veri durumunda sahte MSE oluşturulmuyor.

AI inference başarısız olsa bile mevcut telemetry akışının çalışmaya devam etmesi hedeflendi.

---

## 12. Eklenen dosyalar

```text
src/lib/ml/day20/windowBuilder.ts
src/lib/ml/day20/liveInference.ts
src/lib/ml/useLiveReconstructionMse.ts

src/components/dashboard/ReconstructionMseChart.tsx

src/lib/ml/day20SelfTest.ts
scripts/run-day20-self-test.ts

ml/day20-validation.json
Gun_20/README.md
```

---

## 13. Değiştirilen dosyalar

```text
src/components/dashboard/AiMonitorPanel.tsx
src/components/dashboard/DashboardShell.tsx
package.json
README.md
```

`DashboardShell.tsx` içerisinde AI Monitor dynamic import olarak tutuldu. Böylece TensorFlow.js ilk yükleme JavaScript paketine gereksiz şekilde eklenmedi.

---

## 14. Değiştirilmeyen sistemler

Aşağıdaki bölümlere dokunulmadı:

```text
Day 14 ThresholdDetector
Day 15 evaluation
Day 16 normalization / window methodology
Day 17 Keras Autoencoder
Day 18 TensorFlow.js export
Day 19 browser inference layer
Keras model weights
public/models/ae-v1/
ml/normal_stats.json
Telemetry simulator
WebSocket source
Digital Twin
Existing alarm logic
```

Autoencoder mevcut threshold sisteminin yerine geçirilmedi.

---

## 15. Validation

Day 20 sonunda yapılan kontroller:

| Komut | Sonuç |
|---|---|
| `npx tsc --noEmit` | PASS |
| `npm run lint` | PASS |
| `npm run build` | PASS |
| `npm run test:day14` | PASS |
| `npm run test:day15` | PASS |
| `npm run test:day16` | PASS |
| `npm run test:day17` | PASS |
| `npm run test:day18` | PASS |
| `npm run test:day19` | PASS |
| `npm run test:day20` | PASS |

Build sonrası first-load JavaScript yaklaşık **125 kB** seviyesinde kaldı.

---

## 16. Day 20 sınırlamaları

Bu aşamada henüz yapılmadı:

- MSE anomaly threshold
- NORMAL / ANOMALY classification
- F1–F5 fault identification
- Alarm generation from Autoencoder
- Web Worker
- Browser WebGL/WASM latency measurement
- Autoencoder retraining
- Live fault prediction

Bunların eklenmesinden önce reconstruction MSE dağılımının normal telemetry üzerinde ölçülmesi gerekiyor.

---

## 17. Sonuç

Day 20 ile birlikte Day 19'da tarayıcıya taşınan Autoencoder artık mevcut canlı telemetry pipeline üzerinden gerçek 64-frame window alıp reconstruction MSE üretebiliyor.

En önemli doğrulama noktaları:

```text
Live Telemetry
      ↓
64 × 6
      ↓
384 values
      ↓
Frozen μ/σ
      ↓
TF.js Autoencoder
      ↓
Reconstruction MSE
      ↓
AI Monitor
```

Python ↔ Browser parity kontrolü başarıyla geçti ve önceki Day 14–19 testleri bozulmadı.

Henüz MSE'yi anomaly olarak yorumlamıyorum. Önce normal çalışma sırasında oluşan MSE dağılımının ölçülmesi gerekiyor.

---

## 18. Day 21'e hazırlık

Bir sonraki adım:

```text
Normal telemetry windows
        ↓
Autoencoder
        ↓
Reconstruction MSE distribution
        ↓
Threshold selection
        ↓
Justified anomaly boundary
```

Day 21'de amaç, normal window'lardan reconstruction MSE dağılımını çıkarmak ve seçilen threshold'u ölçülebilir bir gerekçeyle belgelemektir.

Bu threshold, Day 14 ThresholdDetector'ın yerine geçmeyecek; iki sistem ayrı analiz katmanları olarak çalışmaya devam edecektir.

---

## Kısa ilerleme özeti

| Gün | Konu | Sonuç |
|:---:|---|---|
| 14 | Fixed min/max + hysteresis | Deterministic baseline hazırlandı |
| 15 | Frozen evaluation | F1 86% · F2/F3 100% · F4 0% · F5 27% · FPR 0% |
| 16 | Z-score + window 64 | 737 × 384 normal-only dataset |
| 17 | Keras Autoencoder | `384 → 16 → 384` · val MSE ≈ 0.01711 |
| 18 | TensorFlow.js Export | `public/models/ae-v1/` · 207.2 KB |
| 19 | Browser TF.js Inference | `loadLayersModel()` + browser inference |
| **20** | **Live Telemetry + Reconstruction MSE** | **64 × 6 → 384 → Autoencoder → MSE → AI Monitor** |
