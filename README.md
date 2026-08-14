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
├── 📁 data/
│   ├── 📊 day09-normal.csv
│   └── 📊 day09-faults.csv
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

## 🧪 Validation

Projenin farklı aşamalarında aşağıdaki validation kontrolleri kullanılmaktadır:

```bash
npx tsc --noEmit
npm run lint
npm run build
npm run test:day08
npm run test:day09
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
Anomaly Detection
```

şeklinde daha kapsamlı bir telemetry analysis altyapısına dönüştürülmesi hedeflenmektedir.

Day 09 itibarıyla dataset generation ve labeling altyapısı oluşturulmuş durumdadır. Bu nedenle bir sonraki aşamada oluşturulan labeled telemetry datasetleri anomaly detection çalışmalarında kullanılabilecek durumdadır.

---

## 🚀 Sonraki Aşama

Bir sonraki aşamada Day 09 kapsamında oluşturulan labeled telemetry datasetleri kullanılarak anomaly detection yaklaşımının geliştirilmesi planlanmaktadır.

Amaç, normal telemetry davranışları ile fault içeren telemetry davranışları arasındaki farklılıkların analiz edilmesi ve sistem tarafından otomatik olarak tespit edilebilmesidir.

İlerleyen aşamalarda farklı fault tipleri için telemetry davranışlarının karşılaştırılması, uygun feature'ların belirlenmesi ve anomaly detection modelinin oluşturulması hedeflenmektedir.

---

## 📌 Güncel Durum

**Staj Günleri:** 9 / 9 tamamlandı ✅

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
```

**Bir sonraki hedef:**

```text
Labeled Telemetry Dataset
        ↓
Feature Analysis
        ↓
Anomaly Detection
        ↓
Fault Classification
```
