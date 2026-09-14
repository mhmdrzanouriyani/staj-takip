🚀 GÜN 26 — FİNAL DEĞERLENDİRME VE PERFORMANS ANALİZİ

🎯 Çalışmanın Amacı

Gün 26 kapsamında, proje boyunca geliştirilen anomali tespit sisteminin kapsamlı değerlendirmesi yapılmıştır.

Bu aşamada daha önce ayrı ayrı geliştirilen:

• Fixed Threshold yöntemi
• Autoencoder
• Reconstruction MSE
• EWMA
• 3-of-5 Alarm Policy

aynı değerlendirme çerçevesi içerisinde incelenmiştir.

Amaç yalnızca sistemin çalıştığını göstermek değil, elde edilen sonuçları ölçülebilir metrikler üzerinden değerlendirmek ve projenin güçlü ve sınırlı yönlerini ortaya koymaktır.


🏗️ Final Sistem Yapısı

Projenin final anomali tespit akışı:

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
TensorFlow.js Autoencoder
    ↓
Reconstruction MSE
    ↓
EWMA
    ↓
Frozen P99.5 Threshold
    ↓
3-of-5 Alarm Policy
    ↓
NORMAL / PENDING / ACTIVE


📐 Kullanılan Model

Autoencoder modeli normal çalışma verileri kullanılarak eğitilmiştir.

Model:

384
 ↓
Dense(64, ReLU)
 ↓
Dense(16, ReLU)
 ↓
Dense(64, ReLU)
 ↓
Dense(384, Linear)

Parametreler:

📡 Kanal sayısı: 6
🪟 Window: 64 frame
⏱️ Telemetri: 10 Hz
📦 Input size: 384
🔹 Latent size: 16
📉 Loss: MSE


📊 Değerlendirme Metrikleri

Final değerlendirmede aşağıdaki metrikler kullanılmıştır:

• Precision
• Recall
• F1 Score
• False Positive
• False Positive Rate
• Detected / Missed fault sayıları
• Alarm davranışı


1️⃣ Fixed Threshold Sonuçları

Daha önce geliştirilen sabit eşik yöntemi için elde edilen sonuçlar:

F1 → 86%
F2 → 100%
F3 → 100%
F4 → 0%
F5 → 27%

Overall Recall → 53.3%
Precision → 100%
F1 Score → 69.6%

Normal veri değerlendirmesinde False Positive görülmemiştir.

Ancak bu FPR sonucu bağımsız bir held-out normal test setinden elde edilmemiştir. Threshold değerleri aynı normal veri seti üzerinden oluşturulduğu için sonuç bu sınırlama ile değerlendirilmelidir.


2️⃣ Autoencoder — Reconstruction MSE

Frozen P99.5 threshold kullanılarak yapılan değerlendirmede:

Precision → 99.8%
Recall → 76.6%
F1 Score → 86.6%

Toplam:

Detected → 431
Missed → 132
False Alarm → 1


3️⃣ Autoencoder — EWMA + 3-of-5

Final alarm politikası ile:

Precision → 100%
Recall → 76.4%
F1 Score → 86.6%

Sonuç:

Detected → 430
Missed → 133
False Alarm → 0


⚖️ Yöntemlerin Karşılaştırılması

Fixed Threshold:

Precision → 100%
Recall → 53.3%
F1 → 69.6%

Autoencoder:

Precision → 99.8%
Recall → 76.6%
F1 → 86.6%

Autoencoder + EWMA + 3-of-5:

Precision → 100%
Recall → 76.4%
F1 → 86.6%


🔍 Sonuçların Yorumu

Mevcut değerlendirme protokolünde Autoencoder yaklaşımı, Fixed Threshold yöntemine göre daha yüksek Recall ve F1 Score üretmiştir.

Özellikle Recall değerindeki artış:

53.3%
   ↓
76.6%

şeklindedir.

Bu sonuç, Autoencoder'ın normal çalışma davranışından sapmaları yalnızca tek bir kanalın sabit sınırı üzerinden değil, 64 frame'lik çok kanallı pencere içerisindeki genel davranış üzerinden değerlendirebilmesinin faydalı olabileceğini göstermektedir.

Bununla birlikte Autoencoder'ın her fault tipi için aynı seviyede başarılı olduğu sonucuna varılmamıştır.


⚠️ Fault Window Sınırlaması

F1–F4 fault burst'lerinin tamamı 64 frame'den daha kısa olduğu için bu fault'lar için pure window-level değerlendirme yapmak mümkün değildir.

Burst süreleri:

F1 → 50 frame
F2 → 40 frame
F3 → 50 frame
F4 → 60 frame

Window size:

64 frame


Bu nedenle F1–F4 için doğrudan pure-window Precision / Recall / F1 sonucu çıkarılmamıştır.

Bu ayrım final değerlendirmede özellikle korunmuştur.


🧪 F5 Pure Window Değerlendirmesi

F5 fault için 37 adet pure window elde edilmiştir.

Sonuç:

Recall → 16.2%
Precision → 85.7%
F1 → 27.3%

Bu sonuç, Autoencoder performansının fault tipine göre değişebildiğini göstermektedir.

Dolayısıyla modelin yalnızca genel F1 değerine bakılarak değerlendirilmesi yerine fault-by-fault davranışının da incelenmesi gerekmektedir.


🎯 Threshold Kalibrasyonu

Autoencoder threshold değeri:

τ = 0.025459141133630285

Bu değer normal reconstruction MSE dağılımının P99.5 seviyesinden belirlenmiştir.

Normal calibration:

737 window

Threshold üzerinde:

4 window

Calibration FPR:

4 / 737 ≈ 0.543%


⚠️ Bu sonuç bağımsız held-out validation olarak değerlendirilmemelidir.

Threshold aynı normal window seti üzerinden kalibre edildiği için bu değer calibration-set FPR olarak raporlanmıştır.


🔄 EWMA ve Alarm Politikası

Final alarm politikası:

α = 0.2

EWMA:

EWMAₜ =
α × MSEₜ +
(1 − α) × EWMAₜ₋₁


Alarm kararı:

Son 5 pencerenin en az 3'ünde

EWMA > τ

olması durumunda:

🚨 ACTIVE


Ara durum:

EWMA > τ ancak alarm koşulu henüz tamamlanmadıysa:

🟡 PENDING


Normal durumda:

🟢 NORMAL


Bu yapı tek bir yüksek MSE değerinin doğrudan alarm üretmesini engelleyerek daha kararlı bir alarm davranışı sağlamaktadır.


📈 False Positive Analizi

MSE > τ yaklaşımında:

False Alarm → 1

EWMA + 3-of-5 yaklaşımında:

False Alarm → 0


Bu sonuç, persistence policy'nin tekil eşik aşımının alarm üretmesini azaltabildiğini göstermektedir.


💻 Sistem Performansı

Model performansı için Node CPU üzerinde ölçüm yapılmıştır.

Model Load:

23.18 ms

First Inference:

14.43 ms

Warm Inference:

P50 → 0.457 ms
P90 → 0.988 ms
P95 → 1.221 ms

Yaklaşık:

1686 inference / second


Memory gözleminde heap değişimi yaklaşık:

+7.7 MiB

olarak ölçülmüştür.


⚠️ Performans Ölçüm Sınırı

Bu ölçümler Node CPU ortamında gerçekleştirilmiştir.

Aşağıdaki browser-specific ölçümler bu aşamada yapılmamıştır:

❌ Browser WASM gerçek performansı
❌ Browser WebGL karşılaştırması
❌ Gerçek FPS etkisi
❌ Browser tab memory profili

Bu nedenle Node CPU sonuçları doğrudan gerçek browser performansı olarak sunulmamıştır.


🧠 Genel Teknik Değerlendirme

Final sistemde üç farklı katman birbirinden ayrılmıştır:

1️⃣ Detection

Autoencoder + Reconstruction MSE

2️⃣ Stabilization

EWMA

3️⃣ Alarm Confirmation

3-of-5


Bu ayrım sayesinde modelin ürettiği ham anomaly score ile kullanıcıya gösterilen alarm durumu birbirine karıştırılmamıştır.


🏭 Endüstriyel Kullanım Açısından

Sistem şu anda bir prototip seviyesindedir.

Mevcut yapı:

✅ Gerçek zamanlı telemetry
✅ WebSocket iletişimi
✅ 64 frame sliding window
✅ Frozen normalization
✅ Browser-side Autoencoder
✅ Reconstruction MSE
✅ Frozen threshold
✅ EWMA
✅ Alarm persistence
✅ AI Monitor
✅ 3D Viewer altyapısı


Ancak gerçek endüstriyel kullanım için henüz aşağıdaki çalışmalar gereklidir:

• Gerçek cihaz verileri
• Bağımsız held-out validation
• Daha fazla fault senaryosu
• Uzun süreli saha testi
• Browser performance profiling
• Alarm history
• Daha kapsamlı channel attribution
• Gerçek cihaz entegrasyonu


📌 Final Sonuç

Gün 26 sonunda proje kapsamında geliştirilen anomali tespit pipeline'ının ölçülebilir bir değerlendirmesi tamamlanmıştır.

Mevcut deneysel sonuçlarda:

Fixed Threshold
→ F1: 69.6%

Autoencoder
→ F1: 86.6%

Autoencoder + EWMA + 3-of-5
→ F1: 86.6%


Autoencoder yaklaşımı mevcut değerlendirme protokolünde daha yüksek Recall ve F1 göstermiştir.

Aynı zamanda değerlendirme sırasında modelin sınırlamaları da açık şekilde ortaya konmuştur.

Özellikle:

• kısa fault burst'leri
• F5 performansı
• calibration/held-out ayrımı
• browser performance ölçümlerinin eksikliği

final raporda dikkate alınması gereken noktalar olarak kaydedilmiştir.


🏁 Gün Sonu

Gün 26 kapsamında projenin final anomali tespit pipeline'ı kapsamlı şekilde değerlendirilmiştir.

Model sonuçları, alarm politikası, false positive davranışı ve performans ölçümleri birlikte incelenmiştir.

Bu çalışma ile projenin yalnızca çalışan bir prototip olmadığı, aynı zamanda ölçülebilir sonuçlar ve açıkça belirtilmiş sınırlamalar üzerinden değerlendirilebildiği gösterilmiştir.


➡️ SONRAKİ ADIM — GÜN 27

Bir sonraki aşamada sistem performansının daha detaylı incelenmesi, inference pipeline optimizasyonlarının değerlendirilmesi ve browser-side çalışma yapısının final hale getirilmesi planlanmaktadır.
