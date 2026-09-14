🚀 GÜN 25 — AUTOENCODER VE SABİT EŞİK YÖNTEMİNİN KARŞILAŞTIRILMASI

🎯 Çalışmanın Konusu

Gün 25 kapsamında, projede kullanılan yapay zekâ tabanlı Autoencoder yaklaşımı ile daha önce geliştirilen sabit eşik (Fixed Threshold) yönteminin karşılaştırılması üzerine çalışılmıştır.

Bu çalışmanın temel amacı, yalnızca bir model oluşturmak değil, Autoencoder kullanımının mevcut sabit eşik yöntemine göre hangi durumlarda avantaj sağladığını gerçek ölçümler üzerinden değerlendirmektir.

Temel soru:

❓ "Neden sadece sabit eşik kullanmak yerine Autoencoder kullanıyoruz?"


🔎 Karşılaştırılan Yöntemler

Çalışmada iki farklı anomali tespit yaklaşımı ele alınmıştır:

1️⃣ Fixed Threshold
2️⃣ Autoencoder + Reconstruction MSE

📌 Fixed Threshold yaklaşımında her telemetri kanalı için belirlenen normal çalışma aralıkları kullanılır.

🤖 Autoencoder yaklaşımında ise sistem normal çalışma davranışını öğrenir ve yeni verilerin bu davranıştan ne kadar uzak olduğunu Reconstruction MSE üzerinden değerlendirir.


📊 Fixed Threshold

Sabit eşik yöntemi daha önce normal telemetri verileri kullanılarak oluşturulmuştur.

Her kanal için belirlenen minimum ve maksimum değerlerin dışına çıkıldığında sistem anomalinin oluştuğunu değerlendirir.

Örneğin:

Voltage
Normal → 12V civarı
        ↓
Voltage düşer
        ↓
Threshold aşılır
        ↓
🚨 Alarm


🤖 Autoencoder Yaklaşımı

Autoencoder yalnızca normal çalışma verileri kullanılarak eğitilmiştir.

Model, normal telemetri davranışını öğrenerek giriş verisini yeniden oluşturmaya çalışır.

Normal davranış:
Input → Autoencoder → Benzer Reconstruction
→ düşük MSE

Anormal davranış:
Input → Autoencoder → Farklı Reconstruction
→ yüksek MSE

Bu nedenle Reconstruction MSE, sistemin anomali skorunu oluşturmak için kullanılır.


🧠 Model Yapısı

Kullanılan Autoencoder:

384
 ↓
Dense(64, ReLU)
 ↓
Dense(16, ReLU)
 ↓
Dense(64, ReLU)
 ↓
Dense(384, Linear)

📐 Window: 64 frame
📡 Kanal sayısı: 6
🔢 Input: 384 değer
🔹 Latent: 16
📉 Loss: MSE

Model daha önce eğitilmiş ve bu çalışma sırasında yeniden eğitilmemiştir.


🎯 Anomali Threshold

Autoencoder için kullanılan threshold normal doğrulama dağılımının P99.5 seviyesinden belirlenmiştir.

τ = 0.025459141133630285

Bu değer çalışma sırasında değiştirilmemiş ve fault verileri kullanılarak yeniden ayarlanmamıştır.


📈 Gerçek Ölçüm Sonuçları

Mevcut değerlendirme sonuçlarında:

Fixed Threshold:

• Precision: 100%
• Recall: 53.3%
• F1 Score: 69.6%

Autoencoder — MSE > τ:

• Precision: 99.8%
• Recall: 76.6%
• F1 Score: 86.6%

Autoencoder — EWMA + 3-of-5:

• Precision: 100%
• Recall: 76.4%
• F1 Score: 86.6%


🔍 Sonuçların Yorumu

Sonuçlara göre Autoencoder, mevcut değerlendirme protokolünde sabit eşik yöntemine kıyasla daha yüksek Recall ve F1 değerleri göstermiştir.

Bu durum Autoencoder'ın yalnızca tek tek kanal değerlerini değil, birden fazla telemetri kanalının birlikte oluşturduğu davranış modelini değerlendirebilmesinin önemli bir avantaj sağlayabileceğini göstermektedir.

Ancak bu sonuç:

"Autoencoder her durumda daha iyidir."

şeklinde yorumlanmamalıdır.

Örneğin bazı arıza tiplerinde sabit eşik yöntemi daha başarılı olabilir.


⚠️ F1–F5 Değerlendirme Sınırlaması

Autoencoder'ın 64 frame'lik pencere kullanması nedeniyle kısa süreli F1–F4 fault burst'leri temiz bir window-level değerlendirme için yeterli değildir.

F1–F4 burst süreleri 64 frame'den daha kısa olduğundan, bu arızalar için pure-window Precision / Recall / F1 sonuçları doğrudan güvenilir şekilde yorumlanamaz.

Bu nedenle:

❌ F4 için "Autoencoder arızayı tamamen çözüyor" şeklinde bir sonuç çıkarılmamıştır.

F5 için ise 37 adet pure window üzerinden ayrı değerlendirme yapılmıştır:

F5:
• Recall: 16.2%
• Precision: 85.7%
• F1: 27.3%

Bu sonuç, Autoencoder'ın her arıza tipinde aynı performansı göstermediğini açıkça göstermektedir.


⚖️ Neden İki Yöntemi Birlikte İnceliyoruz?

Fixed Threshold:

✅ Basit
✅ Anlaşılması kolay
✅ Belirli sınır ihlallerinde etkili
⚠️ Değer normal aralıkta olsa bile anormal ilişkiyi yakalayamayabilir

Autoencoder:

✅ Çok kanallı davranışı değerlendirebilir
✅ Normal davranıştan sapmayı reconstruction error ile ölçebilir
✅ Daha karmaşık davranış değişimlerini incelemek için kullanılabilir
⚠️ Daha karmaşık bir yapıya sahiptir
⚠️ Performans fault tipine ve değerlendirme protokolüne bağlıdır


🔄 Projedeki Konumu

İki yöntem aynı sistem içinde ayrı yaklaşımlar olarak değerlendirilmiştir:

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


Ayrı olarak:

Telemetry
    ↓
Fixed Threshold
    ↓
Hysteresis
    ↓
Baseline Alarm


🧪 Demo #5

Bu çalışma kapsamında Demo #5 için karşılaştırmalı bir gösterim hazırlanmıştır.

Demo sırasında:

1️⃣ Normal telemetri gösterilir.
2️⃣ Fault senaryosu başlatılır.
3️⃣ Fixed Threshold davranışı gözlemlenir.
4️⃣ Autoencoder Reconstruction MSE izlenir.
5️⃣ EWMA değeri takip edilir.
6️⃣ Threshold ile karşılaştırma yapılır.
7️⃣ Alarm durumunun nasıl oluştuğu gösterilir.
8️⃣ İki yöntemin sonuçları karşılaştırılır.


📌 Önemli Değerlendirme

Bu çalışmanın temel sonucu, yapay zekânın her durumda geleneksel yöntemin yerini alması gerektiği değildir.

Asıl amaç, hangi durumda hangi yöntemin daha anlamlı sonuç verdiğini ölçümler üzerinden ortaya koymaktır.

Bu nedenle proje kapsamında:

"AI kullanıldı."

demek yerine,

"AI yaklaşımının mevcut yönteme göre hangi ölçümlerde avantaj sağladığı ve hangi durumlarda sınırlı kaldığı test edildi."

yaklaşımı benimsenmiştir.


✅ Doğrulama

Çalışma kapsamında:

✔️ Fixed Threshold sonuçları kontrol edilmiştir.
✔️ Autoencoder sonuçları kontrol edilmiştir.
✔️ Precision / Recall / F1 değerleri incelenmiştir.
✔️ Fault senaryoları karşılaştırılmıştır.
✔️ Alarm davranışı test edilmiştir.
✔️ Mevcut model ve frozen threshold korunmuştur.
✔️ TypeScript kontrolü yapılmıştır.
✔️ Lint kontrolü yapılmıştır.
✔️ Production Build kontrol edilmiştir.


⚠️ Kapsam ve Sınırlamalar

Değerlendirmeler simüle edilmiş telemetri verileri üzerinden yapılmıştır.

Normal veri ile threshold kalibrasyonu ve bazı değerlendirme protokolleri bağımsız held-out test yapısına sahip değildir.

Ayrıca 64 frame'lik pencere nedeniyle kısa süreli fault senaryolarının window-level değerlendirilmesinde sınırlamalar bulunmaktadır.

Bu nedenle sonuçlar gerçek endüstriyel cihaz performansı olarak değil, proje prototipinin deneysel değerlendirmesi olarak ele alınmalıdır.


🏁 Gün Sonu

Gün 25 kapsamında Autoencoder tabanlı anomali tespit yaklaşımı ile sabit eşik yöntemi karşılaştırılmıştır.

Elde edilen sonuçlar, Autoencoder'ın mevcut değerlendirme protokolünde Recall ve F1 açısından avantaj sağlayabildiğini göstermiştir. Bununla birlikte, yöntemin her fault tipinde aynı performansı göstermediği ve özellikle kısa süreli arızalarda pencere tabanlı değerlendirmenin sınırlı olduğu görülmüştür.

Bu çalışma ile projenin önemli sorularından biri olan:

"Autoencoder neden kullanılmalı?"

sorusuna yalnızca teorik değil, ölçüme dayalı bir cevap oluşturulmuştur.


➡️ SONRAKİ ADIM — GÜN 26

Bir sonraki aşamada Autoencoder ve Fixed Threshold sonuçlarının daha kapsamlı deneysel değerlendirilmesi, metriklerin düzenli şekilde raporlanması ve final değerlendirme dokümantasyonunun hazırlanması planlanmaktadır.
