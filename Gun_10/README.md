# 📊 Gün 10 - Günlük Çalışma Raporu

**Tarih:** 12 Ağustos 2026
**Konu:** Veri Seti Doğrulama, Dağılım Analizi ve Korelasyon Matrisi

---

## 📝 Günün Özeti

Stajın onuncu gününde SpikeEdge Telemetry projesinde daha önce oluşturulan telemetry datasetlerinin doğrulanması ve analiz edilmesi üzerine çalışıldı.

Day 09 kapsamında oluşturulan normal ve fault içeren CSV veri setleri kullanılarak veri kalitesi, veri dağılımı, fault etiketleri, kanallar arası korelasyon ve veri sızıntısı kontrolleri gerçekleştirildi.

Bu çalışma ile telemetry datasetinin sonraki aşamalarda kullanılacak anomaly detection çalışmaları için uygun ve güvenilir bir yapıda olup olmadığı kontrol edildi.

Day 10 kapsamında canlı telemetry simulator, FaultEngine veya dashboard pipeline değiştirilmedi. Mevcut datasetlerin üzerine bir doğrulama ve analiz katmanı oluşturuldu.

---

## 🛠️ Yapılan Çalışmalar

### 1. Veri Seti Doğrulama

Day 09 kapsamında oluşturulan iki farklı CSV veri seti analiz edildi:

```text
data/day09-normal.csv
data/day09-faults.csv
```

Normal veri seti eğitim amacıyla kullanılacak normal telemetry örneklerini içerirken, fault veri seti farklı arıza senaryolarından oluşturulan etiketli telemetry örneklerini içermektedir.

Dataset schema ve temel veri yapısı kontrol edildi.

---

### 2. Veri Kalitesi Kontrolleri

Dataset içerisinde aşağıdaki kontroller gerçekleştirildi:

- Gerekli kolonların bulunması
- Eksik değerlerin kontrol edilmesi
- Geçersiz numeric değerlerin kontrol edilmesi
- NaN ve Infinity değerlerinin kontrol edilmesi
- Geçersiz fault etiketlerinin kontrol edilmesi
- Sequence sırasının kontrol edilmesi
- Dataset yapısının doğrulanması

Tüm temel dataset kalite kontrolleri başarıyla tamamlandı.

---

### 3. Normal Veri Setinin Kontrolü

Normal dataset içerisindeki örneklerin yalnızca normal telemetry davranışını temsil ettiği doğrulandı.

Eğitim amacıyla kullanılan normal dataset içerisinde fault etiketi bulunmadığı kontrol edildi.

Sonuç:

```text
Normal Dataset: PASS
Fault Label Leakage: 0
```

Bu kontrol, gelecekte oluşturulacak anomaly detection modelinin yalnızca normal sistem davranışını öğrenebilmesi açısından önemlidir.

---

### 4. Fault Veri Setinin Kontrolü

Day 08 kapsamında oluşturulan beş fault tipi Day 09 datasetinde tekrar kontrol edildi.

Kullanılan fault tipleri:

| ID | Fault Type | Sample Count | Durum |
| :---: | :--- | ---: | :---: |
| **F1** | `temperature_spike` | 50 | ✅ PASS |
| **F2** | `voltage_sag` | 40 | ✅ PASS |
| **F3** | `current_surge` | 50 | ✅ PASS |
| **F4** | `fan_degradation` | 60 | ✅ PASS |
| **F5** | `sensor_drift` | 100 | ✅ PASS |

Böylece beş farklı fault senaryosunun dataset içerisinde üretildiği ve etiketlendiği doğrulandı.

---

### 5. İstatistiksel Dağılım Analizi

Normal telemetry datasetindeki temel kanallar için minimum, maksimum, ortalama ve standart sapma değerleri hesaplandı.

| Kanal | Min | Max | Ortalama | Std. Sapma |
| :--- | ---: | ---: | ---: | ---: |
| `temp_core` | 23.45 | 46.90 | 41.89 | 5.80 |
| `cpu_load` | 26.2 | 63.3 | 46.41 | 14.16 |
| `voltage_in` | 12.07 | 12.27 | 12.16 | 0.06 |
| `current_draw` | 0.80 | 1.31 | 1.07 | 0.17 |
| `fan_rpm` | 882 | 2516 | 1987 | 546 |

Bu analiz sayesinde telemetry kanallarının genel dağılımları incelendi ve dataset içerisindeki değerlerin beklenen simülasyon aralıklarında olduğu kontrol edildi.

---

### 6. Korelasyon Matrisi

Telemetry kanalları arasındaki ilişkileri incelemek amacıyla Pearson correlation matrix oluşturuldu.

Analizde aşağıdaki kanallar kullanıldı:

```text
temp_core
temp_ambient
voltage_in
current_draw
fan_rpm
cpu_load
```

Önemli korelasyon sonuçları:

```text
cpu_load ↔ current_draw
r = 0.9961
```

CPU Load ile Current Draw arasında güçlü pozitif korelasyon gözlemlendi.

```text
current_draw ↔ voltage_in
r = -0.9853
```

Current Draw ile Voltage arasında güçlü negatif korelasyon gözlemlendi. Bu sonuç yüksek yük altında oluşan voltage droop davranışıyla uyumludur.

```text
temp_core ↔ fan_rpm
r = 0.9593
```

Core Temperature ile Fan RPM arasında güçlü pozitif korelasyon gözlemlendi.

Ayrıca:

```text
cpu_load ↔ temp_core
r = -0.38
```

CPU Load ile Core Temperature arasındaki korelasyon beklenen pozitif ilişkiden farklı bulundu. Bu sonuç raporda doğrudan pozitif olarak yorumlanmadı. Simülasyondaki termal gecikme ve zaman bağımlı davranış dikkate alınarak değerlendirildi.

Korelasyon değerleri dataset üzerinden dinamik olarak hesaplandı ve hard-code edilmedi.

---

### 7. Fiziksel İlişkilerin Doğrulanması

Correlation analysis sonuçları mevcut PlantModel davranışları ile karşılaştırıldı.

Özellikle aşağıdaki ilişkiler dataset üzerinde gözlemlendi:

```text
CPU Load ↑
    ↓
Current Draw ↑
```

```text
Current Draw ↑
    ↓
Voltage ↓
```

```text
Temperature ↑
    ↓
Fan RPM ↑
```

Bu sonuçlar telemetry simulator içerisinde oluşturulan fiziksel ilişkilerin dataset üzerinde de gözlemlenebildiğini göstermektedir.

---

### 8. Data Leakage Kontrolü

Datasetlerin birbirine karışmaması için data leakage kontrolü gerçekleştirildi.

Kontrol edilen temel kural:

```text
Training
└── Normal Data Only

Test
├── Normal Data
└── Fault Data
```

Normal dataset içerisinde fault etiketi bulunmadığı doğrulandı.

Sonuç:

```text
Training Fault Labels: 0
Data Leakage: 0
```

Ayrıca F4 ve F5 fault rampalarının başlangıcında intensity değerinin yaklaşık sıfır olması nedeniyle bazı frame'lerde sayısal değerlerin normal örneklere benzeyebildiği gözlemlendi.

Toplam 10 frame bu nedenle sayısal olarak örtüşme göstermektedir. Ancak bu durum fault etiketinin training datasetine sızması anlamına gelmemektedir ve label leakage olarak değerlendirilmemiştir.

---

### 9. Deterministic Validation

Day 07 ve Day 09 kapsamında oluşturulan deterministic simulation yapısının korunup korunmadığı kontrol edildi.

Kullanılan temel seed:

```text
1337
```

Aynı simulator ayarları ve aynı seed kullanıldığında dataset ve analiz sonuçlarının tekrar üretilebilir olduğu doğrulandı.

Sonuç:

```text
Determinism: PASS
```

---

## 🧪 Day 10 Self-Test

Day 10 validation aşağıdaki komut ile çalıştırıldı:

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

---

## 🔧 Validation

Day 10 kapsamında aşağıdaki kontroller başarıyla tamamlandı:

| Kontrol | Sonuç |
| :--- | :---: |
| Dataset Schema | ✅ Passed |
| Normal Dataset | ✅ Passed |
| Fault Dataset | ✅ Passed |
| F1 Validation | ✅ Passed |
| F2 Validation | ✅ Passed |
| F3 Validation | ✅ Passed |
| F4 Validation | ✅ Passed |
| F5 Validation | ✅ Passed |
| Statistics | ✅ Passed |
| Distribution Analysis | ✅ Passed |
| Correlation Matrix | ✅ Passed |
| Correlation Symmetry | ✅ Passed |
| Data Leakage Check | ✅ Passed |
| Determinism | ✅ Passed |
| TypeScript | ✅ Passed |
| ESLint | ✅ Passed |
| Production Build | ✅ Passed |
| Day 08 Self-Test | ✅ Passed |
| Day 09 Self-Test | ✅ Passed |
| Day 10 Self-Test | ✅ Passed |

---

## 📊 Demo #2

Day 10 kapsamında oluşturulan dataset validation raporu:

```text
docs/dataset-report.md
```

Ayrıca dataset dağılımı ve correlation analysis sonuçları için SVG tabanlı analiz çıktıları oluşturuldu.

Bu çıktılar datasetin istatistiksel yapısını ve telemetry kanalları arasındaki ilişkileri görsel olarak incelemek amacıyla kullanılmaktadır.

---

## 📌 Sonuç

Day 10 sonunda oluşturulan telemetry datasetlerinin yapısı, dağılımı, fault etiketleri, fiziksel ilişkileri ve veri sızıntısı açısından doğrulanması tamamlandı.

Beş farklı fault tipinin dataset içerisinde bulunduğu, normal dataset içerisinde fault etiketi olmadığı ve data leakage kontrolünün başarıyla tamamlandığı doğrulandı.

Correlation analysis sonucunda CPU Load, Current Draw, Voltage, Temperature ve Fan RPM arasındaki önemli ilişkiler gözlemlendi.

Tüm Day 10 self-test kontrolleri başarıyla tamamlandı.

---

## 🎯 Sonraki Aşama

Dataset validation aşamasının tamamlanmasından sonra proje anomaly detection aşamasına hazırlanmıştır.

Bir sonraki aşamada doğrulanmış telemetry datasetleri kullanılarak normal sistem davranışının modellenmesi ve anomalilerin tespit edilmesi üzerine çalışmalar yapılabilir.

Day 10 kapsamında herhangi bir Machine Learning veya Autoencoder modeli uygulanmamıştır.

---

## ✅ Gün 10 Durumu

```text
Dataset Generation        ✅
Dataset Schema Validation ✅
Dataset Quality Check     ✅
Distribution Analysis     ✅
Correlation Analysis      ✅
Fault Validation          ✅
Data Leakage Check        ✅
Deterministic Validation  ✅
Demo #2 Report            ✅
```

**Gün 10 tamamlandı. ✅**
