# 📊 Gün 08 - Günlük Çalışma Raporu

**Tarih:** 10 Ağustos 2026  
**Konu:** Fault Injection, Ground Truth ve Arıza Senaryolarının Simülasyonu

---

## 📝 Günün Özeti

Stajın sekizinci gününde SpikeEdge Telemetry projesinin mevcut telemetry simulator altyapısı geliştirilerek kontrollü arıza senaryolarının simüle edilebilmesi sağlandı.

Bu kapsamda telemetry akışına bağımsız bir `FaultEngine` katmanı eklenerek farklı arıza türlerinin belirli zaman aralıklarında ve belirli şiddet değerleriyle sisteme uygulanması gerçekleştirildi.

Fault injection yapısı mevcut simulator pipeline içerisine entegre edildi. Böylece normal çalışma koşullarında üretilen telemetry verileri korunurken, belirli zamanlarda kontrollü anormal davranışlar oluşturulabilmektedir.

Günün sonunda beş farklı fault tipi oluşturuldu, ground-truth bilgileri üretildi ve fault senaryoları üzerinde otomatik testler gerçekleştirildi.

---

## 🛠️ Yapılan Çalışmalar

### 1. FaultEngine Yapısının Geliştirilmesi

Telemetry simulator içerisine bağımsız bir `FaultEngine` katmanı eklendi.

FaultEngine'in temel görevi, simulator tarafından üretilen normal telemetry frame'lerini kontrol ederek aktif olan fault senaryolarını ilgili kanallara uygulamaktır.

Geliştirilen telemetry pipeline aşağıdaki şekilde yapılandırıldı:

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

Bu yapı sayesinde fault logic, PlantModel ve React dashboard katmanlarından bağımsız tutuldu.

---

### 2. Fault Modellerinin Oluşturulması

Farklı embedded cihaz problemlerini temsil etmek amacıyla beş farklı fault tipi oluşturuldu.

| ID | Fault Type | Etkilenen Davranış |
| :---: | :--- | :--- |
| **F1** | `temperature_spike` | Core temperature artışı |
| **F2** | `voltage_sag` | Voltage düşüşü |
| **F3** | `current_surge` | Current artışı |
| **F4** | `fan_degradation` | Fan RPM düşüşü |
| **F5** | `sensor_drift` | Sensör değerinde kademeli drift |

Her fault parametreli olarak tasarlandı.

Fault yapısında başlangıç zamanı, süre, severity ve etkilenen channel bilgileri gibi değerler tutulmaktadır.

---

### 3. Temperature Spike

`F1 - temperature_spike` senaryosunda core temperature normal çalışma değerinin üzerine çıkarılmaktadır.

Bu fault, cihazın termal davranışının ve sıcaklık limitlerinin ilerleyen aşamalarda test edilmesi amacıyla kullanılabilecek şekilde tasarlandı.

Temperature spike aktif olduğunda Core Temperature kanalında normal davranıştan belirgin şekilde farklı bir artış gözlemlenebilmektedir.

---

### 4. Voltage Sag

`F2 - voltage_sag` senaryosunda input voltage değeri belirli bir süre boyunca düşürülmektedir.

Canlı dashboard testi sırasında bu fault aktif olduğunda voltage değerinde belirgin bir düşüş gözlemlendi.

Test sırasında fault durumu dashboard üzerinde aşağıdaki şekilde görüntülendi:

```text
faults: on
active: F2-voltage-sag
```

Voltage değerindeki düşüş Power grafiği üzerinden de takip edildi.


### 5. Current Surge

`F3 - current_surge` senaryosunda current draw değerinde geçici bir artış oluşturulmaktadır.

Canlı test sırasında fault aktif olduğunda current değerinin normal seviyenin üzerine çıktığı ve Power grafiğinde belirgin bir değişim oluşturduğu gözlemlendi.

Test sırasında current değeri yaklaşık `2.84 A` seviyesine ulaştı.

Dashboard üzerinde aktif fault aşağıdaki şekilde görüntülendi:

```text
faults: on
active: F3-current-surge
```


### 6. Fan Degradation

`F4 - fan_degradation` senaryosunda cihazın mevcut yük ve sıcaklık koşullarına göre beklenen fan RPM değeri düşürülmektedir.

Bu senaryo fan arızası veya yetersiz soğutma durumlarını temsil etmek amacıyla geliştirilmiştir.

Fan degradation aktif olduğunda fan RPM değerinin normal çalışma koşullarına göre daha düşük kalması beklenmektedir.

Bu davranış ilerleyen aşamalarda sıcaklık ve soğutma sistemi arasındaki ilişkinin analiz edilmesinde kullanılabilecektir.

---

### 7. Sensor Drift

`F5 - sensor_drift` senaryosunda seçilen telemetry channel değeri anlık bir sıçrama yerine zaman içerisinde kademeli olarak normal değerinden uzaklaştırılmaktadır.

Bu yaklaşım gerçek sensörlerde görülebilecek kalibrasyon kayması ve uzun süreli ölçüm sapmalarını simüle etmek amacıyla kullanıldı.

Sensor drift özellikle anomaly detection sistemlerinin ani fault'ların yanında yavaş gelişen anomalileri de tespit edebilmesi açısından önemlidir.

---

## 🎯 Ground Truth

Fault injection sisteminin önemli bir parçası olarak her fault için ground-truth bilgisi oluşturuldu.

Ground truth sayesinde hangi fault'un hangi frame aralığında aktif olduğu sistem tarafından kesin olarak bilinmektedir.

Ground-truth kayıtlarında aşağıdaki bilgiler tutulmaktadır:

- Fault ID
- Fault Type
- Start Frame
- End Frame
- Start Time
- End Time
- Severity
- Affected Channels

Bu yapı ilerleyen aşamalarda kullanılacak anomaly detection modellerinin değerlendirilmesi açısından önemlidir.

Normal telemetry verileri ile bilinen fault verileri birbirinden ayrılabildiği için sistem daha sonra model performansının ölçülmesine uygun hale getirilmiştir.

---

## 🎲 Deterministik Fault Injection

Fault senaryolarının tekrar üretilebilir olması için mevcut seeded random altyapısı kullanılmaya devam edildi.

Projede daha önce kullanılan:

```text
Mulberry32
createSeededRandom(seed)
fork(streamId)
```

yapıları korunarak fault injection mekanizmasına entegre edildi.

Aynı seed, aynı workload profile ve aynı fault configuration kullanıldığında aynı fault davranışının tekrar oluşturulabilmesi sağlandı.

Bu özellik özellikle test, hata ayıklama ve dataset üretimi sırasında önem taşımaktadır.

---

## 🖥️ Dashboard Entegrasyonu

Fault injection sistemi mevcut dashboard mimarisine minimum değişiklik ile entegre edildi.

Dashboard üzerinde fault sisteminin aktif olup olmadığı ve o anda çalışan fault bilgisi gösterilmektedir.

Örneğin:

```text
profile: variable
seed: 1337
noise: on
faults: on
active: F2-voltage-sag
```

Bu bilgiler sayesinde test sırasında fault senaryosunun hangi aşamada olduğu doğrudan dashboard üzerinden takip edilebilmektedir.

---


Fault senaryolarını karşılaştırabilmek amacıyla normal telemetry akışı da kontrol edildi.

Bu aşamada fault aktif değilken simulator normal workload profile üzerinden telemetry üretmeye devam etti.

Normal baseline sırasında dashboard üzerinde:

- CPU Load
- Temperature
- Voltage
- Current
- Fan RPM

değerlerinin normal çalışma aralığında üretildiği doğrulandı.


---

## 🧪 Canlı Dashboard Testleri

Fault injection sistemi gerçek zamanlı dashboard üzerinde test edildi.

Test sırasında aşağıdaki telemetry kanalları gözlemlendi:

- Core Temperature
- Ambient Temperature
- Voltage
- Current
- Fan RPM
- CPU Load

Ayrıca canlı SVG grafikler üzerinden değerlerin zaman içerisindeki değişimleri takip edildi.

Fault aktif olduğunda ilgili telemetry kanalında beklenen değişimin oluştuğu ve dashboard grafiklerine yansıdığı doğrulandı.

---

## 🔬 Fault Senaryolarının Doğrulanması

### Test 01 — Temperature Spike

`F1 - temperature_spike` fault'u aktif edildi.

Bu senaryoda Core Temperature kanalının normal çalışma davranışından farklı şekilde yükselmesi gözlemlendi.

Temperature grafiğinde fault etkisinin zaman içerisinde telemetry değerine yansıdığı doğrulandı.

---

### Test 02 — Voltage Sag

`F2 - voltage_sag` fault'u aktif edildi.

Voltage değerinde belirgin bir düşüş gözlemlendi ve Power grafiğinde bu değişim açık şekilde takip edildi.

Dashboard üzerinde aktif fault bilgisi:

```text
F2-voltage-sag
```

olarak görüntülendi.


---

### Test 03 — Current Surge

`F3 - current_surge` fault'u aktif edildi.

Current değeri normal çalışma seviyesinin üzerine çıkarıldı.

Test sırasında yaklaşık `2.84 A` seviyesinde current değeri gözlemlendi.

Power grafiğinde current kanalındaki artış doğrulandı.


---

### Test 04 — Fan Degradation

`F4 - fan_degradation` fault'u aktif edildi.

Bu testte fan RPM değerinin normal sistem davranışına göre daha düşük seviyede tutulduğu doğrulandı.

Bu durum, fanın yeterli soğutma sağlayamaması senaryosunu temsil etmektedir.

---

### Test 05 — Sensor Drift

`F5 - sensor_drift` fault'u aktif edildi.

Seçilen telemetry channel üzerinde ani bir sıçrama yerine kademeli değişim oluşturuldu.

Bu test sayesinde sistemin yalnızca ani fault'lara değil, yavaş gelişen sensor anomalies davranışlarına da uygun veri üretebildiği doğrulandı.

---

## ⚙️ Fault Configuration

Fault injection sistemi merkezi telemetry configuration üzerinden kontrol edilebilmektedir.

Temel configuration değerleri:

```text
FAULT_INJECTION_ENABLED = true

Workload Profile = variable
Seed = 1337
Noise = enabled
Sample Rate = 10 Hz
History Capacity = 100
```

Fault senaryoları `TELEMETRY_FAULTS` configuration yapısı üzerinden tanımlanabilmektedir.

Bu sayede farklı fault senaryoları Simulator kodunda değişiklik yapılmadan test edilebilmektedir.

---

## 🧠 FaultEngine ve Simulator Entegrasyonu

Day 08 kapsamında Simulator yapısı fault logic ile doğrudan birbirine bağlanmak yerine `FaultEngine` üzerinden genişletildi.

Simulator normal telemetry değerlerini üretmekte, FaultEngine ise aktif fault senaryolarını bu değerler üzerine uygulamaktadır.

Bu yaklaşım sayesinde ilerleyen aşamalarda yeni fault tiplerinin eklenmesi mevcut PlantModel veya Dashboard kodlarının değiştirilmesini gerektirmeyecektir.

---

## 🧾 Ground Truth Event

Her fault olayı için `GroundTruthEvent` yapısı oluşturuldu.

Bu yapı fault'un gerçek zamanlı telemetry akışı içerisinde hangi zaman ve frame aralığında aktif olduğunu takip etmek için kullanılmaktadır.

Ground truth verileri ilerleyen aşamalarda:

- Dataset labeling
- Anomaly detection evaluation
- Fault classification
- Model accuracy measurement

gibi işlemlerde kullanılabilecek şekilde tasarlandı.

---

## 🧪 Otomatik Self-Test

Day 08 için özel bir self-test sistemi oluşturuldu.

Test scripti:

```bash
npm run test:day08
```

komutu üzerinden çalıştırılabilmektedir.

Self-test içerisinde toplam 10 kontrol gerçekleştirildi.

Kontrol edilen başlıca durumlar:

1. Normal telemetry üretimi
2. Temperature spike
3. Voltage sag
4. Current surge
5. Fan degradation
6. Sensor drift
7. Fault timing
8. Ground truth event
9. Deterministic behavior
10. Bounds / clamp kontrolü

Tüm kontroller başarıyla tamamlandı.

---

## ✅ Validation

Day 08 sonunda aşağıdaki validation komutları çalıştırıldı:

```bash
npx tsc --noEmit
npm run lint
npm run build
npm run test:day08
```

Sonuçlar:

| Kontrol | Sonuç |
| :--- | :---: |
| TypeScript Type Check | ✅ Passed |
| ESLint | ✅ Passed |
| Production Build | ✅ Passed |
| Day 08 Self-Test | ✅ Passed |
| Self-Test Checks | ✅ 10 / 10 |

TypeScript type-check işlemi başarıyla tamamlandı.

ESLint kontrolünde herhangi bir warning veya error bulunmadı.

Next.js production build işlemi başarıyla tamamlandı.

Day 08 self-test içerisindeki 10 kontrolün tamamı başarıyla geçti.

---

## 🏁 Gün Sonu Değerlendirmesi

Sekizinci günün sonunda SpikeEdge Telemetry simulator altyapısı normal çalışma koşullarının yanında kontrollü arıza senaryoları da üretebilecek seviyeye getirildi.

Beş farklı fault tipi oluşturuldu ve bunların telemetry verileri üzerindeki etkileri modellenerek test edildi.

FaultEngine'in mevcut simulator pipeline içerisine eklenmesiyle normal telemetry üretimi ile fault injection birbirinden ayrılmış oldu.

Ayrıca ground-truth mekanizması sayesinde fault olaylarının ne zaman başladığı, ne kadar sürdüğü ve hangi telemetry kanallarını etkilediği kayıt altına alınabilmektedir.

Bu yapı, projenin sonraki aşamalarında gerçekleştirilecek anomaly detection ve fault analysis çalışmalarının temelini oluşturmaktadır.

---

## 📌 Day 08 Çıktıları

Günün sonunda aşağıdaki bileşenler projeye kazandırılmıştır:

- `FaultEngine`
- `temperature_spike`
- `voltage_sag`
- `current_surge`
- `fan_degradation`
- `sensor_drift`
- `GroundTruthEvent`
- Fault configuration
- Fault status dashboard integration
- Day 08 automated self-test
- Deterministic fault simulation
- Fault-aware telemetry pipeline

---

## 🚫 Bu Gün Kapsamına Dahil Olmayan Çalışmalar

Day 08 kapsamında aşağıdaki özellikler henüz geliştirilmemiştir:

- WebSocket telemetry source
- AI anomaly detection
- TensorFlow model
- Three.js / 3D Viewer
- Alarm management UI
- Production fault monitoring
- Blender integration

Bu özellikler projenin ilerleyen aşamalarında değerlendirilecektir.

---

## 🚀 Sonraki Aşama

Bir sonraki aşamada oluşturulan normal ve fault içeren telemetry akışlarının offline olarak kaydedilmesi ve eğitim/test verisi olarak kullanılabilecek şekilde düzenlenmesi planlanmaktadır.

Amaç, normal davranış ile fault içeren davranışların ayrıştırılabileceği temiz ve tekrar üretilebilir bir telemetry dataset altyapısı oluşturmaktır.

Bu dataset ilerleyen aşamalarda anomaly detection modeli için training ve evaluation süreçlerinde kullanılabilecektir.
