🚀 GÜN 24 — ALARM GEÇMİŞİ VE VERİ TABANI SİSTEMİ

🎯 Çalışmanın Konusu

Gün 24 kapsamında, anomali tespit sistemi tarafından oluşturulan alarm olaylarının yalnızca anlık olarak gösterilmesi yerine kalıcı olarak saklanabilmesi üzerine çalışılmıştır.

Amaç, sistemde meydana gelen anomalilerin geçmişe dönük olarak incelenebilmesini ve alarm kayıtlarının daha düzenli şekilde takip edilebilmesini sağlamaktır.

Bu çalışma ile birlikte sistem:

📡 Anomaliyi tespit eder
        ↓
🚨 Alarm durumunu oluşturur
        ↓
💾 Alarm kaydını saklar
        ↓
📋 Geçmiş olayların incelenmesine olanak sağlar


🧠 Neden Alarm Geçmişi?

Canlı bir izleme sisteminde yalnızca mevcut alarm durumunu göstermek yeterli değildir.

Örneğin bir alarm birkaç saniye sonra normale dönerse, operatör daha sonra:

❓ Alarm ne zaman oluştu?
❓ Alarmın seviyesi neydi?
❓ Hangi değerler anormaldi?
❓ Alarm ne zaman sona erdi?
❓ Aynı olay daha önce tekrarlandı mı?

gibi soruların cevaplarını görebilmelidir.

Bu nedenle alarm olaylarının geçmişe dönük olarak saklanması, sistemin izlenebilirliğini ve analiz edilebilirliğini artırmaktadır.


💾 Veri Tabanı Yapısı

Alarm kayıtlarının kalıcı olarak saklanması için SQLite tabanlı bir veri tabanı yapısı ele alınmıştır.

Her alarm olayı için mümkün olan temel bilgiler kayıt altına alınmaktadır:

🕒 Timestamp
🚨 Alarm durumu
📊 Reconstruction MSE
📈 EWMA değeri
🎯 Threshold değeri
🔎 Anomali bilgisi
🏷️ Fault / kanal bilgisi mevcutsa ilgili ilişkilendirme


🔄 Alarm Veri Akışı

Sistemdeki mevcut anomali pipeline'ı korunarak alarm geçmişi bu yapının sonuna eklenmiştir:

Telemetry
   ↓
WebSocket
   ↓
Ring Buffer
   ↓
64-Frame Window
   ↓
Frozen Normalization
   ↓
Autoencoder
   ↓
Reconstruction MSE
   ↓
EWMA
   ↓
Threshold
   ↓
3 of Last 5 Windows
   ↓
🚨 Alarm State
   ↓
💾 Alarm History


📋 Alarm Durumları

Sistemde alarm durumu üç temel aşamada takip edilmektedir:

🟢 NORMAL
Sistemin davranışı normal sınırlar içerisindedir.

🟡 PENDING
Anomali sinyali oluşmuştur ancak alarmın aktif olması için gerekli kalıcılık koşulu henüz tamamlanmamıştır.

🔴 ACTIVE
Son 5 pencerenin en az 3'ünde EWMA threshold değerinin üzerinde olduğunda alarm aktif hale gelir.


📊 Geçmiş Kayıtların Faydası

Alarm geçmişinin tutulması sayesinde sistem yalnızca anlık durum gösteren bir dashboard olmaktan çıkarak geçmiş olayların da incelenebildiği bir izleme yapısına yaklaşmaktadır.

Örneğin geçmişte oluşan bir alarm incelenerek:

- alarmın oluştuğu zaman,
- alarm sırasında sistemin durumu,
- MSE ve EWMA değerleri,
- ilgili telemetri bilgileri

üzerinden olayın daha sonra tekrar değerlendirilmesi mümkün hale gelir.


🔗 Önceki Gün ile İlişkisi

Gün 23'te geliştirilen Channel Attribution yaklaşımı ile birlikte düşünüldüğünde, sistemin alarm hakkında daha fazla bilgi sunması mümkün hale gelmektedir.

Gün 23:
🔎 "Anomali hangi kanallarla ilişkili olabilir?"

Gün 24:
💾 "Bu alarm ne zaman oluştu ve geçmişte ne oldu?"

Bu iki yapı birlikte kullanıldığında anomali olaylarının hem açıklanabilirliği hem de geçmişe dönük izlenebilirliği artırılmaktadır.


🛡️ Sistem Güvenliği ve Veri Akışı

Alarm geçmişinin eklenmesi sırasında mevcut canlı telemetri akışının bozulmamasına dikkat edilmiştir.

Özellikle:

✔️ WebSocket telemetri akışı korunmuştur.
✔️ Ring Buffer yapısı değiştirilmemiştir.
✔️ Autoencoder modeli değiştirilmemiştir.
✔️ Frozen normalization değerleri korunmuştur.
✔️ Frozen threshold değiştirilmemiştir.
✔️ EWMA ve alarm mantığının temel yapısı korunmuştur.

Veri tabanı katmanı mevcut sistemin üzerine ek bir kayıt katmanı olarak ele alınmıştır.


🧪 Doğrulama

Çalışma kapsamında alarm kayıtlarının oluşturulması ve veri tabanına yazılması test edilmiştir.

Ayrıca:

✔️ TypeScript kontrolü
✔️ Lint
✔️ Production Build
✔️ Alarm History testleri
✔️ Veri tabanı kayıt kontrolü

gerçekleştirilmiştir.

Testlerde amaç, alarm oluştuğunda ilgili kaydın doğru şekilde oluşturulması ve mevcut telemetri akışının çalışmaya devam etmesidir.


⚠️ Kapsam ve Sınırlamalar

Bu aşamadaki veri tabanı yapısı proje prototipinin alarm geçmişini gösterebilmesi ve saklayabilmesi amacıyla geliştirilmiştir.

Sistem şu aşamada simüle edilmiş telemetri verileriyle çalışmaktadır.

Gerçek bir endüstriyel ortamda daha gelişmiş:

- veri saklama politikaları,
- uzun süreli log yönetimi,
- kullanıcı yetkilendirmesi,
- veri yedekleme,
- uzak veri tabanı altyapısı

gibi özellikler ayrıca ele alınabilir.


🏁 Gün Sonu

Gün 24 kapsamında, anomali tespit sistemi tarafından oluşturulan alarm olaylarının kalıcı olarak saklanabilmesi için Alarm History ve SQLite tabanlı veri tabanı yapısı üzerinde çalışılmıştır.

Bu çalışma ile birlikte sistemin yalnızca anlık anomalileri göstermesi yerine, geçmiş alarm olaylarının da kayıt altına alınması ve daha sonra incelenebilmesi hedeflenmiştir.

Böylece Digital Twin ve AI tabanlı izleme sisteminin izlenebilirliği ve raporlanabilirliği geliştirilmiştir.


➡️ Sonraki Adım — Gün 25

Bir sonraki aşamada Autoencoder tabanlı anomali tespit yöntemi ile mevcut sabit eşik (Fixed Threshold) yaklaşımının karşılaştırılması ve sonuçların Demo #5 kapsamında değerlendirilmesi planlanmaktadır.
