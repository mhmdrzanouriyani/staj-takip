<div align="center">

# 📊 Gün 14 — Günlük Çalışma Raporu

**Sabit Eşik Baseline ve Alarm Mekanizması**

`16 Ağustos 2026` · SpikeEdge Telemetry · Staj Günü 14 / 14

<br />

![Durum](https://img.shields.io/badge/Durum-Tamamlandı-22c55e?style=for-the-badge)
![Baseline](https://img.shields.io/badge/Baseline-Fixed%20Min%2FMax-3b82f6?style=for-the-badge)
![Hysteresis](https://img.shields.io/badge/Hysteresis-5%25-0ea5e9?style=for-the-badge)
![Self--Test](https://img.shields.io/badge/test%3Aday14-PASS-16a34a?style=for-the-badge)

</div>

---

<details>
<summary><strong>İçindekiler</strong></summary>

1. [Günün özeti](#1-günün-özeti)
2. [Günün temel hedefi](#2-günün-temel-hedefi)
3. [Day 14'ün projedeki yeri](#3-day-14ün-projedeki-yeri)
4. [Baseline yaklaşımı](#4-baseline-yaklaşımı)
5. [Normal dataset kullanımı](#5-normal-dataset-kullanımı)
6. [Kanal bazlı min/max](#6-kanal-bazlı-minmax)
7. [Hysteresis mekanizması](#7-hysteresis-mekanizması)
8. [Threshold Detector](#8-threshold-detector)
9. [Alarm sonuçları](#9-alarm-sonuçları)
10. [F1–F5 değerlendirmesi](#10-f1–f5-değerlendirmesi)
11. [Veri sızıntısının önlenmesi](#11-veri-sızıntısının-önlenmesi)
12. [Dashboard değişikliği](#12-dashboard-değişikliği)
13. [Eklenen dosyalar](#13-eklenen-dosyalar)
14. [Değiştirilen dosyalar](#14-değiştirilen-dosyalar)
15. [Self-test kapsamı](#15-self-test-kapsamı)
16. [Day 14 self-test sonucu](#16-day-14-self-test-sonucu)
17. [Validation](#17-validation)
18. [Sınırlamalar](#18-sınırlamalar)
19. [Sonuç](#19-sonuç)
20. [Bir sonraki aşama](#20-bir-sonraki-aşama)
21. [Güncel durum](#21-güncel-durum)

</details>

---

## 1. Günün özeti

Stajın on dördüncü gününde SpikeEdge Telemetry projesinde, daha önce
oluşturulan normal telemetry dataset'ini kullanarak **sabit eşik tabanlı
bir baseline detector** geliştirdim.

Bu çalışmadaki temel amacım, makine öğrenmesi veya yapay zeka kullanmadan
telemetry değerlerinin normal çalışma aralığında olup olmadığını kontrol
edebilen basit, deterministik ve açıklanabilir bir referans sistem
oluşturmaktı.

Baseline yalnızca normal dataset üzerinden oluşturuldu. Fault verileri
baseline hesabına dahil edilmedi ve sadece değerlendirme amacıyla
kullanıldı.

Day 14 sonunda sistem artık her telemetry kanalı için normal çalışma
aralığını temsil eden minimum ve maksimum değerleri kullanarak threshold
kontrolü yapabiliyor.

Ayrıca threshold çevresindeki küçük dalgalanmaların sürekli alarm
oluşturmasını önlemek için hysteresis mekanizması eklendi.

---

## 2. Günün temel hedefi

Day 14 kapsamında aşağıdaki çalışmaları gerçekleştirdim:

- Normal dataset'ten sabit baseline oluşturmak
- Her telemetry kanalı için minimum ve maksimum değerleri belirlemek
- Kanal bazlı threshold kontrolü yapmak
- Hysteresis mekanizması eklemek
- Threshold dışına çıkan değerler için alarm üretmek
- Alarm durumunun tekrar normale dönmesini kontrollü şekilde yönetmek
- Fault verilerini baseline eğitiminden tamamen ayırmak
- F1–F5 senaryolarını detector üzerinden değerlendirebilmek
- Deterministik ve tekrar üretilebilir bir baseline oluşturmak
- Day 15'te yapılacak performans değerlendirmesine temel hazırlamak

Day 14'te herhangi bir ML modeli, Autoencoder veya fault classification
mekanizması geliştirilmedi.

---

## 3. Day 14'ün projedeki yeri

Day 14, önceki günlerde oluşturulan telemetry ve dataset altyapısının
üzerine bir **referans anomaly baseline** ekliyor.

```text
Day 09
Normal + Fault Dataset
        ↓
Day 10
Dataset Validation
        ↓
Day 14
Normal Dataset
        ↓
Fixed Min/Max Baseline
        ↓
Hysteresis
        ↓
Threshold Detector
        ↓
Alarm / Normal
        ↓
Day 15
Performance Evaluation
```

Buradaki önemli nokta, Day 14 detector'ın fault türünü tanımamasıdır.

Örneğin sıcaklık değeri threshold'u aşarsa detector:

```text
F1
```

sonucunu üretmez.

Bunun yerine:

```text
ALARM
reason: above_max
```

gibi açıklanabilir bir sonuç üretir.

Fault tiplerinin performans değerlendirmesi sonraki aşamada yapılacaktır.

---

## 4. Baseline yaklaşımı

Day 14'te kullanılan baseline yöntemi oldukça basit ve açıklanabilir
tutuldu.

Temel mantık:

```text
Normal Dataset
      ↓
Normal Rows
      ↓
Channel Statistics
      ↓
Min / Max
      ↓
Fixed Baseline
      ↓
Incoming Telemetry
      ↓
Threshold Check
```

Normal dataset içerisinde gözlemlenen minimum ve maksimum değerler,
ilgili kanalın normal çalışma aralığını temsil ediyor.

Bu nedenle baseline dışındaki bir değer alarm olarak değerlendiriliyor.

Bu yaklaşımın avantajı, sonucu açıklamanın çok kolay olmasıdır:

```text
Değer > Normal Maximum
        ↓
Alarm
```

veya:

```text
Değer < Normal Minimum
        ↓
Alarm
```

---

## 5. Normal dataset kullanımı

Baseline oluşturmak için:

```text
data/day09-normal.csv
```

dosyası kullanıldı.

Baseline hesaplanırken yalnızca:

```text
label = "normal"
```

olan satırlar dikkate alındı.

Fault dataset baseline oluşturma sürecine dahil edilmedi.

Doğru veri akışı:

```text
day09-normal.csv
        ↓
Baseline
        ↓
Frozen Thresholds
        ↓
Fault Dataset
        ↓
Evaluation
```

Bu ayrım özellikle önemlidir çünkü Day 15'te baseline'ın daha önce
görmediği fault verileri üzerinde performansı ölçülecektir.

---

## 6. Kanal bazlı Min/Max

Mevcut telemetry yapısındaki altı kanal için ayrı baseline oluşturuldu:

| Kanal | Açıklama |
|-------|----------|
| `temp_core` | Core sıcaklığı |
| `temp_ambient` | Ortam sıcaklığı |
| `voltage_in` | Giriş voltajı |
| `current_draw` | Akım tüketimi |
| `fan_rpm` | Fan dönüş hızı |
| `cpu_load` | CPU yükü |

Her kanal için:

```text
min
max
hysteresis
```

bilgileri tutuluyor.

Baseline oluşturma sırasında Day 10'da hazırlanmış olan mevcut
`allChannelStatistics` yapısı tekrar kullanıldı.

Böylece yeni bir istatistik hesaplama sistemi oluşturmak yerine mevcut
dataset analiz altyapısı yeniden kullanılmış oldu.

---

## 7. Hysteresis mekanizması

Sadece min/max threshold kullanıldığında değer threshold çevresinde
dalgalanırsa sistem sürekli olarak:

```text
ALARM
NORMAL
ALARM
NORMAL
```

şeklinde değişebilir.

Bunu önlemek için Day 14'te hysteresis mekanizması ekledim.

Kullanılan oran:

```text
DEFAULT_HYSTERESIS_RATIO = 0.05
```

Yani hysteresis, ilgili kanalın normal veri aralığının %5'i olarak
hesaplanıyor.

Üst threshold için:

```text
value > max
    ↓
ALARM
```

Alarm durumundan normale dönebilmek için:

```text
value < max - hysteresis
    ↓
NORMAL
```

Alt threshold için:

```text
value < min
    ↓
ALARM
```

Alarm durumundan normale dönebilmek için:

```text
value > min + hysteresis
    ↓
NORMAL
```

Threshold ile recovery threshold arasında kalan değerlerde mevcut alarm
durumu korunuyor ve sonuç:

```text
recovering
```

olarak açıklanabiliyor.

Hysteresis değeri ayrıca kanalın normal aralığının yarısından fazla
olmayacak şekilde sınırlandırıldı.

---

## 8. Threshold Detector

Ana detector:

```text
src/lib/threshold-detector.ts
```

dosyasında oluşturuldu.

Detector'ın görevi yalnızca telemetry değerini baseline ile
karşılaştırmaktır.

Temel kontrol:

```text
                    max
                     ↑
                     │
       NORMAL        │
                     │
                     ↓
                    min
```

Değer `max` değerini aşarsa:

```text
ALARM
above_max
```

Değer `min` değerinin altına düşerse:

```text
ALARM
below_min
```

Normal aralık içerisindeyse:

```text
NORMAL
within_baseline
```

Hysteresis bölgesinde ise mevcut alarm state'i korunarak:

```text
recovering
```

reason'ı kullanılabilir.

Day 14'te ayrıca `WARNING` state'i eklenmedi.

Detector yalnızca:

```text
NORMAL
ALARM
```

durumlarını kullanıyor.

Bu sayede baseline mümkün olduğunca basit ve açıklanabilir tutuldu.

---

## 9. Alarm sonuçları

Detector her kanal için yapılandırılmış bir sonuç üretebiliyor.

Sonuç içerisinde kanalın durumu ve reason bilgisi bulunuyor.

Kullanılan temel reason değerleri:

```text
within_baseline
above_max
below_min
recovering
```

Örnek:

```text
channel: temp_core
value: 55
state: ALARM
reason: above_max
```

Burada detector yalnızca threshold ihlalini raporluyor.

Herhangi bir fault ID üretmiyor.

Bu özellikle önemli çünkü:

```text
Threshold Detector ≠ Fault Classifier
```

Day 14'ün amacı baseline oluşturmak ve threshold ihlalini ölçmek.

---

## 10. F1–F5 değerlendirmesi

Mevcut FaultEngine'deki beş fault senaryosu detector tarafından
değerlendirilebilir durumda:

| ID | Fault |
|----|-------|
| F1 | `temperature_spike` |
| F2 | `voltage_sag` |
| F3 | `current_surge` |
| F4 | `fan_degradation` |
| F5 | `sensor_drift` |

Self-test içerisinde F1'in `temp_core` maksimum baseline değerini
aşabildiği ve F2'nin `voltage_in` minimum baseline değerinin altına
inebildiği doğrulandı.

F3, F4 ve F5 de evaluation kapsamına alındı ancak bu fault'ların
mutlaka alarm üretmesi şart koşulmadı.

Bunun nedeni sabit min/max baseline'ın normal aralık içerisinde kalan
fault'ları kaçırabilmesidir.

Bu durum bir hata olarak gizlenmedi. Tam tersine, Day 15'te bu baseline
yönteminin gerçek performansını ölçmek için korunması gereken bir
özelliktir.

Threshold değerleri fault dataset kullanılarak ayarlanmadı.

---

## 11. Veri sızıntısının önlenmesi

Day 14'te veri leakage konusu özellikle kontrol edildi.

Baseline yalnızca normal dataset'ten oluşturuluyor:

```text
Normal Dataset
      ↓
Baseline
```

Fault dataset ise yalnızca değerlendirme aşamasında kullanılıyor:

```text
Baseline
   ↓
Fault Dataset
   ↓
Evaluation
```

Self-test sırasında fault-labeled satırların baseline hesaplamasına
karıştırılması durumunda min/max değerlerinin değişmediği de kontrol
edildi.

Böylece baseline'ın fault verilerini önceden görerek avantaj elde
etmesinin önüne geçildi.

---

## 12. Dashboard değişikliği

Day 14 kapsamında Dashboard'a yalnızca küçük bir Normal / Alarm bilgisi
eklendi.

Değiştirilen dosya:

```text
src/components/dashboard/LiveTelemetryPanel.tsx
```

Bu değişiklik mevcut Dashboard mimarisini değiştirmiyor.

Day 13'te oluşturulan:

```text
WebSocket Server
      ↓
TelemetryClient
      ↓
Ring Buffer
      ↓
Dashboard
```

akışı korunmaya devam ediyor.

Threshold detector bu akışın üzerine eklenen ayrı bir analiz katmanı
olarak çalışıyor.

Ayrıca `src/app/page.tsx` tarafında normal dataset'in server tarafında
yüklenmesi sağlandı.

---

## 13. Eklenen dosyalar

Day 14 kapsamında aşağıdaki dosyaları ekledim:

| Dosya | Görev |
|-------|-------|
| `src/lib/threshold-detector.ts` | Fixed min/max threshold detector |
| `src/lib/threshold/day14SelfTest.ts` | Day 14 self-test mantığı |
| `scripts/run-day14-self-test.ts` | Self-test çalıştırma script'i |
| `Gun_14/README.md` | Günlük çalışma raporu |

---

## 14. Değiştirilen dosyalar

Day 14 kapsamında aşağıdaki dosyalarda değişiklik yapıldı:

```text
package.json
src/app/page.tsx
src/components/dashboard/LiveTelemetryPanel.tsx
README.md
```

`package.json` içerisine:

```bash
npm run test:day14
```

komutu eklendi.

Day 08–13 davranışları değiştirilmedi.

Datasetler de yeniden oluşturulmadı.

---

## 15. Self-test kapsamı

Day 14 için özel bir self-test sistemi oluşturuldu.

Çalıştırma komutu:

```bash
npm run test:day14
```

Self-test aşağıdaki kontrolleri gerçekleştiriyor:

| Kontrol | Amaç |
|---------|------|
| Baseline schema | Baseline yapısının doğruluğu |
| Normal dataset | Normal verinin kullanılabilirliği |
| Channel baselines | Tüm kanallar için baseline |
| Min/max validation | Threshold değerlerinin kontrolü |
| Hysteresis | Hysteresis hesaplamasının doğrulanması |
| Normal classification | Normal değerlerin doğru sınıflandırılması |
| Upper threshold alarm | Üst threshold alarmı |
| Lower threshold alarm | Alt threshold alarmı |
| Hysteresis recovery | Alarmdan recovery davranışı |
| Determinism | Aynı input ile aynı baseline |
| Leakage protection | Fault verisinin baseline'a karışmaması |
| F1 evaluation | Temperature spike değerlendirmesi |
| F2 evaluation | Voltage sag değerlendirmesi |
| F3 evaluation | Current surge değerlendirmesi |
| F4 evaluation | Fan degradation değerlendirmesi |
| F5 evaluation | Sensor drift değerlendirmesi |

---

## 16. Day 14 self-test sonucu

Çalıştırdığım komut:

```bash
npm run test:day14
```

Sonuç:

```text
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
```

Bütün Day 14 self-test kontrolleri başarıyla tamamlandı.

---

## 17. Validation

Day 14 sonunda aşağıdaki kontroller gerçekleştirildi:

| Komut | Sonuç |
|-------|-------|
| `npx tsc --noEmit` | ✅ PASS |
| `npm run lint` | ✅ PASS |
| `npm run build` | ✅ PASS |
| `npm run test:day08` | ✅ PASS |
| `npm run test:day09` | ✅ PASS |
| `npm run test:day10` | ✅ PASS |
| `npm run test:day11` | ✅ PASS |
| `npm run test:day12` | ✅ PASS |
| `npm run test:day13` | ✅ PASS |
| `npm run test:day14` | ✅ PASS |

No browser test was run.

Bu nedenle Day 14 için ayrıca bir manuel browser test sonucu
belirtilmedi.

---

## 18. Sınırlamalar

Fixed min/max baseline basit ve açıklanabilir bir yöntem olduğu için
bazı fault türlerini kaçırabilir.

Özellikle değer normal min/max aralığının içerisinde kalıyorsa:

- yavaş sensor drift,
- threshold'u aşmayan current değişimleri,
- threshold'u aşmayan fan degradation

gibi durumlar alarm üretmeyebilir.

Bu durum baseline'ın bir implementasyon hatası değildir.

Tam tersine, sabit threshold yönteminin sınırını göstermektedir.

Day 15'te bu sınırlamaların gerçek fault dataset üzerinde ne kadar
etkili olduğu ölçülecektir.

Ayrıca Dashboard üzerindeki Normal / Alarm bilgisi stateless bir trip
kontrolüdür. Hysteresis destekli ve stateful detector ise evaluation
API olarak kullanılmaktadır.

---

## 19. Sonuç

Day 14 sonunda SpikeEdge Telemetry projesinde normal telemetry
dataset'inden öğrenilen sabit bir min/max baseline detector oluşturuldu.

Her kanal için normal dataset'ten minimum ve maksimum değerler
hesaplandı.

Threshold çevresindeki küçük dalgalanmaların sürekli alarm oluşturmasını
önlemek için %5 hysteresis mekanizması eklendi.

Detector artık:

```text
Telemetry Value
      ↓
Min / Max Check
      ↓
Hysteresis
      ↓
NORMAL / ALARM
```

şeklinde çalışıyor.

Ayrıca baseline'ın yalnızca normal dataset üzerinden oluşturulduğu ve
fault verilerinin baseline hesaplamasına dahil edilmediği doğrulandı.

Day 14 self-test, TypeScript, ESLint, production build ve Day 08–13
testlerinin tamamı başarıyla tamamlandı.

Bu çalışma ile daha gelişmiş anomaly detection yöntemlerinden önce
kullanılabilecek basit, deterministic ve açıklanabilir bir referans
baseline hazırlanmış oldu.

---

## 20. Bir sonraki aşama

Day 15'te oluşturulan bu baseline'ın fault dataset üzerinde gerçek
performansını ölçmek planlanıyor.

Beklenen akış:

```text
Frozen Baseline
       ↓
Fault Dataset
       ↓
┌──────┬──────┬──────┬──────┬──────┐
│  F1  │  F2  │  F3  │  F4  │  F5  │
└──────┴──────┴──────┴──────┴──────┘
       ↓
Detection Rate
False Alarms
Per-Fault Results
       ↓
Evaluation Report
```

Amaç, fixed threshold baseline'ın hangi fault'ları yakalayabildiğini ve
hangi durumlarda yetersiz kaldığını objektif olarak ölçmek.

Bu aşamada hâlâ:

```text
❌ Machine Learning
❌ Autoencoder
❌ Neural Network
❌ Fault Classification
❌ Image-to-3D
```

kapsam dışında tutuluyor.

---

## 21. Güncel durum

<div align="center">

**Staj Günleri: 14 / 14 tamamlandı** ✅

</div>

| Bileşen | Durum |
|---------|:-----:|
| Telemetry Simulator | ✅ |
| Plant Model | ✅ |
| Workload Profiles | ✅ |
| Deterministic Random | ✅ |
| Noise Pipeline | ✅ |
| Live Dashboard | ✅ |
| Live Telemetry Charts | ✅ |
| Fault Injection | ✅ |
| FaultEngine | ✅ |
| Ground Truth | ✅ |
| Dataset Recorder | ✅ |
| CSV Dataset Export | ✅ |
| Dataset Validation | ✅ |
| Distribution Analysis | ✅ |
| Correlation Matrix | ✅ |
| Leakage Check | ✅ |
| WebSocket Server | ✅ |
| Real-time Telemetry Stream | ✅ |
| Multi-client Broadcast | ✅ |
| TelemetryClient | ✅ |
| Ring Buffer | ✅ |
| Backpressure | ✅ |
| Automatic Reconnect | ✅ |
| Dashboard WebSocket Integration | ✅ |
| Fixed Threshold Baseline | ✅ |
| Hysteresis | ✅ |
| Alarm Detection | ✅ |
| Day 14 Self-Test | ✅ |

---

<div align="center">

**Gün 14 tamamlandı.** ✅

**Sonraki hedef: Baseline Evaluation — Day 15**

</div>
