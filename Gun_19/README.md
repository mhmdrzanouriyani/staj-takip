# Gün 19 — Günlük Çalışma Raporu

**Tarih:** 21 Ağustos 2026  
**Konu:** TensorFlow.js Modelinin Tarayıcıda Yüklenmesi ve Inference Hazırlığı

---

## 1. Günün amacı

Bugünkü amaç, Day 18'de TensorFlow.js formatına export edilen Autoencoder modelini tarayıcı ortamında çalıştırılabilir hale getirmekti.

Day 18'de model:

```text
Keras Autoencoder
384 → 16 → 384
        ↓
TensorFlow.js Export
        ↓
public/models/ae-v1/
```

şeklinde hazırlandı.

Day 19'da bir sonraki önemli adım olan browser-side inference akışını hazırlamaya odaklandım:

```text
Browser
   ↓
TensorFlow.js
   ↓
loadLayersModel()
   ↓
model.json
   ↓
384 değerlik telemetry window
   ↓
model.predict()
   ↓
Reconstruction
   ↓
MSE
```

---

## 2. Yapılan çalışma

Day 18'de oluşturulan TensorFlow.js model export'unun browser tarafından erişilebilir olması kontrol edildi.

Model dosyaları:

```text
public/models/ae-v1/
```

altında bulunuyor.

Ana model dosyası:

```text
public/models/ae-v1/model.json
```

şeklinde kullanılıyor.

Model boyutları korunuyor:

```text
Input:  384
Output: 384
Latent: 16
```

---

## 3. Browser-side model loading

Tarayıcı tarafında TensorFlow.js Layers API kullanılarak model yükleme akışı hazırlandı.

Temel çağrı:

```ts
const model = await tf.loadLayersModel(
  "/models/ae-v1/model.json"
);
```

Buradaki amaç modeli yeniden eğitmek değil, Day 18'de freeze edilen modeli inference için kullanmaktır.

Browser tarafındaki model:

- yeniden eğitilmiyor
- ağırlıkları değiştirilmiyor
- yeni dataset oluşturmuyor
- threshold üretmiyor

Sadece mevcut model üzerinde prediction gerçekleştiriyor.

---

## 4. 384 değerlik telemetry window

Model tek bir telemetry frame'i kullanmıyor.

Day 16'da oluşturulan yapı korunarak model:

```text
64 frames × 6 channels
        =
384 values
```

girişini bekliyor.

Akış:

```text
Telemetry Frames
      ↓
Window Builder
      ↓
64 frames
      ↓
6 channels
      ↓
384 values
      ↓
Autoencoder
```

şeklinde devam ediyor.

Bu aşamadaki temel amaç, 384 değerlik telemetry window'un modele doğru şekilde hazırlanmasını sağlamaktır.

---

## 5. Reconstruction Error

Autoencoder'ın temel çıktısı reconstruction'dır.

Akış:

```text
input
  ↓
Autoencoder
  ↓
reconstruction
```

sonrasında reconstruction error Mean Squared Error ile ölçülür:

```text
MSE = mean((input - reconstruction)²)
```

Bu değer ilerleyen aşamada anomaly detection için kullanılabilecek temel sinyaldir.

Day 19'da bu değer henüz Dashboard'da Anomaly Score olarak gösterilmedi.

---

## 6. Python ↔ Browser parity

Day 19'un önemli hedeflerinden biri, browser inference sonucunun Python tarafındaki frozen model sonucu ile karşılaştırılabilir olmasıdır.

Day 18'de oluşturulan probe sonuçları referans olarak tutuldu.

Örnek Python reconstruction MSE değerleri:

| Window | Python MSE |
|:------:|-----------:|
| 321 | 0.02067520 |
| 120 | 0.01639085 |
| 209 | 0.02538610 |

Browser tarafında aynı window'ların kullanılmasıyla model çıktılarının karşılaştırılması planlandı.

Amaç:

```text
Python Model
      ↓
Reference MSE

Browser TF.js Model
      ↓
Browser MSE

        ↓

Parity Check
```

oluşturmaktır.

Bu kontrol, modelin export/import sırasında davranışının değişmediğini doğrulamak için kullanılacaktır.

---

## 7. Model bütünlüğü

Day 18'de freeze edilen Keras modeline dokunulmadı.

Model:

```text
ml/autoencoder.keras
```

SHA-256:

```text
d365bb…3582a
```

olarak korunuyor.

TensorFlow.js export:

```text
public/models/ae-v1/
```

altında bulunuyor.

Export boyutu:

```text
212,172 bytes
≈ 207.2 KB
```

Model architecture:

```text
384 → 16 → 384
```

olarak korunuyor.

---

## 8. Henüz yapılmayanlar

Day 19'da özellikle bazı özellikleri henüz eklemedim.

Henüz:

- Dashboard anomaly score eklenmedi.
- Alarm mekanizması TensorFlow.js'e bağlanmadı.
- UI scoring yapılmadı.
- Threshold integration yapılmadı.
- Worker/WASM runtime eklenmedi.
- Browser inference sonucu canlı telemetry pipeline'a bağlanmadı.

Bunun nedeni önce modelin Python ile browser arasında aynı sonucu ürettiğini doğrulamaktır.

Bu doğrulama tamamlandıktan sonra inference çıktısı Dashboard'a bağlanabilir.

---

## 9. Mimari

Day 19 için hazırlanan AI inference mimarisi:

```text
Telemetry
    ↓
Window Builder
    ↓
64 × 6
    ↓
384 values
    ↓
TensorFlow.js
    ↓
loadLayersModel()
    ↓
Autoencoder
    ↓
384 reconstructed values
    ↓
MSE
```

Mevcut telemetry pipeline ise değişmeden kalıyor:

```text
WebSocket
    ↓
TelemetryClient
    ↓
Ring Buffer
    ↓
Dashboard
```

AI inference bu pipeline'ın üzerine ayrı bir analiz katmanı olarak ekleniyor.

---

## 10. Validation

Day 19 için kontrol noktaları:

```text
TF.js package: PASS
Model path: PASS
model.json: PASS
Model export found: PASS
Input shape: 384
Output shape: 384
Latent dimension: 16
Python probe data: AVAILABLE
Browser inference: IN PROGRESS
Python ↔ Browser parity: PENDING
```

---

## 11. Sonuç

Day 18'de hazırlanan Autoencoder modeli browser-side inference aşamasına taşındı.

Bugünkü çalışma özellikle model loading ve inference altyapısının hazırlanmasına odaklandı.

Henüz anomaly score veya alarm üretmedim.

Öncelik:

> Önce Python ve Browser sonuçlarının aynı olduğunu kanıtlamak, daha sonra inference'ı canlı telemetry pipeline'a bağlamak.

Bu yaklaşım sayesinde modelin export edilmesi ile gerçek zamanlı anomaly detection arasında kontrol edilebilir bir validation noktası oluşturuldu.

---

## 12. Day 20'ye hazırlık

Bir sonraki adım:

```text
Browser TF.js inference
        ↓
Python reference
        ↓
Parity validation
        ↓
Reconstruction MSE
        ↓
Live telemetry
```

olacak.

Parity doğrulaması tamamlandıktan sonra model çıktısının AI Monitor içerisindeki gerçek telemetry akışına bağlanması planlanıyor.

---

## Kısa ilerleme özeti

| Gün | Konu | Sonuç |
|:---:|------|--------|
| 14 | Fixed min/max + hysteresis | Deterministic baseline hazırlandı |
| 15 | Frozen evaluation | F1 86% · F2/F3 100% · F4 0% · F5 27% · FPR 0% |
| 16 | Z-score + window 64 | 737 × 384 normal-only dataset |
| 17 | Keras Autoencoder | `384 → 16 → 384` · val MSE ≈ 0.01711 |
| 18 | TensorFlow.js Export | `public/models/ae-v1/` · 207.2 KB |
| **19** | **Browser TF.js Inference** | **`loadLayersModel()` + 384-value inference hazırlığı** |
