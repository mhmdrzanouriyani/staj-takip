Proje: SpikeEdge Telemetry
Gün: 16
Konu: Sliding Window, Z-Score Normalizasyonu ve Autoencoder Eğitim Verisinin Hazırlanması

1. GÜNÜN ÖZETİ

Bugün SpikeEdge Telemetry projesinde Machine Learning aşamasına geçiş için gerekli veri hazırlama işlemleri tamamlandı.

Gün 14'te oluşturulan Fixed Threshold Baseline ve Gün 15'te elde edilen evaluation sonuçları değiştirilmedi.

Gün 15 sonuçları:

F1 — temperature_spike: 86.0%
F2 — voltage_sag: 100.0%
F3 — current_surge: 100.0%
F4 — fan_degradation: 0.0%
F5 — sensor_drift: 27.0%
Overall Detection Rate: 53.3%

Gün 16'nın amacı bu baseline'ı geliştirmek değil, Autoencoder eğitimi için temiz, deterministik ve yalnızca normal sistem davranışını temsil eden bir eğitim veri seti hazırlamaktır.

Bu aşamada herhangi bir Autoencoder eğitilmemiştir.


2. GÜNÜN AMACI

Autoencoder'ın yalnızca normal sistem davranışını öğrenebilmesi için normal telemetry verilerinden eğitim örnekleri oluşturuldu.

Temel pipeline:

Normal Telemetry
      ↓
Sliding Window
      ↓
64 Timesteps
      ↓
6 Telemetry Channels
      ↓
384 Features
      ↓
Z-Score Normalization
      ↓
Frozen Training Dataset

Bu yapı Gün 17'de eğitilecek Autoencoder için kullanılacaktır.


3. KULLANILAN VERİ SETİ

Eğitim verisi olarak yalnızca:

data/day09-normal.csv

kullanıldı.

Toplam normal telemetry satırı:

800

Tüm satırlar:

label = normal

olarak doğrulandı.

Fault verileri eğitim verisi olarak kullanılmadı.

Özellikle:

data/day09-faults.csv

dosyasındaki veriler μ, σ veya training window oluşturmak için kullanılmadı.


4. KULLANILAN TELEMETRY KANALLARI

Autoencoder veri setinde toplam 6 telemetry channel kullanıldı.

Channel sırası sabit tutuldu:

1. temp_core
2. temp_ambient
3. voltage_in
4. current_draw
5. fan_rpm
6. cpu_load

Bu sıra mevcut DATASET_CHANNEL_KEYS tanımı ile aynıdır.

Channel sırasının sabit tutulması önemlidir çünkü daha sonraki Autoencoder aşamasında aynı feature sırasının korunması gerekmektedir.


5. SLIDING WINDOW

Telemetry verileri tek tek sample olarak kullanılmak yerine zaman içerisindeki davranışı temsil edecek şekilde Sliding Window yöntemiyle gruplandırıldı.

Window size:

64 timesteps

Channel count:

6

Her window:

64 × 6

boyutundadır.

Dolayısıyla her window toplam:

384 features

içermektedir.

Window yapısı:

[64][6]

şeklindedir.

Flatten edildiğinde:

[384]

olmaktadır.

Flatten index mantığı:

index = timestep × 6 + channelIndex

şeklinde tanımlandı.


6. OLUŞTURULAN WINDOW SAYISI

Normal dataset 800 satırdan oluşmaktadır.

Window size 64 ve stride 1 kullanıldığında:

800 - 64 + 1 = 737

training window oluşturuldu.

Toplam:

737 windows

elde edildi.

Window'lar arasında herhangi bir shuffle uygulanmadı.

Bu sayede telemetry zaman sırası korunmuş oldu.


7. Z-SCORE NORMALİZASYONU

Telemetry channel'larının farklı ölçeklerde olması nedeniyle Z-Score normalization uygulandı.

Kullanılan formül:

z = (x - μ) / σ

Burada:

x = gerçek telemetry değeri
μ = normal dataset ortalaması
σ = normal dataset standart sapması
z = normalize edilmiş değer

Önemli olarak μ ve σ yalnızca normal training dataset üzerinden hesaplandı.


8. NORMALİZASYON İSTATİSTİKLERİ

Population standard deviation kullanılarak hesaplanan değerler:

Channel              Mean (μ)       Std (σ)

temp_core            41.8849375     5.8003778
temp_ambient         23.5865625     0.0723537
voltage_in           12.1568250     0.0611998
current_draw          1.0658625     0.1740726
fan_rpm            1986.8525000   545.8843657
cpu_load              46.4131250    14.1483759

Bu değerler:

ml/normal_stats.json

dosyasında saklandı.

Bu dosya ilerleyen aşamalarda Python ve Browser tarafında aynı normalization değerlerinin kullanılabilmesi için frozen statistics olarak kullanılacaktır.


9. NORMALİZASYON VALIDATION

Normalize edilmiş tüm değerler kontrol edildi.

Sonuçlar:

Minimum Z: -3.178230
Maximum Z:  2.258868

NaN: No
Infinity: No
Zero-variance channel: None

Herhangi bir channel'ın standart sapması sıfır olmadığı için bütün channel'lar normal şekilde normalize edilebildi.


10. INVERSE NORMALİZASYON TESTİ

Normalization işleminin doğru çalıştığını doğrulamak için normalize edilmiş değerler tekrar orijinal ölçeğe dönüştürüldü.

Kullanılan formül:

x' = z × σ + μ

Toplam:

800 × 6

değer üzerinde test gerçekleştirildi.

Maksimum reconstruction error:

3.55 × 10⁻¹⁵

Kabul edilen tolerance:

1 × 10⁻⁹

Sonuç:

PASS

Bu sonuç normalization ve inverse normalization işlemlerinin doğru çalıştığını göstermektedir.


11. DATA LEAKAGE KONTROLÜ

Machine Learning aşamasında data leakage oluşmaması için veri kaynağı kontrol edildi.

Kontroller:

- Fault dataset training için kullanılmadı.
- Fault verileri μ hesaplamasında kullanılmadı.
- Fault verileri σ hesaplamasında kullanılmadı.
- Fault verileri training window oluşturmak için kullanılmadı.
- Normalization statistics yalnızca normal dataset üzerinden oluşturuldu.
- Gün 14 threshold sistemi değiştirilmedi.
- Gün 15 evaluation sonuçları değiştirilmedi.

Leakage validation:

PASS


12. DETERMINISTIC DATASET

Training dataset'in tekrar üretilebilir olması için preparation pipeline deterministic olarak tasarlandı.

Aynı input dataset ile pipeline iki kez çalıştırıldı.

İki çalıştırmanın sonuçları:

μ values: identical
σ values: identical
737 windows: identical

Herhangi bir random shuffle kullanılmadı.

Sonuç:

Determinism: PASS


13. OLUŞTURULAN ML DOSYALARI

Gün 16 kapsamında aşağıdaki dosyalar oluşturuldu:

src/lib/ml/constants.ts
src/lib/ml/zscore.ts
src/lib/ml/slidingWindow.ts
src/lib/ml/prepareTrainingDataset.ts
src/lib/ml/day16Report.ts
src/lib/ml/day16SelfTest.ts

scripts/run-day16-self-test.ts

Frozen ML artifacts:

ml/normal_stats.json
ml/training_windows.json
ml/day16-validation.json

Dokümantasyon:

Gun_16/README.md


14. DEĞİŞTİRİLEN DOSYALAR

Aşağıdaki dosyalar Gün 16 kapsamında güncellendi:

package.json
README.md

package.json içerisine Day 16 validation command eklendi:

npm run test:day16

Aşağıdaki önemli dosyalara dokunulmadı:

threshold-detector.ts
Day 15 evaluation logic

Böylece önceki baseline ve evaluation sonuçları korunmuş oldu.


15. VALIDATION SONUÇLARI

Gün 16 sonunda aşağıdaki kontroller başarıyla tamamlandı:

npx tsc --noEmit     → PASS
npm run lint         → PASS
npm run build        → PASS

npm run test:day14   → PASS
npm run test:day15   → PASS
npm run test:day16   → PASS

Day 15 sonuçları da aynı şekilde korunmuştur:

F1 → 86.0%
F2 → 100.0%
F3 → 100.0%
F4 → 0.0%
F5 → 27.0%
Overall → 53.3%


16. DATASET ÖZETİ

Gün 16 sonunda hazırlanan training dataset:

Source:
data/day09-normal.csv

Normal rows:
800

Window size:
64

Channels:
6

Features per window:
384

Total windows:
737

Stride:
1

Shuffle:
No

Normalization:
Z-Score

Training data:
Normal only

Pipeline:

800 Normal Rows
       ↓
Sliding Window
       ↓
737 Windows
       ↓
64 × 6
       ↓
384 Features
       ↓
Z-Score
       ↓
Autoencoder Training Dataset


17. ÖNEMLİ SINIRLAMALAR

Gün 16 yalnızca veri hazırlama aşamasıdır.

Henüz:

- Autoencoder eğitilmedi.
- Keras modeli oluşturulmadı.
- TensorFlow kullanılmadı.
- TensorFlow.js kullanılmadı.
- WASM backend kullanılmadı.
- Web Worker oluşturulmadı.
- Browser inference yapılmadı.
- Anomaly score hesaplanmadı.

Bu işlemler sonraki günlerde gerçekleştirilecektir.


18. GÜN 17'YE HAZIRLIK

Gün 16 sonunda Autoencoder eğitimi için gerekli normal training dataset hazırlanmış ve frozen normalization statistics oluşturulmuştur.

Bir sonraki aşamada:

384 Input Features
        ↓
64
        ↓
16
        ↓
64
        ↓
384

şeklinde bir Autoencoder mimarisinin Python/Keras kullanılarak normal telemetry davranışı üzerinde eğitilmesi planlanmaktadır.

Autoencoder'ın temel amacı normal sistem davranışını öğrenmek ve daha sonra normal davranıştan sapmaları reconstruction error üzerinden belirlemektir.

Gün 16 sonunda veri pipeline'ı Autoencoder training için hazır durumdadır.


19. SONUÇ

Gün 16'da SpikeEdge Telemetry projesinin Machine Learning veri hazırlama aşaması tamamlandı.

Normal telemetry verilerinden 737 adet 64-timestep window oluşturuldu.

Her window 6 telemetry channel ve toplam 384 feature içermektedir.

Z-Score normalization uygulanarak her channel için μ ve σ değerleri hesaplandı ve frozen olarak ml/normal_stats.json içerisinde saklandı.

Data leakage, determinism, normalization ve inverse-normalization kontrollerinin tamamı başarıyla tamamlandı.

Autoencoder training bu aşamada gerçekleştirilmedi.

Gün 17'de hazırlanan bu dataset kullanılarak Autoencoder modelinin eğitilmesine geçilecektir.
