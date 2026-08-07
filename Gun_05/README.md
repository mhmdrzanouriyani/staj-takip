# Gün 05 - Günlük Çalışma Raporu

**Tarih:** 7 Ağustos 2026  
**Konu:** Telemetri Simülatörünün Gerçekleştirilmesi ve Canlı Dashboard Entegrasyonu

---

# Günün Özeti

Stajın beşinci gününde SpikeEdge Telemetry projesinde daha önce oluşturulan sistem mimarisi çalışır hale getirildi. Önceki günlerde hazırlanan `PlantModel` ve telemetri veri yapıları kullanılarak gerçek zamanlı veri üreten deterministik bir simulator geliştirildi.

Simulator tarafından üretilen telemetri verileri dashboard tarafına aktarılmış ve kullanıcı arayüzünde canlı olarak gösterilmiştir. Böylece proje statik bir arayüzden çıkarılarak sürekli veri üreten ve bu verileri görüntüleyen çalışan bir prototip haline getirilmiştir.

---

# Yapılan Çalışmalar

## 1. Telemetri Simulatorünün Geliştirilmesi

Önceki günlerde oluşturulan `PlantModel` kullanılarak `Simulator` yapısı çalışır hale getirildi.

Simulator;

- Telemetri frame'leri üretmektedir.
- Her frame için sequence numarası oluşturmaktadır.
- Zaman bilgisini üretmektedir.
- `PlantModel` tarafından hesaplanan kanal değerlerini kullanmaktadır.
- Telemetri değerlerini tanımlanan sınırlar içerisinde tutmaktadır.
- Deterministik çalışmaktadır.

Üretilen temel kanallar:

- Core Temperature
- Ambient Temperature
- Voltage
- Current
- Fan RPM
- CPU Load

---

## 2. Canlı Telemetri Akışının Oluşturulması

Simulator 10 Hz örnekleme frekansı ile çalışacak şekilde yapılandırıldı.

Böylece yaklaşık her 100 ms'de bir yeni telemetri frame'i oluşturulmaktadır.

Üretilen veriler içerisinde sequence numarası ve timestamp bilgileri de tutulmaktadır.

Bu yapı ilerleyen aşamalarda WebSocket veya başka bir gerçek zamanlı veri kaynağı ile değiştirilebilecek şekilde modüler olarak tasarlanmıştır.

---

## 3. Dashboard Entegrasyonu

Simulator tarafından üretilen telemetri verileri mevcut dashboard yapısına bağlandı.

Dashboard üzerinde aşağıdaki değerler canlı olarak görüntülenmektedir:

- Core Temperature
- Ambient Temperature
- Voltage
- Current
- Fan RPM
- CPU Load

Değerler simulator çalıştığı sürece otomatik olarak güncellenmektedir.

---

## 4. Telemetri Frame Takibi

Dashboard'a ek olarak telemetri akışının takip edilebilmesi için;

- Sequence
- Frame Time
- Frames Received

bilgileri de gösterilmektedir.

Bu bilgiler sayesinde veri akışının devam edip etmediği ve frame'lerin sıralı şekilde alınıp alınmadığı kontrol edilebilmektedir.

---

## 5. Sistem Testleri

Geliştirme tamamlandıktan sonra proje farklı seviyelerde test edildi.

Aşağıdaki kontroller başarıyla gerçekleştirildi:
```text
npm run build
npm run lint
npx tsc --noEmit
```
gorseller :

<img width="1917" height="1020" alt="image" src="https://github.com/user-attachments/assets/8e157c4f-bd10-446c-8688-97f9f3369997" />


<img width="1915" height="975" alt="image" src="https://github.com/user-attachments/assets/f85eb077-bdb9-42b6-b7b2-f62242ea1ed7" />


