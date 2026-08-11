# 📊 Gün 07 - Günlük Çalışma Raporu

**Tarih:** 9 Ağustos 2026  
**Konu:** Telemetri Yük Profilleri, Deterministik Gürültü ve Simülasyon Testleri

---

## 📝 Günün Özeti

Stajın yedinci gününde SpikeEdge Telemetry projesindeki telemetri simülasyon altyapısı geliştirilerek farklı sistem yük davranışlarının modellenmesi sağlandı.

Bu kapsamda cihazın farklı çalışma koşullarını temsil eden üç farklı workload profili oluşturuldu: `idle`, `variable` ve `sustained`.

Her profil farklı CPU yük davranışı üretmekte ve bu yükün PlantModel üzerinden sıcaklık, fan devri, akım ve voltaj gibi diğer telemetri kanallarına etkisi simüle edilmektedir.

Ayrıca simülasyon sonuçlarının tekrar üretilebilir olması için seeded random mekanizması ve kontrollü bir Noise Pipeline kullanıldı. Böylece aynı seed ve aynı çalışma koşulları kullanıldığında benzer telemetri akışlarının tekrar oluşturulabilmesi sağlandı.

Günün sonunda üç farklı workload profili canlı dashboard üzerinde test edildi ve sistemin beklenen fiziksel davranışları ürettiği doğrulandı.

---

## 🛠️ Yapılan Çalışmalar

### 1. Workload Profillerinin Oluşturulması

Simulator içerisine farklı cihaz çalışma koşullarını temsil eden üç farklı workload profili eklendi.

`idle` profili düşük sistem yükünü temsil etmektedir. Bu profilde CPU yükü düşük seviyelerde tutulmakta ve buna bağlı olarak akım, core temperature ve fan RPM değerlerinin daha düşük seviyelerde kalması beklenmektedir.

`variable` profili değişken sistem yükünü temsil etmektedir. Bu profilde CPU yükü zaman içerisinde yükselip düşmekte ve PlantModel içerisindeki fiziksel ilişkiler sayesinde sıcaklık, akım, fan RPM ve voltaj gibi diğer kanallar da CPU yüküne bağlı olarak değişmektedir.

`sustained` profili ise uzun süreli yüksek sistem yükünü temsil etmektedir. Bu profilde CPU yükü yaklaşık olarak yüksek seviyelerde tutulmakta ve buna bağlı olarak core temperature, fan RPM ve current değerlerinin yükselmesi beklenmektedir. Aynı zamanda yük artışı sonucunda voltage droop etkisi de gözlemlenebilmektedir.

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

### 6. Workload Profil Testleri

İlk test `idle` profili ile gerçekleştirildi. Bu testte CPU Load yaklaşık %10 seviyelerinde, Core Temperature yaklaşık 31°C, Fan RPM yaklaşık 915 RPM ve Current yaklaşık 0.56 A seviyelerinde gözlemlendi. Elde edilen sonuçlar düşük sistem yükünü temsil eden idle profili ile uyumlu bulundu.

İkinci test `variable` profili ile gerçekleştirildi. Bu testte CPU yükünün zaman içerisinde değiştiği ve buna bağlı olarak diğer telemetri değerlerinin de değişim gösterdiği gözlemlendi. Canlı grafikler üzerinden CPU Load, Fan RPM, Temperature, Voltage ve Current değerlerinin zamana bağlı davranışları incelendi.

Üçüncü test `sustained` profili ile gerçekleştirildi. Bu test sırasında CPU Load yaklaşık %80 seviyesine ulaşırken Core Temperature yaklaşık 53°C, Fan RPM yaklaşık 3192 RPM, Current yaklaşık 1.58 A ve Voltage yaklaşık 11.98 V seviyelerinde gözlemlendi.

Bu sonuçlar yüksek CPU yükünün sistem içerisindeki diğer fiziksel parametreleri etkilediğini ve PlantModel'in bu ilişkileri beklenen şekilde simüle ettiğini gösterdi.

---

## 🔍 Sistem Doğrulama

Day 07 sonunda geliştirilen yapı üzerinde TypeScript, ESLint ve production build kontrolleri gerçekleştirildi.

Aşağıdaki komutlar başarıyla çalıştırıldı:

    npx tsc --noEmit
    npm run lint
    npm run build

TypeScript type-check işlemi başarıyla tamamlandı.

ESLint kontrolünde herhangi bir warning veya error bulunmadı.

Production build işlemi de başarıyla tamamlandı ve Next.js uygulaması production için sorunsuz şekilde derlendi.

Build sonucunda:

    ✓ Compiled successfully
    ✓ Linting and checking validity of types
    ✓ Collecting page data
    ✓ Generating static pages
    ✓ Finalizing page optimization

adımları başarıyla tamamlandı.

---

## 🏁 Gün Sonu Değerlendirmesi

Yedinci günün sonunda SpikeEdge Telemetry simulator altyapısı farklı çalışma koşullarını temsil edebilecek seviyeye getirildi.

Idle, variable ve sustained workload profilleri oluşturuldu ve canlı dashboard üzerinde test edildi. Seeded random mekanizması sayesinde simülasyonun tekrar üretilebilir olması sağlandı. Noise Pipeline ile sensör verilerine daha gerçekçi davranış kazandırıldı ve bounds/clamp kontrolleri ile değerlerin tanımlanan sınırlar içerisinde tutulması sağlandı.

Böylece sistem yalnızca sabit telemetri değerleri üreten bir simulator olmaktan çıkarılarak farklı yük koşullarını ve gerçek sensör davranışlarını temsil edebilen daha gerçekçi bir telemetri simülasyon altyapısına dönüştürüldü.

Ayrıca TypeScript, ESLint ve production build kontrollerinin tamamı başarıyla geçilerek geliştirilen yapının mevcut dashboard mimarisi ile uyumlu olduğu doğrulandı.

---

## 🚀 Sonraki Aşama

Bir sonraki aşamada oluşturulan normal ve değişken telemetri akışları üzerinde fault injection ve anomaly senaryolarının geliştirilmesi planlanmaktadır. Bu kapsamda farklı hata durumlarının zaman damgalı olarak oluşturulması ve bu durumların telemetri verileri üzerindeki etkilerinin incelenmesi hedeflenmektedir.
