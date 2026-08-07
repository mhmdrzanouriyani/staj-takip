# Gün 05 - Günlük Çalışma Raporu

**Tarih:** 7 Ağustos 2026  
**Konu:** Telemetri Simülatörünün Geliştirilmesi ve Canlı Dashboard Entegrasyonu

<img width="1917" height="1020" alt="image" src="https://github.com/user-attachments/assets/cd1883fb-eb15-4d44-a219-a8224fd3d840" />
<br>
<img width="1917" height="977" alt="image" src="https://github.com/user-attachments/assets/9df5f550-c547-4535-956e-ad576ed237f4" />

---

## 📌 Günün Özeti

Stajın beşinci gününde SpikeEdge Telemetry projesinin daha önce oluşturulan temel mimarisi üzerinde çalışılarak telemetri sisteminin ilk çalışan prototipi geliştirildi.

Önceki günlerde hazırlanan `PlantModel` ve telemetri veri yapıları kullanılarak deterministik çalışan bir simulator geliştirildi. Simulator tarafından üretilen veriler dashboard'a aktarılmış ve gerçek zamanlı olarak kullanıcı arayüzünde görüntülenmesi sağlandı.

Böylece proje yalnızca statik bir arayüz olmaktan çıkarılarak sürekli telemetri verisi üreten ve bu verileri görüntüleyen çalışan bir yapıya dönüştürüldü.

---

## 🛠️ Yapılan Çalışmalar

### 1. Telemetri Simulatorünün Geliştirilmesi

Önceki günlerde hazırlanan `PlantModel` kullanılarak `Simulator` yapısının temel çalışma mantığı tamamlandı.

Simulator tarafından;

- Telemetri frame'leri oluşturuldu.
- Her frame için sequence numarası üretildi.
- Timestamp bilgisi eklendi.
- `PlantModel` tarafından hesaplanan değerler kullanıldı.
- Kanal değerlerinin belirlenen sınırlar içerisinde kalması sağlandı.
- Deterministik veri üretimi korundu.

Simulator içerisinde kullanılan temel telemetri kanalları:

- Core Temperature
- Ambient Temperature
- Voltage
- Current
- Fan RPM
- CPU Load

---

### 2. Canlı Telemetri Akışının Oluşturulması

Simulator, **10 Hz** örnekleme frekansı ile çalışacak şekilde yapılandırıldı.

Bu yapı sayesinde yaklaşık her **100 ms** içerisinde yeni bir telemetri frame'i oluşturulmaktadır.

Her frame içerisinde;

- Sequence
- Timestamp
- Telemetry Channels

bilgileri bulunmaktadır.

Ayrıca oluşturulan yapı ilerleyen aşamalarda WebSocket gibi gerçek zamanlı iletişim yöntemleriyle kullanılabilecek şekilde modüler tutuldu.

---

### 3. Dashboard Entegrasyonu

Simulator tarafından üretilen telemetri verileri dashboard ile entegre edildi.

Dashboard üzerinde aşağıdaki değerler canlı olarak gösterilmektedir:

| Telemetri | Birim |
|---|---|
| Core Temperature | °C |
| Ambient Temperature | °C |
| Voltage | V |
| Current | A |
| Fan RPM | rpm |
| CPU Load | % |

Simulator çalıştığı sürece bu değerler otomatik olarak güncellenmektedir.

---

### 4. Telemetri Akışının Takip Edilmesi

Veri akışının durumunu daha kolay takip edebilmek için dashboard'a ek bilgiler de eklendi.

Bu bilgiler:

- **Sequence:** Son alınan frame'in sıra numarası
- **Frame Time:** Son frame'in zaman bilgisi
- **Frames Received:** Alınan toplam frame sayısı

Bu değerler sayesinde telemetri akışının devamlılığı ve frame'lerin sıralı şekilde alınması kontrol edilebilmektedir.

---

## 🧪 Sistem Testleri

Geliştirme tamamlandıktan sonra sistemin çalışabilirliği farklı kontroller ile doğrulandı.

Aşağıdaki komutlar başarıyla çalıştırıldı:

```bash
npm run build
npm run lint
npx tsc --noEmit
