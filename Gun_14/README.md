# 📊 Gün 14 - Günlük Çalışma Raporu

Tarih: 16 Ağustos 2026
Konu: Sabit Eşik Baseline ve Alarm Mekanizması

---

## 1. Günün Özeti

Stajın on dördüncü gününde SpikeEdge Telemetry projesinde, daha önce
oluşturulan normal telemetry dataset'ini kullanarak sabit eşik tabanlı
bir baseline sistemi geliştirdim.

Bu çalışmadaki amacım herhangi bir makine öğrenmesi veya yapay zeka
modeli kullanmadan, telemetry değerlerinin normal çalışma aralığında
olup olmadığını kontrol edebilen basit, deterministik ve açıklanabilir
bir alarm mekanizması oluşturmaktı.

Baseline oluşturmak için Day 09'da hazırlanan:

data/day09-normal.csv

dosyasını kullandım.

Fault içeren verileri baseline hesabına dahil etmedim.

---

## 2. Günün Amacı

Day 14 kapsamında şu çalışmaları yaptım:

- Normal telemetry dataset'inden baseline oluşturdum.
- Her kanal için minimum ve maksimum değerleri belirledim.
- Kanal bazlı threshold kontrolü ekledim.
- Threshold çevresindeki gereksiz alarm değişimlerini azaltmak için
  hysteresis mekanizması ekledim.
- Threshold dışına çıkan değerler için alarm durumu oluşturdum.
- Normal ve fault verilerinin birbirine karışmasını engelledim.
- Baseline hesaplamasının deterministik olduğunu test ettim.
- F1-F5 fault senaryolarının bu baseline üzerinden değerlendirilebilmesi
  için gerekli altyapıyı hazırladım.

Bu aşamada sistem fault tipini tanımıyor. Sadece telemetry değerinin
normal baseline'ın dışında olup olmadığını kontrol ediyor.

---

## 3. Normal Dataset'ten Baseline Oluşturulması

Baseline yalnızca normal telemetry verilerinden oluşturuldu.

Kullandığım akış:

Normal Dataset
      ↓
Normal Rows
      ↓
Channel Statistics
      ↓
Min / Max
      ↓
Fixed Threshold Baseline
      ↓
Telemetry Evaluation

Baseline oluşturulurken sadece:

label = "normal"

olan kayıtlar kullanılıyor.

Fault etiketli kayıtlar baseline hesaplamasına dahil edilmiyor.

Ayrıca Day 10'da oluşturulan mevcut channel statistics yapısını tekrar
kullandım. Böylece aynı istatistik hesaplama mantığını devam ettirmiş
oldum.

---

## 4. Kullanılan Telemetry Kanalları

Baseline mevcut altı telemetry kanalı için oluşturuldu:

- temp_core
- temp_ambient
- voltage_in
- current_draw
- fan_rpm
- cpu_load

Her kanal için ayrı bir minimum ve maksimum threshold oluşturuluyor.

Örneğin:

temp_core
    min → normal çalışma aralığının alt sınırı
    max → normal çalışma aralığının üst sınırı

Aynı mantık diğer kanallar için de uygulanıyor.

---

## 5. Threshold Mantığı

Detector'ın temel mantığını mümkün olduğunca basit tuttum.

Bir değer:

value > max

olduğunda:

ALARM

durumuna geçiyor.

Aynı şekilde:

value < min

olduğunda:

ALARM

durumuna geçiyor.

Değer normal aralık içerisindeyse:

min <= value <= max

NORMAL olarak değerlendiriliyor.

Detector burada fault tipini belirlemiyor.

Örneğin sıcaklık threshold'un dışına çıktığında sistem:

F1

demiyor.

Sadece:

above_max

veya

below_min

gibi bir reason döndürüyor.

---

## 6. Hysteresis Mekanizması

Threshold değerinin hemen çevresinde küçük dalgalanmalar olduğunda
sistemin sürekli:

ALARM
NORMAL
ALARM
NORMAL

şeklinde değişmesini önlemek için hysteresis mekanizması ekledim.

Hysteresis oranını:

DEFAULT_HYSTERESIS_RATIO = 0.05

olarak belirledim.

Yani her kanalın normal veri aralığının %5'i hysteresis olarak
kullanılıyor.

Örneğin üst threshold:

max = 46

ve hysteresis:

1

ise değer 46'nın üzerine çıktığında alarm oluşuyor.

Alarmdan tekrar normal duruma dönebilmesi için değerin:

46 - 1 = 45

seviyesinin altına dönmesi gerekiyor.

Alt threshold için de aynı mantığın tersi uygulanıyor.

Kullanılan mantık:

value > max
    ↓
ALARM

value < max - hysteresis
    ↓
NORMAL'e dönüş

ve alt tarafta:

value < min
    ↓
ALARM

value > min + hysteresis
    ↓
NORMAL'e dönüş

Threshold ile recovery threshold arasında kalındığında mevcut alarm
durumu korunuyor.

---

## 7. Alarm Durumları

Day 14'te detector için iki temel state kullandım:

NORMAL
ALARM

Her sonuçta ayrıca durumun nedeni belirtiliyor:

- within_baseline
- above_max
- below_min
- recovering

Örneğin:

temp_core
value: 55
state: ALARM
reason: above_max

şeklinde bir sonuç alınabilir.

Bu yapıda F1-F5 gibi fault ID'leri üretilmiyor.

Çünkü Day 14'ün amacı fault classification yapmak değil, basit bir
threshold baseline oluşturmaktır.

---

## 8. F1-F5 ile Değerlendirme

Mevcut FaultEngine içerisinde bulunan beş fault tipi baseline üzerinden
değerlendirilebilir hale getirildi:

F1 - temperature_spike
F2 - voltage_sag
F3 - current_surge
F4 - fan_degradation
F5 - sensor_drift

Detector sadece telemetry değerlerini değerlendiriyor.

F1 için temperature değerinin baseline maksimumunu aşabilmesi kontrol
edildi.

F2 için voltage değerinin baseline minimumunun altına düşebilmesi
kontrol edildi.

F3, F4 ve F5 de değerlendirmeye alındı ancak bu fault'ların mutlaka
alarm üretmesi beklenmedi.

Bu özellikle önemli çünkü sabit min/max baseline bazı fault'ları
kaçırabilir.

Örneğin bir fault normal min/max aralığının içerisinde kalıyorsa bu
yöntem tarafından tespit edilmeyebilir.

Bu durum Day 15'te gerçek performans değerlendirmesi yapılırken
ölçülecek.

---

## 9. Veri Sızıntısının Önlenmesi

Day 14'te dikkat ettiğim en önemli konulardan biri baseline'ın fault
verilerinden etkilenmemesiydi.

Doğru akış:

Normal Dataset
      ↓
Baseline
      ↓
Frozen Thresholds
      ↓
Fault Dataset
      ↓
Evaluation

Fault dataset baseline oluşturulurken kullanılmıyor.

Ayrıca self-test sırasında fault etiketli kayıtların baseline
hesaplamasına dahil edilmesi durumunda min/max değerlerinin değişmediği
kontrol edildi.

Bu sayede Day 15'te yapılacak değerlendirmede baseline'ın gerçekten
normal dataset'ten öğrenilmiş sabit bir referans olarak kalması
sağlandı.

---

## 10. Deterministic Baseline

Baseline oluşturma işlemini deterministik olacak şekilde tuttum.

Aynı normal dataset tekrar kullanıldığında aynı:

- min
- max
- hysteresis

değerlerinin oluşturulması gerekiyor.

Self-test içerisinde determinism kontrolü yapıldı ve başarıyla
tamamlandı.

---

## 11. Day 14 Self-Test

Day 14 için özel bir self-test oluşturdum.

Çalıştırdığım komut:

npm run test:day14

Test sonucu:

Day 14 Threshold Baseline Self-Test
-----------------------------------
Baseline schema: PASS
Normal dataset: PASS

temp_core baseline: PASS
temp_ambient baseline: PASS
voltage_in baseline: PASS
current_draw baseline: PASS
fan_rpm baseline: PASS
cpu_load baseline: PASS

Min/max validation: PASS
Hysteresis: PASS
Normal classification: PASS
Upper threshold alarm: PASS
Lower threshold alarm: PASS
Hysteresis recovery: PASS
Determinism: PASS
Leakage protection: PASS

F1 evaluation: PASS
F2 evaluation: PASS
F3 evaluation: PASS
F4 evaluation: PASS
F5 evaluation: PASS

All Day 14 checks passed.

Bütün Day 14 self-test kontrolleri başarıyla tamamlandı.

---

## 12. Validation

Day 14 sonunda aşağıdaki kontrolleri çalıştırdım:

npx tsc --noEmit
npm run lint
npm run build
npm run test:day08
npm run test:day09
npm run test:day10
npm run test:day11
npm run test:day12
npm run test:day13
npm run test:day14

Sonuçların tamamı başarılı oldu:

TypeScript Type Check: PASS
ESLint: PASS
Production Build: PASS
Day 08 Self-Test: PASS
Day 09 Self-Test: PASS
Day 10 Self-Test: PASS
Day 11 Self-Test: PASS
Day 12 Self-Test: PASS
Day 13 Self-Test: PASS
Day 14 Self-Test: PASS

Bu gün browser üzerinde manuel test gerçekleştirmedim. Bu nedenle
browser testi için PASS sonucu vermiyorum.

---

## 13. Eklenen Dosyalar

Day 14 kapsamında aşağıdaki dosyaları ekledim:

src/lib/threshold-detector.ts
src/lib/threshold/day14SelfTest.ts
scripts/run-day14-self-test.ts
Gun_14/README.md

Ayrıca package.json içerisine:

test:day14

komutunu ekledim.

---

## 14. Değiştirilen Dosyalar

Day 14 kapsamında aşağıdaki dosyalarda değişiklik yaptım:

package.json
src/app/page.tsx
src/components/dashboard/LiveTelemetryPanel.tsx
README.md

Day 08-13 içerisinde oluşturulan temel telemetry davranışlarını
değiştirmedim.

Datasetleri de yeniden oluşturmadım.

---

## 15. Dashboard Tarafındaki Değişiklik

Dashboard'a çok küçük bir Normal / Alarm bilgisi ekledim.

Bu bölüm sadece threshold dışına çıkan değerleri kullanıcıya göstermek
için kullanılıyor.

Asıl hysteresis destekli detector ise ayrı bir evaluation API olarak
çalışıyor.

Bu nedenle Dashboard tarafında gereksiz bir alarm sistemi
oluşturmadım ve mevcut telemetry mimarisini değiştirmedim.

---

## 16. Sınırlamalar

Sabit min/max baseline oldukça basit ve açıklanabilir bir yöntem.

Ancak bazı fault türleri normal çalışma aralığının içerisinde
kalabilir.

Örneğin:

- yavaş sensor drift
- threshold'u aşmayan current değişimleri
- threshold'u aşmayan fan degradation

gibi durumlar bu yöntem tarafından her zaman yakalanamayabilir.

Bu bir implementasyon hatası olarak değerlendirilmemeli. Tam tersine,
bu yöntemin sınırlarını göstermek Day 15'te yapılacak değerlendirme
açısından önemli.

Day 15'te bu baseline'ın fault dataset üzerindeki gerçek performansını
ölçmeyi planlıyorum.

---

## 17. Sonuç

Bugünkü çalışma sonunda SpikeEdge Telemetry projesinde normal telemetry
dataset'ine dayalı sabit bir threshold baseline oluşturdum.

Her telemetry kanalı için normal dataset'ten minimum ve maksimum değerler
hesaplandı ve bu değerler üzerinden alarm kontrolü oluşturuldu.

Threshold çevresindeki gereksiz alarm değişimlerini azaltmak için %5
hysteresis mekanizması eklendi.

Ayrıca baseline'ın yalnızca normal dataset'ten oluşturulduğunu ve fault
verilerinin threshold hesaplamasına karışmadığını self-test ile
doğruladım.

Day 14 self-test, TypeScript, ESLint, production build ve önceki
günlerin testleri başarılı şekilde tamamlandı.

Bu baseline, Day 15'te fault dataset üzerinde değerlendirilecek sabit
ve karşılaştırılabilir bir referans olarak kullanılacak.

---

## 18. Day 15'e Hazırlık

Bir sonraki aşamada oluşturduğum bu frozen baseline'ı fault dataset
üzerinde test edeceğim.

Planlanan yapı:

Frozen Baseline
      ↓
Fault Dataset
      ↓
F1 ─┐
F2 ─┤
F3 ─┤
F4 ─┤
F5 ─┘
      ↓
Detection Rate
False Alarms
Per-Fault Results
      ↓
Evaluation Report

Amaç, sabit threshold yönteminin hangi fault'ları yakalayabildiğini ve
hangi durumlarda yetersiz kaldığını objektif şekilde görmek.

ML, Autoencoder ve fault classification bu aşamanın dışında kalıyor.

Gün 14 tamamlandı. ✅
