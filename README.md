# 📘 SpikeEdge Telemetry - Staj Takibi

Bu depo, SpikeEdge Telemetry projesi kapsamında gerçekleştirilen günlük staj çalışmalarını düzenli olarak belgelemek amacıyla oluşturulmuştur.

Proje boyunca telemetry simulator, dashboard, workload profilleri, deterministic simulation, noise pipeline, fault injection, ground-truth ve labeled dataset altyapısı gibi bileşenler geliştirilmekte ve test edilmektedir.

---

## 📅 Günlük Çalışma Raporları

| 📅 Gün | 📝 Konu | 📌 Durum |
| :---: | :--- | :---: |
| **Gün 01** | Proje Analizi ve Geliştirme Ortamının Hazırlanması | ✅ Tamamlandı |
| **Gün 02** | Proje Altyapısı ve Dashboard Mimarisi | ✅ Tamamlandı |
| **Gün 03** | Telemetry Veri Modeli ve Simulator Altyapısı | ✅ Tamamlandı |
| **Gün 04** | Plant Model Geliştirilmesi ve Sistem Testleri | ✅ Tamamlandı |
| **Gün 05** | Telemetri Simulatorünün Geliştirilmesi ve Canlı Dashboard Entegrasyonu | ✅ Tamamlandı |
| **Gün 06** | Canlı Telemetri Grafikleri ve Telemetri Veri Akışının Geliştirilmesi | ✅ Tamamlandı |
| **Gün 07** | Telemetri Yük Profilleri, Deterministik Gürültü ve Simülasyon Testleri | ✅ Tamamlandı |
| **Gün 08** | Fault Injection, Ground Truth ve Arıza Senaryolarının Simülasyonu | ✅ Tamamlandı |
| **Gün 09** | Telemetri Dataset Oluşturma, Etiketleme ve CSV Export | ✅ Tamamlandı |
| **Gün 10** | Veri Seti Doğrulama, Dağılım Analizi ve Korelasyon Matrisi | ✅ Tamamlandı |

---

## 📂 Rapor Yapısı

Her klasör ilgili günün ayrıntılı çalışma raporunu ve o güne ait gerekli çıktıları içermektedir.

```text
staj-takip/
│
├── 📁 Gun_01/
│   └── 📄 README.md
│
├── 📁 Gun_02/
│   └── 📄 README.md
│
├── 📁 Gun_03/
│   └── 📄 README.md
│
├── 📁 Gun_04/
│   └── 📄 README.md
│
├── 📁 Gun_05/
│   └── 📄 README.md
│
├── 📁 Gun_06/
│   └── 📄 README.md
│
├── 📁 Gun_07/
│   ├── 📄 README.md
│   ├── 🖼️ day07-idle.png
│   ├── 🖼️ day07-variable.png
│   └── 🖼️ day07-sustained.png
│
├── 📁 Gun_08/
│   ├── 📄 README.md
│   ├── 🖼️ day08-normal-baseline.png
│   ├── 🖼️ day08-f1-temperature-spike.png
│   ├── 🖼️ day08-f2-voltage-sag.png
│   ├── 🖼️ day08-f3-current-surge.png
│   ├── 🖼️ day08-f4-fan-degradation.png
│   └── 🖼️ day08-f5-sensor-drift.png
│
├── 📁 Gun_09/
│   └── 📄 README.md
│
├── 📁 Gun_10/
│   └── 📄 README.md
│
├── 📁 data/
│   ├── 📊 day09-normal.csv
│   └── 📊 day09-faults.csv
│
├── 📁 docs/
│   ├── 📄 dataset-report.md
│   └── 📁 dataset/
│       ├── 📊 correlation-matrix.svg
│       ├── 📊 distribution-power.svg
│       ├── 📊 distribution-system.svg
│       └── 📊 distribution-temperature.svg
│
└── 📄 README.md
```

---

## 📊 Proje Gelişim Süreci

### 🔹 Gün 01 — Proje Analizi

İlk gün SpikeEdge Telemetry projesinin genel yapısı incelendi ve geliştirme ortamı hazırlandı.

Projenin amacı, embedded sistemlerden alınabilecek telemetry verilerinin tarayıcı üzerinde gerçek zamanlı olarak izlenebileceği bir monitoring paneli oluşturmaktır.

---

### 🔹 Gün 02 — Dashboard Mimarisi

Dashboard'ın temel frontend mimarisi oluşturuldu.

Telemetry değerlerinin kullanıcıya kartlar ve monitoring bileşenleri üzerinden gösterilebilmesi için temel dashboard yapısı geliştirildi.

---

### 🔹 Gün 03 — Telemetry Veri Modeli ve Simulator

Telemetry veri modeli ve simulator altyapısının temel yapısı oluşturuldu.

Simulator üzerinden sıcaklık, voltaj, akım, fan RPM ve CPU Load gibi telemetry kanallarının üretilmesi sağlandı.

---

### 🔹 Gün 04 — Plant Model

Telemetry kanalları arasındaki fiziksel ilişkileri temsil etmek amacıyla Plant Model geliştirildi.

CPU Load değişiminin sıcaklık, fan RPM, current ve voltage gibi diğer telemetry kanallarına etkisinin modellenmesi sağlandı.

---

### 🔹 Gün 05 — Telemetry Simulator

Simulator daha gelişmiş hale getirildi ve canlı dashboard ile entegre edildi.

Telemetry değerlerinin belirli sample rate ile sürekli olarak üretilmesi ve dashboard üzerinde görüntülenmesi sağlandı.

---

### 🔹 Gün 06 — Live Telemetry Charts

Canlı telemetry veri akışı ve SVG tabanlı grafikler geliştirildi.

Temperature, System ve Power bölümleri üzerinden telemetry değerlerinin zaman içerisindeki değişimleri görselleştirildi.

Dashboard üzerinde aşağıdaki kanalların canlı olarak izlenmesi sağlandı:

- Core Temperature
- Ambient Temperature
- Voltage
- Current
- Fan RPM
- CPU Load

---

### 🔹 Gün 07 — Workload Profiles ve Deterministic Simulation

Yedinci günde simulator içerisine farklı sistem çalışma koşullarını temsil eden workload profilleri eklendi.

Üç farklı profil oluşturuldu:

- `idle`
- `variable`
- `sustained`

Ayrıca seeded random mekanizması ve Noise Pipeline geliştirildi.

Kullanılan temel seed değeri:

```text
1337
```

Noise Pipeline içerisinde drift, jitter, outlier kontrolü ve quantization gibi işlemler kullanıldı.

Bu yapı sayesinde simulator verilerinin daha gerçekçi ve tekrar üretilebilir olması sağlandı.

Day 07 sonunda farklı workload profilleri dashboard üzerinde test edildi ve TypeScript, ESLint ve production build kontrolleri başarıyla tamamlandı.

---

### 🔹 Gün 08 — Fault Injection ve Ground Truth

Sekizinci günde mevcut telemetry simulator altyapısı kontrollü arıza senaryolarını destekleyecek şekilde geliştirildi.

Simulator pipeline içerisine bağımsız bir `FaultEngine` katmanı eklendi.

Pipeline şu şekilde yapılandırıldı:

```text
PlantModel
    ↓
FaultEngine
    ↓
NoisePipeline
    ↓
Bounds / Clamp
    ↓
TelemetryFrame
```

Beş farklı fault tipi oluşturuldu:

| ID | Fault Type | Etkilenen Davranış |
| :---: | :--- | :--- |
| **F1** | `temperature_spike` | Core temperature artışı |
| **F2** | `voltage_sag` | Voltage düşüşü |
| **F3** | `current_surge` | Current artışı |
| **F4** | `fan_degradation` | Fan RPM düşüşü |
| **F5** | `sensor_drift` | Sensör değerinde kademeli drift |

Fault senaryolarının hangi frame aralığında ve hangi zaman içerisinde aktif olduğunu takip etmek amacıyla `GroundTruthEvent` yapısı oluşturuldu.

Bu yapı sayesinde normal telemetry ile fault içeren telemetry verilerinin daha sonraki dataset ve anomaly detection çalışmalarında birbirinden ayrılması mümkün hale getirildi.

Dashboard üzerinde fault durumunun takip edilebilmesi için aşağıdaki bilgiler gösterilmektedir:

```text
profile: variable
seed: 1337
noise: on
faults: on
active: F2-voltage-sag
```

Day 08 kapsamında ayrıca otomatik self-test sistemi oluşturuldu.

Self-test aşağıdaki komut ile çalıştırılabilmektedir:

```bash
npm run test:day08
```

Toplam 10 kontrol gerçekleştirildi ve tüm kontroller başarıyla tamamlandı.

---

### 🔹 Gün 09 — Telemetry Dataset Oluşturma, Etiketleme ve CSV Export

Dokuzuncu günde daha önce geliştirilen telemetry simulator altyapısı kullanılarak etiketlenmiş telemetry datasetlerinin oluşturulması üzerine çalışıldı.

Bu aşamada canlı telemetry pipeline değiştirilmeden mevcut `TelemetryFrame` verilerini bounded şekilde toplayan bir `DatasetRecorder` yapısı oluşturuldu.

Dataset içerisindeki etiketler Day 08 kapsamında oluşturulan `GroundTruthEvent` yapısından alınmaktadır. Fault bulunmayan frame'ler `normal` olarak değerlendirilirken, fault içeren frame'ler ilgili fault ID bilgisi ile `fault` olarak etiketlenmektedir.

Dataset üretiminde kullanılan simulator ayarları:

```text
Seed        : 1337
Sample Rate : 10 Hz
Workload    : variable
```

CSV export işlemi için sabit bir header yapısı kullanıldı ve iki farklı dataset oluşturuldu:

```text
data/day09-normal.csv
data/day09-faults.csv
```

Normal dataset içerisinde:

```text
800 normal samples
```

oluşturuldu.

Fault dataset içerisinde ise:

```text
800 mixed samples
300 fault-labeled samples
```

oluşturuldu. Fault etiketleri Day 08 kapsamında tanımlanan F1 ile F5 arasındaki fault senaryolarından oluşmaktadır.

Dataset üretimi dashboard üzerinden gerçekleştirilmemektedir. CSV dosyalarının oluşturulması için mevcut `Simulator` yapısını kullanan ayrı bir Node scripti kullanılmaktadır.

Day 09 kapsamında anomaly detection veya machine learning modeli geliştirilmemiştir. Oluşturulan datasetler ilerleyen aşamalarda gerçekleştirilecek anomaly detection ve machine learning çalışmalarının labeled input verisi olarak hazırlanmıştır.

Day 09 için ayrıca otomatik self-test sistemi kullanıldı.

Self-test aşağıdaki komut ile çalıştırılabilmektedir:

```bash
npm run test:day09
npm run test:day10
```

Test sonucu:

```text
Day 09 Dataset Self-Test
------------------------
Normal rows: 800
Fault rows: 300
CSV export: PASS
Labels: PASS
Sequence order: PASS
Determinism: PASS

All checks passed.
```

Bu test sonucuyla CSV export, label doğruluğu, sequence order ve deterministic dataset üretimi başarıyla doğrulanmıştır.


---

### 🔹 Gün 10 — Veri Seti Doğrulama, Dağılım Analizi ve Korelasyon Matrisi

Onuncu günde Day 09 kapsamında oluşturulan normal ve fault içeren telemetry datasetleri doğrulandı ve analiz edildi.

Bu aşamada simulator, FaultEngine, dashboard ve Day 09 dataset üretim pipeline'ı değiştirilmedi. Mevcut CSV datasetlerinin üzerine bağımsız bir doğrulama ve analiz katmanı oluşturuldu.

Day 10 kapsamında aşağıdaki dataset bileşenleri geliştirildi:

- `DatasetCsv`
- `DatasetStats`
- `Correlation`
- `DatasetQuality`
- `DatasetLeakage`
- `FaultValidation`
- `DatasetAnalysis`
- `DatasetReport`
- `day10SelfTest`

Ayrıca aşağıdaki scriptler eklendi:

```text
scripts/run-day10-self-test.ts
scripts/generate-day10-report.ts
```

Dataset analiz raporu aşağıdaki dosyada oluşturuldu:

```text
docs/dataset-report.md
```

Dağılım ve korelasyon görselleri ise aşağıdaki klasörde oluşturuldu:

```text
docs/dataset/
```

#### 📊 Normal Dataset İstatistikleri

Normal eğitim datasetinde toplam 800 sample analiz edildi.

| Kanal | Min | Max | Ortalama | Std. Sapma |
| :--- | ---: | ---: | ---: | ---: |
| `temp_core` | 23.45 | 46.90 | 41.89 | 5.80 |
| `cpu_load` | 26.2 | 63.3 | 46.41 | 14.16 |
| `voltage_in` | 12.07 | 12.27 | 12.16 | 0.06 |
| `current_draw` | 0.80 | 1.31 | 1.07 | 0.17 |
| `fan_rpm` | 882 | 2516 | 1987 | 546 |

#### 🔗 Korelasyon Analizi

Normal dataset üzerinde Pearson correlation analizi gerçekleştirildi.

Önemli ilişkiler:

```text
cpu_load      ↔ current_draw    r =  0.9961
current_draw  ↔ voltage_in      r = -0.9853
temp_core     ↔ fan_rpm         r =  0.9593
cpu_load      ↔ temp_core       r = -0.38
```

`cpu_load` ile `current_draw` arasında güçlü pozitif, `current_draw` ile `voltage_in` arasında güçlü negatif ve `temp_core` ile `fan_rpm` arasında güçlü pozitif korelasyon gözlemlendi.

`cpu_load` ile `temp_core` arasındaki anlık korelasyonun beklenen pozitif ilişkiden farklı olduğu görüldü. Bu durum termal gecikme ile açıklanabilir ve analizde abartılı bir sonuç çıkarılmadı.

#### 🧪 Fault Validation

Day 08'de oluşturulan beş fault tipi dataset üzerinde doğrulandı:

| Fault | Doğrulanan Örnek |
| :---: | ---: |
| F1 — `temperature_spike` | 50 |
| F2 — `voltage_sag` | 40 |
| F3 — `current_surge` | 50 |
| F4 — `fan_degradation` | 60 |
| F5 — `sensor_drift` | 100 |

Tüm fault tipleri başarıyla doğrulandı.

Eğitim datasetinde fault etiketi bulunmadığı doğrulandı.

#### 🔐 Data Leakage Kontrolü

Dataset validation aşamasında veri sızıntısı kontrolü gerçekleştirildi.

Sonuçlar:

```text
Training fault labels : 0
Data leakage           : 0
```

F4 ve F5 rampalarının başlangıcında intensity değerinin yaklaşık 0 olması nedeniyle aynı sequence içerisinde 10 frame sayısal olarak örtüşmektedir. Bu durum etiket sızıntısı olarak değerlendirilmemiş ve raporda belgelenmiştir.

Bu kontrol özellikle sonraki anomaly detection aşaması için kritik olarak ele alınmıştır. Eğitim datasetinin yalnızca normal davranış içermesi korunmuştur.

#### 🔁 Deterministic Validation

Dataset üretiminin tekrar üretilebilirliği de kontrol edildi.

Aynı simulator seed ve aynı parametreler kullanıldığında aynı dataset değerlerinin elde edildiği doğrulandı.

Kullanılan temel seed:

```text
1337
```

#### 🧪 Day 10 Self-Test

Day 10 için otomatik validation sistemi oluşturuldu.

Çalıştırma komutu:

```bash
npm run test:day10
```

Test sonucu:

```text
Day 10 Dataset Validation
-------------------------

Dataset schema: PASS
Normal dataset: PASS
Fault dataset: PASS

F1 temperature_spike: PASS
F2 voltage_sag: PASS
F3 current_surge: PASS
F4 fan_degradation: PASS
F5 sensor_drift: PASS

Statistics: PASS
Distribution analysis: PASS
Correlation matrix: PASS
Correlation symmetry: PASS
Leakage check: PASS
Determinism: PASS

All Day 10 checks passed.
```

#### ✅ Validation Sonuçları

| Kontrol | Sonuç |
| :--- | :---: |
| TypeScript Type Check | ✅ Passed |
| ESLint | ✅ Passed |
| Production Build | ✅ Passed |
| Day 08 Self-Test | ✅ Passed |
| Day 09 Self-Test | ✅ Passed |
| Day 10 Self-Test | ✅ Passed |
| Dataset Schema | ✅ Passed |
| Normal Dataset | ✅ Passed |
| Fault Dataset | ✅ Passed |
| F1–F5 Validation | ✅ Passed |
| Distribution Analysis | ✅ Passed |
| Correlation Matrix | ✅ Passed |
| Leakage Check | ✅ Passed |
| Determinism | ✅ Passed |

Day 10 sonunda telemetry datasetlerinin yapısı, dağılımları، kanallar arası ilişkileri, fault etiketleri, veri sızıntısı ve deterministic üretim özellikleri başarıyla doğrulanmıştır.


---

## 🧪 Validation

Projenin farklı aşamalarında aşağıdaki validation kontrolleri kullanılmaktadır:

```bash
npx tsc --noEmit
npm run lint
npm run build
npm run test:day08
npm run test:day09
npm run test:day10
```

Day 08 ve Day 09 sonunda gerçekleştirilen kontroller:

| Kontrol | Sonuç |
| :--- | :---: |
| TypeScript Type Check | ✅ Passed |
| ESLint | ✅ Passed |
| Production Build | ✅ Passed |
| Day 08 Self-Test | ✅ Passed |
| Day 08 Self-Test Checks | ✅ 10 / 10 |
| Day 09 Self-Test | ✅ Passed |
| Day 09 CSV Export | ✅ Passed |
| Day 09 Labels | ✅ Passed |
| Day 09 Sequence Order | ✅ Passed |
| Day 09 Determinism | ✅ Passed |
| Day 10 Self-Test | ✅ Passed |
| Day 10 Dataset Validation | ✅ Passed |
| Day 10 Distribution Analysis | ✅ Passed |
| Day 10 Correlation Matrix | ✅ Passed |
| Day 10 Leakage Check | ✅ Passed |
| Day 10 Determinism | ✅ Passed |

---

## 🧩 Kullanılan Teknolojiler

Projenin frontend, simulator ve dataset generation tarafında aşağıdaki teknolojiler kullanılmaktadır:

- Next.js
- React
- TypeScript
- Tailwind CSS
- SVG
- Node.js
- Git
- GitHub
- CSV
- Dataset Analysis
- Pearson Correlation

---

## 🎯 Proje Hedefi

SpikeEdge Telemetry projesinin temel amacı, embedded sistemlerde bulunabilecek telemetry verilerinin gerçekçi bir şekilde simüle edilmesi ve bu verilerin monitoring paneli üzerinden izlenebilmesini sağlamaktır.

Proje ilerledikçe sistemin yalnızca normal telemetry üretmesi yerine;

```text
Normal Telemetry
       ↓
Workload Simulation
       ↓
Physical Plant Model
       ↓
Fault Injection
       ↓
Noise
       ↓
Ground Truth
       ↓
Dataset Recording
       ↓
CSV Export
       ↓
Dataset Validation
       ↓
Distribution Analysis
       ↓
Correlation Analysis
       ↓
Anomaly Detection
```

şeklinde daha kapsamlı bir telemetry analysis altyapısına dönüştürülmesi hedeflenmektedir.

Day 09 itibarıyla dataset generation ve labeling altyapısı oluşturulmuş durumdadır. Bu nedenle bir sonraki aşamada oluşturulan labeled telemetry datasetleri anomaly detection çalışmalarında kullanılabilecek durumdadır.

---

## 🚀 Sonraki Aşama

Day 10 sonunda labeled telemetry datasetlerinin kalite ve bütünlük kontrolleri tamamlanmıştır.

Bir sonraki aşamada bu doğrulanmış datasetler kullanılarak feature analysis ve anomaly detection çalışmalarına geçilmesi planlanmaktadır.

Amaç, normal telemetry davranışının öğrenilmesi ve fault içeren davranışların bu normal modelden sapma olarak tespit edilebilmesidir.

Özellikle eğitim datasetinin yalnızca normal veri içermesi ve test tarafındaki fault örneklerinin eğitim sürecine sızmaması korunacaktır.

---

## 📌 Güncel Durum

**Staj Günleri:** 10 / 10 tamamlandı ✅

**Mevcut Durum:**

```text
Telemetry Simulator        ✅
Plant Model                ✅
Workload Profiles          ✅
Deterministic Random       ✅
Noise Pipeline             ✅
Live Dashboard             ✅
Live Telemetry Charts      ✅
Fault Injection            ✅
FaultEngine                ✅
Ground Truth               ✅
Fault Self-Test            ✅
Deterministic Fault Sim.   ✅
Fault-aware Telemetry      ✅
Dataset Recorder           ✅
Ground Truth Labeling      ✅
CSV Dataset Export         ✅
Day 09 Self-Test           ✅
Normal Dataset             ✅
Fault Dataset              ✅
Dataset Validation          ✅
Distribution Analysis       ✅
Correlation Matrix          ✅
Leakage Check               ✅
Deterministic Validation    ✅
```

**Bir sonraki hedef:**

```text
Labeled Telemetry Dataset
        ↓
Dataset Validation
        ↓
Feature Analysis
        ↓
Anomaly Detection
        ↓
Fault Classification
```
