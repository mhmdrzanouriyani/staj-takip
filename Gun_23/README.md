Gun_23 — Kanal Bazlı Anomali İlişkilendirme

Çalışmanın Konusu

Bu çalışmada, Autoencoder tabanlı anomali tespit sisteminde oluşan Reconstruction Error değerlerinin kanal bazında incelenmesi ve anomalinin hangi telemetri kanallarıyla daha fazla ilişkili olduğunun belirlenmesi amaçlanmıştır.

Sistemde kullanılan altı temel telemetri kanalı:
- temp_core
- temp_ambient
- voltage_in
- current_draw
- fan_rpm
- cpu_load

Çalışmanın Amacı

Autoencoder normal çalışma verileri üzerinde eğitildiği için normal davranıştan uzaklaşan bir pencere daha yüksek Reconstruction MSE üretebilir. Ancak yalnızca toplam MSE değerini göstermek, anomalinin hangi fiziksel değişkenle ilişkili olduğunu açıklamak için yeterli değildir.

Bu nedenle 64 frame'lik pencere üzerinden kanal bazlı reconstruction error değerleri incelenerek Channel Attribution yaklaşımı ele alınmıştır.

Attribution Yaklaşımı

Her 64 frame'lik pencere için Autoencoder'ın giriş ve yeniden oluşturduğu değerler karşılaştırılır. Kanal bazında reconstruction error hesaplanarak toplam hata içerisindeki kanal katkısı incelenir.

Temel yaklaşım:

Channel Error =
mean((input_channel - reconstruction_channel)²)

Daha yüksek kanal hatası, ilgili telemetri değişkeninin model tarafından normal davranıştan daha farklı şekilde yeniden oluşturulduğunu gösterir.

Bu değer doğrudan fiziksel arıza kanıtı olarak değerlendirilmemeli; anomaliyi yorumlamaya yardımcı olan bir ilişkilendirme göstergesi olarak kullanılmalıdır.

Kullanılan Model

Çalışmada daha önce eğitilmiş ve sabitlenmiş Autoencoder modeli kullanılmıştır.

Model yapısı:

384
 ↓
Dense(64, ReLU)
 ↓
Dense(16, ReLU)
 ↓
Dense(64, ReLU)
 ↓
Dense(384, Linear)

Girdi boyutu: 64 × 6 = 384
Latent boyutu: 16
Kayıp fonksiyonu: MSE

Model yalnızca normal çalışma verileri kullanılarak eğitilmiştir. Bu çalışma kapsamında model yeniden eğitilmemiş ve mevcut model değiştirilmemiştir.

Anomali Değerlendirme

Toplam pencere Reconstruction MSE değeri, mevcut Autoencoder threshold değeri ile karşılaştırılır.

Mevcut threshold:
τ = 0.025459141133630285

Threshold değeri normal doğrulama dağılımının P99.5 seviyesinden alınmış ve değiştirilmemiştir.

Alarm mekanizması EWMA ve son 5 pencerenin 3'ünün threshold üzerinde olması kuralını kullanır.

MSE
 ↓
EWMA
 ↓
Threshold τ
 ↓
3 of last 5 windows
 ↓
Alarm State
 ↓
Channel Attribution

Sonuçların Yorumlanması

Kanal attribution yaklaşımının temel amacı, anomali tespitinden sonra sistem davranışını daha anlaşılır hale getirmektir.

Örneğin voltage_in kanalının reconstruction error değeri diğer kanallara göre belirgin şekilde yüksekse, anomalinin giriş voltajındaki değişimle daha fazla ilişkili olduğu düşünülebilir.

Benzer şekilde yüksek sıcaklık hatası termal davranışla, yüksek fan RPM hatası ise soğutma davranışıyla ilişkili olabilir.

Bu yorumlar doğrudan fiziksel arıza teşhisi olarak değil, model çıktısını açıklamaya yardımcı olan göstergeler olarak değerlendirilmelidir.

Sistem İçindeki Konumu

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
Alarm
 ↓
Channel Attribution

Bu yapı sayesinde sistem yalnızca "Anomali var." demek yerine, anomalinin hangi telemetri kanallarıyla daha fazla ilişkili olabileceğini incelemeye yardımcı olur.

Doğrulama

Çalışma sonunda TypeScript kontrolü, Lint, Production Build, mevcut ML testleri ve kanal attribution testleri kontrol edilmiştir.

Deterministik veri akışı korunmuş ve mevcut Autoencoder modeli değiştirilmemiştir.

Kapsam ve Sınırlamalar

Channel Attribution sonuçları fiziksel sensör arızasının kesin teşhisi değildir. Sonuçlar reconstruction error değerlerine dayanır ve simüle edilmiş telemetri verileri üzerinde değerlendirilir.

Gerçek cihaz üzerindeki fiziksel ölçümlerle doğrulama yapılmadığından, attribution sonucu operatör veya mühendis incelemesini destekleyen bir açıklama mekanizması olarak değerlendirilmelidir.

Gün Sonu

Gün 23 kapsamında Autoencoder çıktılarının kanal bazında incelenmesi ve anomali sonuçlarının daha açıklanabilir hale getirilmesi üzerine çalışılmıştır.

Bu aşama ile birlikte anomali tespit sisteminin yalnızca alarm üretmesi değil, alarmın hangi telemetri değişkenleriyle ilişkili olabileceğini incelemeye yardımcı olması hedeflenmiştir.

Sonraki Adım

Bir sonraki aşamada alarm olaylarının kalıcı olarak saklanması ve geçmiş alarm kayıtlarının incelenebilmesi için alarm history ve veri tabanı yapısının geliştirilmesi planlanmaktadır.
