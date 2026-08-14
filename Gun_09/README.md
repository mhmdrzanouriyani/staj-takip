# 📊 Gün 09 - Günlük Çalışma Raporu

**Tarih:** 11 Ağustos 2026
**Konu:** Telemetri Dataset Oluşturma, Etiketleme ve CSV Export

---

## 📝 Günün Özeti

Stajın dokuzuncu gününde SpikeEdge Telemetry projesinde daha önce geliştirilen telemetri simülasyon altyapısı kullanılarak etiketlenmiş telemetri datasetlerinin oluşturulması üzerine çalışıldı.

Day 07 ve Day 08 kapsamında oluşturulan workload profilleri, deterministik simülasyon yapısı, NoisePipeline ve FaultEngine kullanılarak üretilen telemetri verilerinin daha sonra analiz ve anomaly detection çalışmalarında kullanılabilmesi amacıyla kayıt ve dışa aktarma yapısı geliştirildi.

Bu kapsamda mevcut `TelemetryFrame` verilerini toplayabilen bounded bir `DatasetRecorder`, dataset kayıtlarını temsil eden typed yapı ve sabit CSV formatında çıktı oluşturabilen bir CSV exporter geliştirildi.

Ayrıca Day 08 kapsamında oluşturulan `GroundTruthEvent` yapısı kullanılarak telemetri verilerine `normal` veya `fault` etiketi verilmesi ve aktif fault durumunda ilgili fault ID bilgisinin saklanması sağlandı.

---

## 🛠️ Yapılan Çalışmalar

### 1. Dataset Recorder

Mevcut Simulator tarafından üretilen `TelemetryFrame` verilerini toplayabilmek amacıyla `DatasetRecorder` yapısı oluşturuldu.

Recorder mevcut telemetri pipeline'ına müdahale etmeden çalışacak şekilde tasarlandı. Böylece Dashboard tarafındaki canlı telemetri akışı değiştirilmeden dataset üretilebilmektedir.

Bellek kullanımının sınırsız büyümemesi için recorder bounded bir yapı olarak tasarlandı ve maksimum frame sayısı sınırlandırıldı.

---

### 2. Dataset Etiketleme

Day 08 kapsamında oluşturulan `GroundTruthEvent` yapısı dataset etiketleme işleminde kullanıldı.

Normal telemetri kayıtlarında:

```text
label = normal
fault_id = empty
```

Fault bulunan kayıtlarda:

```text
label = fault
fault_id = F1 / F2 / F3 / F4 / F5
```

şeklinde etiketleme gerçekleştirildi.

Bu yapı sayesinde dataset içerisindeki normal çalışma koşulları ile hata durumları birbirinden ayrılabilir hale getirildi.

---

### 3. Dataset Row Yapısı

Dataset içerisindeki her kayıt için typed bir yapı oluşturuldu.

Her dataset satırında aşağıdaki bilgiler bulunmaktadır:

```text
timestamp
sequence
temp_core
temp_ambient
voltage_in
current_draw
fan_rpm
cpu_load
label
fault_id
```

Bu yapı mevcut TelemetryFrame içerisindeki temel telemetri kanallarını korurken dataset analizi için gerekli label ve fault bilgilerini de içermektedir.

---

### 4. CSV Export

Dataset verilerinin daha sonra Python, veri analizi veya machine learning çalışmalarında kullanılabilmesi amacıyla CSV export mekanizması geliştirildi.

CSV dosyalarında sabit bir header kullanıldı:

```text
timestamp,sequence,temp_core,temp_ambient,voltage_in,current_draw,fan_rpm,cpu_load,label,fault_id
```

CSV exporter'ın React Dashboard'dan bağımsız olması sağlandı. Böylece dataset üretimi frontend'e bağlı olmadan gerçekleştirilebilmektedir.

---

### 5. Deterministik Dataset Üretimi

Dataset üretiminde mevcut Simulator altyapısı yeniden kullanıldı.

Seed değeri:

```text
1337
```

olarak kullanıldı.

Sample rate:

```text
10 Hz
```

ve workload profili:

```text
variable
```

olarak kullanıldı.

Deterministik random yapısı sayesinde aynı simulator ayarları kullanıldığında tekrar üretilebilir datasetler elde edilmesi sağlandı.

Bu özellik özellikle testlerin tekrarlanabilir olması ve daha sonraki anomaly detection çalışmalarında aynı dataset üzerinde karşılaştırma yapılabilmesi açısından önemlidir.

---

### 6. Normal Dataset

Normal çalışma davranışını temsil eden bir dataset oluşturuldu.

Dosya:

```text
data/day09-normal.csv
```

Normal dataset içerisinde **800 örnek** telemetri verisi bulunmaktadır.

Bu kayıtlar herhangi bir aktif fault olmadan Simulator tarafından üretilen normal telemetri akışından oluşturuldu.

---

### 7. Fault Dataset

Fault senaryolarını içeren ikinci dataset oluşturuldu.

Dosya:

```text
data/day09-faults.csv
```

Bu dataset toplamda **800 örnek** içermekte ve bunların **300 tanesi fault-labeled** olarak oluşturulmuştur.

Fault kayıtlarında Day 08 kapsamında oluşturulan aşağıdaki fault ID'leri kullanılmaktadır:

```text
F1 - Temperature Spike
F2 - Voltage Sag
F3 - Current Surge
F4 - Fan Degradation
F5 - Sensor Drift
```

Bu sayede farklı hata senaryolarının daha sonraki analiz ve anomaly detection çalışmalarında kullanılabilecek şekilde etiketlenmesi sağlandı.

---

## 🔍 Dataset Doğrulama

Day 09 sonunda oluşturulan dataset yapısı için otomatik self-test geliştirildi.

Aşağıdaki komut başarıyla çalıştırıldı:

```bash
npm run test:day09
```

Test sonucunda:

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

sonucu elde edildi.

Bu sonuçlar dataset satırlarının doğru oluşturulduğunu, label bilgilerinin doğru şekilde atandığını, sequence değerlerinin sıralı olduğunu, CSV export işleminin başarılı olduğunu ve deterministic üretimin tekrar edilebilir olduğunu göstermektedir.

---

## 🧪 Proje Genel Doğrulaması

Day 09 geliştirmeleri sonrasında mevcut proje yapısının bozulmadığından emin olmak amacıyla önceki validation kontrolleri de başarıyla gerçekleştirildi.

TypeScript type-check, ESLint, production build ve Day 08 self-test kontrolleri başarılı şekilde tamamlandı.

Böylece Day 09 kapsamında eklenen dataset katmanının mevcut telemetri simulator ve fault injection mimarisiyle uyumlu olduğu doğrulandı.

---

## 🏗️ Gün Sonu Mimari

Day 09 sonunda veri akışı aşağıdaki yapıya ulaşmıştır:

```text
PlantModel
    ↓
FaultEngine
    ↓
NoisePipeline
    ↓
TelemetryFrame
    ↓
DatasetRecorder
    ↓
Dataset Row
    ↓
CSV Export
    ↓
day09-normal.csv
day09-faults.csv
```

Dashboard canlı telemetri göstermeye devam ederken dataset üretimi ayrı bir katman üzerinden gerçekleştirilmektedir.

---

## 🎯 Gün Sonu Değerlendirmesi

Dokuzuncu günün sonunda SpikeEdge Telemetry projesinde üretilen telemetri verilerinin yalnızca canlı olarak görüntülenmesi yerine kayıt altına alınması ve analiz edilebilir dataset formatına dönüştürülmesi sağlandı.

Normal ve fault senaryoları için etiketlenmiş CSV datasetleri oluşturuldu. Day 08 kapsamında geliştirilen GroundTruthEvent yapısı kullanılarak fault kayıtlarının hangi hata senaryosuna ait olduğu belirlenebilir hale getirildi.

Ayrıca dataset üretiminin deterministik olması sağlanarak aynı seed ile tekrar üretilebilir veri elde edilmesi mümkün hale getirildi.

Bu çalışma ile proje, yalnızca embedded telemetry monitoring paneli olmaktan çıkarak ileride anomaly detection ve machine learning çalışmalarında kullanılabilecek veri üretim altyapısına sahip hale getirildi.

---

## 🚀 Sonraki Aşama

Bir sonraki aşamada oluşturulan normal ve fault-labeled datasetler kullanılarak telemetri verilerindeki anormal davranışların tespit edilmesi üzerine çalışmalar yapılması planlanmaktadır.

Bu aşamada öncelikle normal telemetri davranışının analiz edilmesi, daha sonra fault örneklerinin normal davranıştan nasıl ayrılabileceğinin incelenmesi ve anomaly detection yaklaşımının belirlenmesi hedeflenmektedir.

Day 09 kapsamında machine learning veya anomaly detection algoritması henüz uygulanmamıştır. Oluşturulan datasetler bu çalışmalar için temel veri kaynağı olarak hazırlanmıştır.
