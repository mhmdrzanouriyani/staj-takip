GÜN 17 — AUTOENCODER EĞİTİMİ

Proje: SpikeEdge Telemetry
Gün: 17
Konu: Normal Telemetry Verileri ile Autoencoder Modelinin Eğitilmesi

1. GÜNÜN ÖZETİ

Bugün SpikeEdge Telemetry projesinde Machine Learning aşamasının temel modeli olan Autoencoder eğitildi.

Gün 16'da hazırlanan ve yalnızca normal sistem davranışını içeren frozen training dataset kullanıldı.

Gün 14 Fixed Threshold Baseline ve Gün 15 evaluation sonuçları değiştirilmedi.

Gün 15 sonuçları:

F1 — temperature_spike: 86.0%
F2 — voltage_sag: 100.0%
F3 — current_surge: 100.0%
F4 — fan_degradation: 0.0%
F5 — sensor_drift: 27.0%
Overall Detection Rate: 53.3%

Bu sonuçlar Day 17 training sürecinde kullanılmadı ve model bu fault verileriyle eğitilmedi.

Day 17'nin temel amacı, normal telemetry davranışını öğrenebilen bir Autoencoder modeli oluşturmaktır.


2. GÜNÜN AMACI

Autoencoder yalnızca normal telemetry verileri üzerinde eğitildi.

Modelin amacı normal davranışı öğrenmek ve giriş verisini mümkün olduğunca doğru şekilde yeniden oluşturmaktır.

Temel çalışma mantığı:

Normal Telemetry
      ↓
Autoencoder
      ↓
Reconstruction
      ↓
Reconstruction Error

Normal davranış model tarafından iyi öğrenildiğinde reconstruction error düşük olacaktır.

Daha sonraki aşamalarda modele normal davranıştan farklı telemetry verileri verildiğinde reconstruction error değerinin yükselmesi anomaly detection için kullanılacaktır.

Day 17'de henüz anomaly threshold belirlenmemiştir ve F1–F5 fault evaluation yapılmamıştır.


3. KULLANILAN DATASET

Model eğitimi için yalnızca Gün 16'da hazırlanmış dataset kullanıldı:

ml/training_windows.json

Toplam:

737 windows

Her window:

64 timesteps × 6 channels

Toplam feature sayısı:

384

Bu nedenle Autoencoder input shape:

(384,)

şeklindedir.

Fault dataset training sürecinde kullanılmadı.


4. TRAINING / VALIDATION SPLIT

737 normal window deterministik şekilde iki bölüme ayrıldı:

Training:
589 windows

Validation:
148 windows

Split oranı:

80% Training
20% Validation

Shuffle işlemi seed = 1337 kullanılarak gerçekleştirildi.

Training ve validation verileri yalnızca normal telemetry davranışını içermektedir.


5. AUTOENCODER MİMARİSİ

Proje roadmap'inde belirtilen Autoencoder mimarisi kullanıldı:

384 → 64 → 16 → 64 → 384

Detaylı yapı:

Input
384 features
    ↓
Dense(64, ReLU)
    ↓
Dense(16, ReLU)
    ↓
Dense(64, ReLU)
    ↓
Dense(384, Linear)
    ↓
Output
384 features

16 boyutlu katman modelin latent representation alanıdır.

Bu alan, 384 boyutlu telemetry bilgisinin daha küçük bir temsilini öğrenmektedir.


6. MODELİN ÇALIŞMA MANTIĞI

Autoencoder giriş olarak 384 feature içeren bir telemetry window alır.

Model daha sonra bu veriyi:

384 → 64 → 16

şeklinde sıkıştırır.

Ardından:

16 → 64 → 384

şeklinde tekrar genişleterek orijinal veriyi yeniden oluşturmaya çalışır.

Training sırasında:

Input = X
Target = X

şeklinde kullanıldı.

Yani model kendi girişini yeniden oluşturmaya çalışmaktadır.

Kullanılan loss:

Mean Squared Error (MSE)


7. TRAINING KONFİGÜRASYONU

Python sürümü:

3.11.16

Python ortamı:

ml/.venv

TensorFlow:

2.19.1

Optimizer:

Adam

Learning Rate:

0.001

Loss:

MSE

Batch Size:

32

Epoch:

60

Random Seed:

1337

Python, NumPy ve TensorFlow için seed ayarları kullanıldı.

Bit-for-bit determinism garanti edilmemiştir ancak aynı eğitim çalıştırması önceki çalışma ile aynı sonuçları üretmiştir.


8. PYTHON ORTAMI

Makinede yalnızca Python 3.14 bulunuyordu.

TensorFlow mevcut Python 3.14 sürümünü desteklemediği için global Python kurulumu değiştirilmedi.

Bunun yerine proje içerisinde Python 3.11.16 kullanılarak local virtual environment oluşturuldu:

ml/.venv

Bu ortam proje dışında herhangi bir global Python kurulumunu değiştirmemektedir.

Gerekli Python bağımlılıkları proje içerisinde:

ml/requirements.txt

dosyasında tanımlandı.


9. TRAINING SONUÇLARI

Model toplam 60 epoch boyunca eğitildi.

Final Training Loss:

0.01681758

Final Validation Loss:

0.01710983

Loss değerlerinde:

NaN: Yok
Infinity: Yok

bulundu.

Training süreci başarıyla tamamlandı.


10. NORMAL VALIDATION RECONSTRUCTION ERROR

Modelin normal validation datasetindeki reconstruction performansı ayrıca ölçüldü.

Toplam validation window:

148

Reconstruction MSE sonuçları:

Mean:
0.01710983

Median:
0.01663261

Minimum:
0.00762946

Maximum:
0.02610665

Standard Deviation:
0.00489986


11. RECONSTRUCTION PROBE TESTLERİ

Deterministik birkaç validation window ayrıca test edildi.

Window index 321:

MSE = 0.02067520

Window index 120:

MSE = 0.01639086

Window index 209:

MSE = 0.02538610

Bu testler modelin normal telemetry windowlarını yeniden oluşturabildiğini doğrulamak amacıyla gerçekleştirildi.


12. MODEL VALIDATION

Model shape kontrolleri:

Input:

(384,)

Output:

(384,)

Latent dimension:

16

Model normal telemetry verilerini başarıyla reconstruct edebildi.

Training loss finite olarak kaldı.

NaN veya Infinity oluşmadı.


13. OLUŞTURULAN MODEL ARTIFACTLARI

Gün 17 kapsamında aşağıdaki önemli dosyalar oluşturuldu:

ml/autoencoder.keras

ml/training_history.json

ml/model_summary.txt

ml/day17-validation.json

ml/day17-ci-validation.json

Bu dosyalar Autoencoder modelinin training sonuçlarını ve validation bilgilerini saklamaktadır.


14. OLUŞTURULAN KOD DOSYALARI

Training pipeline için:

ml/train_autoencoder.py

Self-test için:

ml/day17_self_test.py

Project test integration:

scripts/run-day17-self-test.ts


15. PYTHON ENVIRONMENT DOSYALARI

Aşağıdaki dosyalar da oluşturuldu:

ml/requirements.txt

ml/.python-version

Virtual environment:

ml/.venv/

ml/.venv/ Git tarafından ignore edilmektedir.


16. DEĞİŞTİRİLEN DOSYALAR

Gün 17 kapsamında:

package.json
README.md
.gitignore

dosyaları güncellendi.

package.json içerisine:

npm run test:day17

komutu eklendi.

.gitignore içerisine:

ml/.venv/
__pycache__/

eklenerek local Python environment ve cache dosyalarının Git repository'ye gönderilmesi engellendi.


17. ÖNEMLİ KORUMALAR

Gün 17 sırasında:

- Day 14 threshold değerleri değiştirilmedi.
- Day 15 evaluation logic değiştirilmedi.
- Day 16 normalizasyon istatistikleri değiştirilmedi.
- Day 16 training windows değiştirilmedi.
- Fault dataset training için kullanılmadı.
- F1–F5 sonuçları model training sırasında kullanılmadı.
- Fault performance'a göre model tuning yapılmadı.
- TensorFlow.js eklenmedi.
- Browser inference yapılmadı.
- Web Worker oluşturulmadı.
- WASM backend kullanılmadı.


18. VALIDATION SONUÇLARI

Aşağıdaki kontroller başarıyla tamamlandı:

npx tsc --noEmit     → PASS
npm run lint         → PASS
npm run build        → PASS

npm run test:day14   → PASS
npm run test:day15   → PASS
npm run test:day16   → PASS
npm run test:day17   → PASS

Day 14 ve Day 15 sonuçları değişmeden korunmuştur.


19. DAY 17 SINIRLAMALARI

Day 17 yalnızca Autoencoder training aşamasıdır.

Bu nedenle henüz:

- Anomaly threshold belirlenmedi.
- F1–F5 fault detection yapılmadı.
- Fixed Threshold ile Autoencoder karşılaştırılmadı.
- TensorFlow.js conversion yapılmadı.
- Browser inference yapılmadı.
- Web Worker oluşturulmadı.
- WASM backend kullanılmadı.
- Dashboard'a ML anomaly score bağlanmadı.

Bu işlemler sonraki günlerde gerçekleştirilecektir.


20. GÜN 18'E HAZIRLIK

Day 17 sonunda eğitilmiş Autoencoder modeli:

ml/autoencoder.keras

konumunda hazır durumdadır.

Gün 18'in amacı bu Keras/TensorFlow modelini browser ortamında kullanılabilecek TensorFlow.js formatına dönüştürmektir.

Planlanan akış:

Keras Model
    ↓
TensorFlow Model Conversion
    ↓
model.json
    +
Binary Weight Files
    ↓
TensorFlow.js
    ↓
Browser Inference

Day 18'de model yeniden eğitilmemelidir.

Mevcut ml/autoencoder.keras artifact'ı conversion için kullanılmalıdır.


21. SONUÇ

Gün 17'de SpikeEdge Telemetry projesinin ilk Autoencoder modeli başarıyla eğitildi.

Model yalnızca normal telemetry davranışından oluşan 737 window üzerinde çalıştırıldı.

Model mimarisi:

384 → 64 → 16 → 64 → 384

Final Training Loss:

0.01681758

Final Validation Loss:

0.01710983

Normal validation reconstruction MSE:

Mean: 0.01710983
Median: 0.01663261
Min: 0.00762946
Max: 0.02610665
Std: 0.00489986

Model başarıyla kaydedildi ve validation testleri PASS oldu.

Day 17 sonunda model browser ortamına taşınmaya hazırdır.

Bir sonraki aşama Gün 18'de TensorFlow/Keras modelinin TensorFlow.js formatına dönüştürülmesidir.
