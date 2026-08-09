# 📊 Gün 06 - Günlük Çalışma Raporu

**Tarih:** 8 Ağustos 2026  
**Konu:** Canlı Telemetri Grafikleri ve Telemetri Veri Akışının Geliştirilmesi

---

## 📝 Günün Özeti

Stajın altıncı gününde SpikeEdge Telemetry Panel üzerinde mevcut telemetri altyapısı geliştirilerek canlı verilerin geçmiş değerlerinin de dashboard üzerinde görüntülenmesi sağlandı.

Bu kapsamda sınırlı bir telemetri history buffer oluşturuldu ve simulator tarafından üretilen son telemetri frame'leri bellekte tutulmaya başlandı. Böylece telemetri değerlerinin yalnızca anlık olarak değil, geçmiş değişimleriyle birlikte takip edilmesi mümkün hale getirildi.

Ayrıca telemetri kaynağını simulator uygulamasından bağımsız hale getirmek amacıyla `TelemetrySource` abstraction yapısı geliştirildi. Bu yapı sayesinde ilerleyen aşamalarda simulator yerine WebSocket gibi farklı bir veri kaynağının kullanılabilmesi için gerekli mimari temel oluşturuldu.

Günün sonunda dashboard üzerinde sıcaklık, sistem ve güç değerlerini gösteren canlı grafikler başarıyla çalışır hale getirildi.

---

## 🛠️ Yapılan Çalışmalar

### 1. Telemetri History Buffer

Canlı telemetri verilerinin geçmiş değerlerini saklamak amacıyla bounded bir history buffer geliştirildi.

Buffer yaklaşık olarak:

- 100 telemetri frame'i
- 10 Hz örnekleme hızında yaklaşık 10 saniyelik veri

tutacak şekilde yapılandırıldı.

Bu yapı sayesinde:

- Telemetri geçmişi sınırlı tutuldu.
- Bellek kullanımının sürekli büyümesi engellendi.
- Eski frame'ler otomatik olarak sistemden çıkarıldı.
- Mevcut `TelemetryFrame` veri yapısı yeniden kullanıldı.

---

### 2. TelemetrySource Abstraction

Telemetri kaynağının simulator implementasyonundan bağımsız hale getirilmesi için `TelemetrySource` abstraction yapısı oluşturuldu.

Temel yapı;

```text
start()
stop()
subscribe()

<img width="1917" height="972" alt="image" src="https://github.com/user-attachments/assets/ad3255f6-b559-489c-9366-7b091d9dfd20" />
