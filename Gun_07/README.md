# 📊 Gün 07 - Günlük Çalışma Raporu

**Tarih:** 9 Ağustos 2026  
**Konu:** Telemetri Yük Profilleri, Deterministik Gürültü ve Simülasyon Testleri

---

## 📝 Günün Özeti

Stajın yedinci gününde SpikeEdge Telemetry projesindeki telemetri simülasyon altyapısı geliştirilerek farklı sistem yük davranışlarının modellenmesi sağlandı.

Bu kapsamda cihazın farklı çalışma koşullarını temsil eden üç farklı workload profili oluşturuldu: `idle`, `variable` ve `sustained`.

Her profil farklı CPU yük davranışı üretmekte ve bu yükün PlantModel üzerinden sıcaklık, fan devri, akım ve voltaj gibi diğer telemetri kanallarına etkisi simüle edilmektedir.

Ayrıca simülasyon sonuçlarının tekrar üretilebilir olması için seeded random mekanizması ve kontrollü bir Noise Pipeline kullanıldı. Böylece aynı seed ve aynı çalışma koşulları kullanıldığında telemetri akışının tekrar üretilebilir olması sağlandı.

Günün sonunda üç farklı workload profili canlı dashboard üzerinde test edildi ve sistemin beklenen fiziksel davranışları ürettiği doğrulandı.

---

## 🛠️ Yapılan Çalışmalar

### 1. Workload Profillerinin Oluşturulması

Simulator içerisine farklı cihaz çalışma koşullarını temsil eden üç farklı workload profili eklendi.

`idle` profili düşük sistem yükünü temsil etmektedir. Bu profilde CPU yükü düşük seviyelerde tutulmakta ve buna bağlı olarak akım, core temperature ve fan RPM değerlerinin daha düşük seviyelerde kalması beklenmektedir.

`variable` profili değişken sistem yükünü temsil etmektedir. Bu profilde CPU yükü zaman içerisinde yükselip düşmekte ve PlantModel içerisindeki fiziksel ilişkiler sayesinde sıcaklık, akım, fan RPM ve voltaj gibi diğer kanallar da CPU yüküne bağlı olarak değişmektedir.

`sustained` profili ise uzun süreli yüksek sistem yükünü temsil etmektedir. Bu profilde CPU yükü yüksek seviyelerde tutulmakta ve buna bağlı olarak core temperature, fan RPM ve current değerlerinin yükselmesi beklenmektedir. Aynı zamanda yük artışı sonucunda voltage droop etkisi de gözlemlenebilmektedir.

---

### 2. Deterministik Random Yapısı

Simülasyon sonuçlarının tekrar üretilebilir olması amacıyla seeded random mekanizması kullanıldı.

Simulator içerisinde `Mulberry32`, `createSeededRandom(seed)` ve `fork(streamId)` yapıları kullanılarak kontrollü rastgelelik sağlandı.

Kullanılan temel seed değeri `1337` olarak belirlendi.

Bu yapı sayesinde aynı seed ve aynı simulator ayarları kullanıldığında telemetri kanal değerlerinin tekrar üretilebilir olması sağlandı. Bu özellik özellikle sistem testleri, hata ayıklama ve farklı workload profillerinin karşılaştırılması açısından önemlidir.

---

### 3. Noise Pipeline

Gerçek sensör davranışına daha yakın telemetri verileri elde etmek amacıyla simulator içerisine kontrollü bir Noise Pipeline eklendi.

Noise işlemi drift, jitter, outlier kontrolü ve quantization aşamalarından oluşmaktadır.

Noise özelliği merkezi telemetri konfigürasyonu üzerinden açılıp kapatılabilmektedir.

Ayrıca noise uygulandıktan sonra telemetri değerlerinin tanımlanan fiziksel sınırlar içerisinde kalması için bounds ve clamp kontrolleri uygulanmaktadır.

Bu sayede simülasyon verilerinin tamamen yapay ve sabit görünmesi yerine daha gerçekçi sensör davranışına yakın olması sağlandı.

---

### 4. Telemetri Konfigürasyonu

Telemetri simulator ayarları merkezi bir configuration yapısında tutuldu.

Kullanılan temel ayarlar şu şekildedir:

- Sample Rate: 10 Hz
- Seed: 1337
- History Capacity: 100 frame
- Noise: Enabled
- Workload Profile: variable

Workload profili `idle`, `variable` veya `sustained` olarak değiştirilebilmektedir.

Bu yapı sayesinde simulator içerisindeki ana kod değiştirilmeden farklı çalışma senaryolarının test edilmesi mümkün hale getirildi.

---

### 5. Dashboard Üzerinde Test

Oluşturulan workload profilleri mevcut SpikeEdge Telemetry dashboard üzerinde test edildi.

Dashboard üzerinde aşağıdaki telemetri kanalları canlı olarak gözlemlendi:

- Core Temperature
- Ambient Temperature
- Voltage
- Current
- Fan RPM
- CPU Load

Ayrıca daha önce oluşturulan canlı SVG grafikler kullanılarak telemetri değerlerinin zaman içerisindeki değişimleri takip edildi.

Testler sırasında CPU yükündeki değişimin PlantModel üzerinden sıcaklık, fan RPM, current ve voltage gibi diğer kanallara yansıdığı gözlemlendi.

---

## 🧪 6. Workload Profil Testleri

### 🟢 Test 01 — Idle Profile

İlk test `idle` profili ile gerçekleştirildi.

Test ayarları:

```text
Profile     : idle
Seed        : 1337
Noise       : ON
Sample Rate : 10 Hz
