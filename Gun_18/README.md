SPIKEEDGE TELEMETRY
DAY 18 — DÖNÜŞÜM TAMAMLAMA RAPORU
=================================

DURUM: TAMAMLANDI
TARİH: 2026-08-23


1. YAPILAN ÇALIŞMALAR
---------------------

Day 18 dönüşüm işlemi başarıyla tamamlandı.

Day 17'de dondurulmuş olan Keras modeli:

ml/autoencoder.keras

resmî TensorFlow.js Layers formatına dönüştürüldü:

public/models/ae-v1/

Day 17 Keras modeli yeniden eğitilmedi.

SHA-256:
d365bb…3582a

Orijinal .keras dosyası değiştirilmedi veya üzerine yazılmadı.

Windows üzerinde TensorFlow 2.19.1 ile tam `pip install tensorflowjs`
kurulumu mümkün değildir. Bunun nedeni tensorflow-decision-forests
paketinin uyumlu bir wheel paketine sahip olmamasıdır.

Bu nedenle:

tensorflowjs==4.22.0

`--no-deps` parametresi ile kuruldu.

Resmî `save_keras_model` dönüşüm yöntemi, JAX/TFDF olmadan kullanıldı.

Ara HDF5 dosyası yalnızca geçici dönüştürme dosyası olarak kullanıldı.

Day 17 self-test kodu da mevcut model varsa dondurulmuş modeli yeniden
kullanacak şekilde güncellendi.

Böylece:

npm run test:day17

komutu modelin sessiz şekilde yeniden eğitilmesine neden olamaz.


2. KULLANILAN SÜRÜMLER
----------------------

Python:
3.11.16 (ml/.venv)

TensorFlow:
2.19.1

Keras:
3.15.1

TensorFlow.js Converter:
4.22.0

@tensorflow/tfjs npm paketi eklenmedi.


3. DÖNÜŞTÜRÜLEN MODEL
---------------------

Konum:

public/models/ae-v1/

Dosyalar:

model.json
- TensorFlow.js Layers modeli
- Model topology mevcut

group1-shard1of1.bin
- 206,912 byte
- 51,728 adet float32 ağırlık

Model mimarisi:

Input:   384
Latent:   16
Output:  384


4. MODEL BOYUTU
---------------

Toplam model boyutu:

212,172 byte
yaklaşık 207.2 KB

Hedef:

300 KB'dan küçük

Sonuç:

PASS

Herhangi bir quantization kullanılmadı.


5. PROBE DOĞRULAMASI
--------------------

Probe window'ları Day 16 sıralamasından alınmıştır:

321 / 120 / 209

Python reconstruction MSE sonuçları:

Window 321:
0.02067520

Window 120:
0.01639085

Window 209:
0.02538610

Probe verileri Day 19/20 için aşağıdaki dosyada kaydedildi:

ml/day18-probes.json

Day 18 sırasında browser üzerinde `predict` işlemi yapılmadı.


6. TEST SONUÇLARI
-----------------

npx tsc --noEmit
PASS

npm run lint
PASS

npm run build
PASS

test:day14
PASS

test:day15
PASS

Day 15 değerlendirmesi:

F1: 86.0%
F2: 100.0%
F3: 100.0%
F4: 0.0%
F5: 27.0%

Genel sonuç:
53.3%

test:day16
PASS

test:day17
PASS

Dondurulmuş model yeniden kullanıldı.

test:day18
PASS


7. OLUŞTURULAN DOSYALAR
-----------------------

ml/convert_to_tfjs.py
ml/tfjs_converter_loader.py
ml/day18_self_test.py
scripts/run-day18-self-test.ts

public/models/ae-v1/model.json
public/models/ae-v1/group1-shard1of1.bin

ml/day18-probes.json
ml/day18-validation.json
ml/day18-ci-validation.json

Gun_18/README.md


8. DEĞİŞTİRİLEN DOSYALAR
------------------------

package.json
README.md
.gitignore
ml/requirements.txt
ml/day17_self_test.py

.gitignore içerisine aşağıdaki dosyanın korunması için gerekli kural
eklendi:

!/public/models/ae-v1/*.bin

Day 17 self-test, dondurulmuş modelin yeniden kullanılacağı şekilde
güncellendi.


9. DEĞİŞTİRİLMEYEN DOSYALAR VE BÖLÜMLER
----------------------------------------

ml/autoencoder.keras

Day 14 detector

Day 15 evaluation logic

Day 16 μ/σ normalization

Training history

GLB dosyaları

Worker implementasyonu

WASM implementasyonu

UI scoring


10. DAY 18 KAPSAMI
------------------

Day 18 tamamlandı.

Aşağıdaki özellikler özellikle Day 18 kapsamında uygulanmadı:

- Browser TensorFlow.js runtime
- Web Worker
- WASM backend
- Browser model prediction
- Dashboard anomaly score
- Browser alarm üretimi

Bu özellikler Day 19 ve Day 20 kapsamında ele alınacaktır.


11. DAY 19 DURUMU
----------------

Day 19 henüz uygulanmadı.

Şu anda mevcut olmayan özellikler:

- Browser TF.js runtime
- Web Worker
- WASM backend
- Dashboard anomaly score

Day 19 kapsamında:

public/models/ae-v1/model.json

yüklenerek browser inference sonuçları:

ml/day18-probes.json

içerisindeki Python referans sonuçları ile karşılaştırılacaktır.


12. DAY 19 SONRAKİ ADIM
-----------------------

Bir sonraki aşama browser tarafında inference sisteminin
uygulanmasıdır.

Beklenen akış:

Browser
   |
   v
Web Worker
   |
   v
TensorFlow.js
   |
   v
model.json
   |
   v
model.predict()
   |
   v
Reconstruction
   |
   v
MSE
   |
   v
Browser doğrulaması

Day 19 doğrulamasında aşağıdaki probe window'ları kullanılmalıdır:

Window 321
Window 120
Window 209

Browser tarafında elde edilen MSE değerleri Python referans değerlerine
çok yakın olmalıdır.

Yalnızca küçük floating-point farklılıkları kabul edilebilir.

ÖNEMLİ:

Browser inference Python modeli ile doğrulanmadan önce alarm veya
threshold mantığı uygulanmamalıdır.


13. SON DURUM
-------------

DAY 18: TAMAMLANDI

Model dönüşümü:             PASS
Model boyutu hedefi:        PASS
Python doğrulaması:         PASS
TypeScript:                 PASS
Lint:                       PASS
Production build:           PASS
Day 14 regression:          PASS
Day 15 regression:          PASS
Day 16 regression:          PASS
Day 17 regression:          PASS
Day 18 self-test:           PASS


SONRAKİ AŞAMA:

DAY 19 — Browser TensorFlow.js Runtime + Web Worker
