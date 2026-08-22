GÜN 15 — SABİT EŞİK TABANLI ANOMALİ DEĞERLENDİRMESİ

Proje: SpikeEdge Telemetry
Gün: 15
Konu: Fixed Threshold Baseline Evaluation ve Fault Detection Analizi

1. GÜNÜN AMACI

Bugün, Gün 14'te oluşturulan Fixed Threshold yaklaşımının mevcut fault senaryoları üzerindeki performansı değerlendirildi.

Amaç, mevcut eşik tabanlı sistemin hangi arıza türlerini tespit edebildiğini ve hangi durumlarda yetersiz kaldığını ölçmekti.

Bu aşamada threshold değerleri değiştirilmedi ve sonuçları iyileştirmek amacıyla yeniden ayarlanmadı.

Değerlendirme, mevcut veri setleri kullanılarak deterministik ve tekrar üretilebilir şekilde gerçekleştirildi.

Bu çalışma aynı zamanda ilerleyen günlerde geliştirilecek Autoencoder modeli için temel karşılaştırma (baseline) noktası olarak kullanılacaktır.


2. GÜN 14'TE OLUŞTURULAN BASELINE

Gün 14'te oluşturulan baseline yalnızca normal telemetry verilerinden öğrenildi.

Kullanılan normal veri:

data/day09-normal.csv

Normal veri sayısı:

800 sample

İzlenen telemetry kanalları:

- temp_core
- temp_ambient
- voltage_in
- current_draw
- fan_rpm
- cpu_load

Her kanal için normal çalışma aralığı belirlendi ve %5 hysteresis uygulandı.

Önemli nokta:

Fault verileri threshold oluşturma aşamasında kullanılmadı.

Bu sayede evaluation sırasında data leakage önlendi.


3. EVALUATION YÖNTEMİ

Gün 15 evaluation işlemi aşağıdaki akış üzerinden gerçekleştirildi:

Normal Telemetry Data
        ↓
Frozen Day 14 Baseline
        ↓
Threshold Detector
        ↓
Fault Dataset
        ↓
F1–F5 Evaluation
        ↓
Detection Metrics

Her fault grubu için yeni bir ThresholdDetector instance'ı kullanıldı.

Her senaryo için:

- toplam sample sayısı
- tespit edilen sample sayısı
- kaçırılan sample sayısı
- detection rate

hesaplandı.

Bunun yanında genel olarak:

- Detection Rate / Recall
- False Positive Rate
- Precision
- F1 Score

hesaplandı.


4. TEST EDİLEN FAULT SENARYOLARI

Gün 15 kapsamında toplam 5 fault senaryosu değerlendirildi:

F1 — temperature_spike
F2 — voltage_sag
F3 — current_surge
F4 — fan_degradation
F5 — sensor_drift

Toplam fault sample:

300

Normal sample:

800


5. FAULT DETECTION SONUÇLARI

F1 — temperature_spike

Sample: 50
Detected: 43
Missed: 7
Detection Rate: 86.0%


F2 — voltage_sag

Sample: 40
Detected: 40
Missed: 0
Detection Rate: 100.0%


F3 — current_surge

Sample: 50
Detected: 50
Missed: 0
Detection Rate: 100.0%


F4 — fan_degradation

Sample: 60
Detected: 0
Missed: 60
Detection Rate: 0.0%


F5 — sensor_drift

Sample: 100
Detected: 27
Missed: 73
Detection Rate: 27.0%


6. GENEL SONUÇLAR

Normal sample: 800
Fault sample: 300

Detected fault: 160
Missed fault: 140

Detection Rate / Recall:

53.3%

False Positive Rate:

0.0%

Precision:

100.0%

F1 Score:

69.6%


7. F1 — TEMPERATURE SPIKE ANALİZİ

F1 senaryosunda toplam 50 sample bulunmaktadır.

43 sample başarıyla tespit edilirken 7 sample kaçırılmıştır.

Detection Rate:

86.0%

Kaçırılan örneklerin temel nedeni, fault başlangıcındaki bazı sıcaklık değerlerinin henüz normal maksimum threshold değerini aşmamış olmasıdır.

Bu nedenle detector fault tamamen belirginleşmeden önce her durumda alarm üretmemektedir.


8. F2 — VOLTAGE SAG ANALİZİ

F2 senaryosundaki 40 sample'ın tamamı tespit edilmiştir.

Detection Rate:

100.0%

Voltage değerindeki düşüş doğrudan normal çalışma aralığının dışına çıktığı için Fixed Threshold yöntemi bu fault türünde başarılı olmuştur.


9. F3 — CURRENT SURGE ANALİZİ

F3 senaryosundaki 50 sample'ın tamamı tespit edilmiştir.

Detection Rate:

100.0%

Current değerindeki ani yükselme threshold sınırını geçtiği için detector tarafından başarılı şekilde alarm olarak işaretlenmiştir.


10. F4 — FAN DEGRADATION ANALİZİ

Gün 15'in en önemli sonucu F4 senaryosudur.

Toplam 60 F4 sample'ının hiçbirisi Fixed Threshold tarafından tespit edilmemiştir.

Detected:

0

Missed:

60

Detection Rate:

0.0%

Bunun temel nedeni, fan hızındaki düşüşün tek başına fan_rpm kanalının normal sınırlarını aşmamasıdır.

Normal fan_rpm aralığı yaklaşık:

882 – 2516 RPM

şeklindedir.

Fan hızında yaklaşık %55 oranında düşüş olmasına rağmen değer hâlâ normal aralık içerisinde kalabilmektedir.

Buradaki gerçek problem fan hızının mutlak değeri değil, fan hızının sistem yükü ve sıcaklığı ile olan ilişkisidir.

Örneğin:

CPU Load yüksek
+
Temperature yüksek
+
Fan RPM beklenenden düşük

olduğunda sistem davranışı anormal olabilir.

Ancak her channel ayrı ayrı değerlendirildiğinde tüm değerler normal görünebilir.

Bu nedenle:

Single-Channel Threshold → NORMAL

Actual System Behavior → FAULT

durumu ortaya çıkmaktadır.

Bu sonuç, tek değişkenli threshold yaklaşımının multivariate anomaly durumlarında yetersiz kalabileceğini göstermektedir.


11. F5 — SENSOR DRIFT ANALİZİ

F5 senaryosunda toplam 100 sample bulunmaktadır.

27 sample tespit edilmiş ve 73 sample kaçırılmıştır.

Detection Rate:

27.0%

Sensor drift başlangıçta yavaş ilerlediği için değerler uzun süre normal threshold aralığında kalabilmektedir.

Detector çoğunlukla temp_core değerinin threshold sınırını aşmasından sonra alarm üretmektedir.

Bu durum yavaş gelişen anomalilerin sabit threshold yöntemiyle erken tespit edilmesinin zor olduğunu göstermektedir.


12. THRESHOLD İÇERİSİNDE KALAN FAULT'LAR

Evaluation sonucunda özellikle F4 senaryosu dikkat çekmiştir.

F4'teki 60 sample'ın tamamı her bir channel için belirlenen minimum ve maksimum threshold değerleri içerisinde kalmıştır.

Yani sistem:

Telemetry değerleri → Normal

olarak değerlendirilirken sistem davranışı aslında:

Actual Behavior → Fault

durumundadır.

Bu durum, anomaly'nin tek bir telemetry channel'ında değil, birden fazla channel arasındaki ilişkide oluştuğunu göstermektedir.


13. ÇOK DEĞİŞKENLİ ANOMALİ AÇISINDAN SONUÇ

Gün 15 sonuçları proje hipotezini desteklemektedir.

Fixed Threshold yöntemi, tek bir channel'ın normal sınırlarını aşan fault'larda oldukça başarılıdır.

Örnek:

F2 → 100%
F3 → 100%
F1 → 86%

Ancak fault sistemin birden fazla telemetry channel'ı arasındaki ilişkide oluşuyorsa performans ciddi şekilde düşmektedir.

En önemli örnek:

F4 → 0%

Bu nedenle yalnızca bağımsız min/max threshold'lara dayanan bir sistem, sistem davranışındaki daha karmaşık anomalileri yakalamakta yetersiz kalabilir.


14. FALSE POSITIVE DEĞERLENDİRMESİ

Evaluation sırasında False Positive Rate:

0.0%

olarak ölçülmüştür.

Ancak burada önemli bir sınırlama bulunmaktadır.

False Positive hesabında kullanılan normal veri seti, aynı zamanda threshold baseline oluşturmak için kullanılan normal veri setidir.

Bu nedenle %0 False Positive sonucu bağımsız bir validation dataset üzerinde elde edilmiş bir sonuç değildir.

İlerleyen aşamalarda ayrı bir normal validation dataset kullanılarak daha güvenilir bir False Positive değerlendirmesi yapılabilir.


15. DATA LEAKAGE KORUMASI

Gün 15 evaluation sürecinde:

- Fault verileri threshold oluşturmak için kullanılmadı.
- Gün 14 threshold değerleri freeze edildi.
- Her fault grubu fresh detector ile test edildi.
- Detection Rate'i yükseltmek amacıyla threshold değerleri değiştirilmedi.
- Bu aşamada Autoencoder veya başka bir ML modeli kullanılmadı.

Bu nedenle elde edilen sonuçlar mevcut Frozen Baseline'ın gerçek performansını göstermektedir.


16. OLUŞTURULAN DOSYALAR

src/lib/threshold/thresholdEvaluation.ts
src/lib/threshold/day15EvaluationReport.ts
src/lib/threshold/day15SelfTest.ts
scripts/run-day15-self-test.ts

data/evaluation/day15-threshold-results.json
data/evaluation/day15-validation.json

Gun_15/README.md


17. DEĞİŞTİRİLEN DOSYALAR

package.json
README.md

threshold-detector.ts dosyası Gün 15 kapsamında değiştirilmedi.


18. VALIDATION SONUÇLARI

Aşağıdaki kontroller başarıyla tamamlandı:

npx tsc --noEmit  → PASS
npm run lint      → PASS
npm run build     → PASS

test:day08        → PASS
test:day09        → PASS
test:day10        → PASS
test:day11        → PASS
test:day12        → PASS
test:day13        → PASS
test:day14        → PASS
test:day15        → PASS


19. GÜN 15 SONUCU

Gün 15 sonunda Fixed Threshold yaklaşımının güçlü ve zayıf yönleri ölçülebilir şekilde ortaya konmuştur.

Sonuç:

Overall Detection Rate:

53.3%

Precision:

100.0%

F1 Score:

69.6%

Ancak özellikle F4 fan degradation senaryosunda:

Detection Rate:

0.0%

olmuştur.

Bu sonuç, yalnızca tek tek telemetry channel'larının minimum ve maksimum değerlerini kontrol etmenin bazı sistem anomalilerini tespit etmek için yeterli olmadığını göstermektedir.

Özellikle fan RPM, CPU Load ve Temperature gibi değişkenlerin birbirleriyle olan ilişkisi dikkate alınmalıdır.


20. GÜN 16'YA GEÇİŞ

Gün 15 ile birlikte Fixed Threshold yaklaşımı için baseline evaluation tamamlandı.

Bir sonraki aşamada proje Rule-Based Detection'dan Machine Learning tabanlı anomaly detection yaklaşımına geçecektir.

Gün 16'nın temel amacı Autoencoder eğitimi için veri hazırlamaktır.

Planlanan pipeline:

Normal Telemetry
        ↓
Sliding Window
        ↓
64 Timesteps
        ↓
6 Channels
        ↓
384 Features
        ↓
Z-Score Normalization
        ↓
Mean (μ) / Standard Deviation (σ)
        ↓
Autoencoder Training Dataset

Gün 16'da henüz Autoencoder eğitilmeyecektir.

Model training işlemi Gün 17'de gerçekleştirilecektir.

Gün 15'in temel çıktısı:

Fixed Threshold = Baseline

ve bu baseline daha sonra Autoencoder performansı ile karşılaştırılacaktır.
