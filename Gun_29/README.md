🚀 GÜN 29 — AKADEMİK RAPORUN HAZIRLANMASI VE SONUÇLARIN DÜZENLENMESİ

🎯 Çalışmanın Amacı

Gün 29 kapsamında proje boyunca gerçekleştirilen teknik çalışmalar akademik rapor formatında düzenlenmeye başlanmıştır.

Bu aşamada amaç yalnızca yapılan işlemleri sıralamak değil, projenin:

• Problemini
• Tasarım yaklaşımını
• Sistem mimarisini
• Kullanılan yöntemleri
• Deneysel sonuçlarını
• Performansını
• Sınırlamalarını
• Elde edilen kazanımlarını

bir bütün halinde açıklamaktır.

Böylece proje geliştirme süreci ile akademik rapor arasında bağlantı kurulmuştur.


📚 Raporun Genel Yapısı

Final rapor için aşağıdaki bölüm yapısı temel alınmıştır:

1. Giriş
2. Problem Tanımı
3. Sistem Mimarisi
4. Gerçek Zamanlı Telemetry Sistemi
5. Digital Twin ve 3D Viewer
6. Anomali Tespit Yaklaşımı
7. Autoencoder Modeli
8. Threshold Calibration
9. Alarm Policy
10. Deneysel Değerlendirme
11. Performans Analizi
12. Sınırlamalar
13. Sonuç
14. Gelecek Çalışmalar


1️⃣ Giriş

Projenin temel amacı, endüstriyel bir cihazın gerçek zamanlı telemetry verilerinin izlenmesi ve normal çalışma davranışından sapmaların tespit edilmesidir.

Sistem yalnızca telemetry verilerini göstermekle sınırlı tutulmamış, aynı zamanda bu verilerin:

Telemetry
→ Window
→ Model
→ Anomaly Score
→ Alarm

akışı içerisinde değerlendirilmesi hedeflenmiştir.

Bunun yanında 3D Viewer ile cihazın dijital temsilinin oluşturulması amaçlanmıştır.


2️⃣ Problem Tanımı

Endüstriyel cihazlarda sıcaklık, voltaj, akım, fan hızı ve CPU yükü gibi parametreler sürekli değişmektedir.

Bu değerlerin tek tek sabit sınırlarla kontrol edilmesi bazı durumlarda yeterli olmayabilir.

Özellikle birden fazla kanalın birlikte oluşturduğu davranışın normalden sapması yalnızca tek bir kanalın threshold değerine bakılarak kolayca tespit edilemeyebilir.

Bu nedenle projede iki farklı yaklaşım incelenmiştir:

• Fixed Threshold
• Autoencoder-based Anomaly Detection


3️⃣ Sistem Mimarisi

Final sistem mimarisi raporda aşağıdaki şekilde açıklanmıştır:

Telemetry Simulator
        ↓
WebSocket Server
        ↓
Telemetry Client
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
P99.5 Threshold
        ↓
3-of-5 Alarm Policy
        ↓
Alarm State


3D Viewer ise telemetry verilerini görsel olarak temsil eden ayrı bir katman olarak değerlendirilmiştir.


4️⃣ Gerçek Zamanlı Telemetry

Sistem 10 Hz telemetry akışı üzerinden çalışmaktadır.

Kullanılan temel kanallar:

• temp_core
• temp_ambient
• voltage_in
• current_draw
• fan_rpm
• cpu_load


Telemetry akışı WebSocket üzerinden dashboard'a aktarılmış ve Ring Buffer içerisinde tutulmuştur.

Bu yapı sayesinde son telemetry değerleri gerçek zamanlı olarak izlenebilmiştir.


5️⃣ 64-Frame Window

Anomali tespit sisteminde tek bir sample yerine 64 frame'lik pencere kullanılmıştır.

Telemetry:

10 Hz

Window:

64 frame

Yaklaşık süre:

64 / 10 = 6.4 saniye


6 kanal kullanıldığı için model input boyutu:

64 × 6 = 384


Bu yaklaşım ile model yalnızca tek bir telemetry değerini değil, yaklaşık 6.4 saniyelik çok kanallı davranışı birlikte değerlendirmektedir.


6️⃣ Autoencoder Modeli

Autoencoder yalnızca normal çalışma verileri kullanılarak eğitilmiştir.

Model mimarisi:

384
 ↓
Dense(64, ReLU)
 ↓
Dense(16, ReLU)
 ↓
Dense(64, ReLU)
 ↓
Dense(384, Linear)


Modelin temel amacı normal çalışma davranışını öğrenmek ve giriş verisini yeniden oluşturmaktır.

Normal davranış iyi şekilde yeniden oluşturulduğunda Reconstruction Error düşük kalmaktadır.

Normal davranıştan sapma olduğunda Reconstruction Error artabilir.


7️⃣ Reconstruction MSE

Model çıktısı ile giriş arasındaki fark MSE kullanılarak hesaplanmıştır.

Temel süreç:

Input Window
     ↓
Autoencoder
     ↓
Reconstructed Window
     ↓
MSE
     ↓
Anomaly Score


Bu değer daha sonra alarm kararının oluşturulmasında kullanılmıştır.


8️⃣ Threshold Calibration

Threshold değeri normal reconstruction MSE dağılımı kullanılarak belirlenmiştir.

Kullanılan percentile:

P99.5


Frozen threshold:

τ = 0.025459141133630285


Calibration set:

737 normal windows


Threshold üzerinde kalan window:

4 / 737


Calibration FPR:

≈ 0.543%


Bu değer bağımsız held-out test sonucu olarak değerlendirilmemiştir.

Threshold aynı normal calibration seti üzerinden belirlendiği için bu durum raporda açıkça belirtilmiştir.


9️⃣ Alarm Policy

Ham MSE değerinin doğrudan alarm üretmesi yerine EWMA kullanılmıştır.

EWMA parametresi:

α = 0.2


Ardından persistence policy uygulanmıştır.

Final alarm kuralı:

Son 5 window'un en az 3'ünde

EWMA > τ


olması durumunda:

🔴 ACTIVE


Ara durum:

🟡 PENDING


Normal durum:

🟢 NORMAL


Bu yapı kısa süreli ve tekil threshold aşımının doğrudan kalıcı alarm oluşturmasını azaltmayı amaçlamaktadır.


📊 10️⃣ Deneysel Sonuçlar

Fixed Threshold:

Precision → 100%
Recall → 53.3%
F1 → 69.6%


Autoencoder — MSE > τ:

Precision → 99.8%
Recall → 76.6%
F1 → 86.6%


Autoencoder + EWMA + 3-of-5:

Precision → 100%
Recall → 76.4%
F1 → 86.6%


Bu sonuçlar, mevcut değerlendirme protokolünde Autoencoder yaklaşımının Fixed Threshold yöntemine göre daha yüksek Recall ve F1 Score verdiğini göstermektedir.


🔬 Fault Bazlı Değerlendirme

Fixed Threshold sonuçları:

F1 → 86%
F2 → 100%
F3 → 100%
F4 → 0%
F5 → 27%


Autoencoder tarafında F1–F4 fault burst'leri 64 frame'den kısa olduğu için pure window-level değerlendirme yapılamamıştır.

Bu nedenle bu fault'lar için doğrudan window-level başarı iddiasında bulunulmamıştır.


🧪 F5 Değerlendirmesi

F5 için:

Pure windows → 37

Recall → 16.2%
Precision → 85.7%
F1 → 27.3%


Bu sonuç, Autoencoder performansının fault tipine göre değişebildiğini göstermektedir.


⚠️ F4 Konusundaki Sınırlama

Özellikle F4 için Autoencoder'ın problemi tamamen çözdüğü şeklinde bir sonuç çıkarılmamıştır.

Önceki Fixed Threshold değerlendirmesinde:

F4 → 0%


olarak ölçülmüştür.

Ayrıca F4 fault burst'inin 64 frame'den kısa olması nedeniyle pure-window değerlendirmesi mümkün değildir.

Bu nedenle F4 için elde edilen sonuçlar dikkatli yorumlanmalıdır.


💻 11️⃣ Performans Analizi

Node CPU benchmark sonuçları:

Model Load:

23.18 ms


First Inference:

14.43 ms


Warm Inference:

P50 → 0.457 ms
P90 → 0.988 ms
P95 → 1.221 ms


Yaklaşık throughput:

≈ 1686 inference / second


Memory observation:

Heap Delta → +7.7 MiB

RSS → ≈ 180 MiB


Bu sonuçlar Node CPU ortamındaki benchmark değerleridir.


⚠️ Browser Performans Notu

Gerçek browser ortamında:

• WASM latency
• WebGL latency
• FPS etkisi
• Browser memory

ayrıntılı olarak ölçülmemiştir.

Bu nedenle Node benchmark sonuçları gerçek browser performansı olarak sunulmamıştır.


🧩 12️⃣ Teknik Kararların Özeti

Projede alınan temel teknik kararlar:

TensorFlow.js
→ Browser-side inference


WASM
→ TensorFlow.js inference backend


Web Worker
→ AI hesaplamalarını main thread'den ayırmak


Three.js WebGL
→ 3D rendering


Bounded SVG
→ Sınırlı sayıda telemetry ve MSE sample'ının görselleştirilmesi


Frozen normalization
→ Deterministic inference


Frozen P99.5 threshold
→ Reproducible alarm boundary


EWMA + 3-of-5
→ Daha kararlı alarm davranışı


📦 13️⃣ Reproducibility

Deneylerin tekrar edilebilir olması için önemli parametreler sabitlenmiştir.

Bunlar:

• Window size
• Channel order
• Normalization statistics
• Model
• Threshold
• EWMA α
• Alarm policy


Bu değerlerin korunması, farklı çalıştırmalarda aynı sistem davranışının elde edilmesine yardımcı olmaktadır.


📁 14️⃣ Deney Kayıtları

Deney sonuçları ayrıca evaluation klasöründe tutulmuştur.

Örnek:

docs/eval/
└── ae_v1_w64_b16_20260914/
    └── NOTES.md


Burada:

• run_id
• detector
• model
• test set
• threshold
• persistence
• result
• fault-by-fault sonuçları
• performance
• observations

gibi bilgiler kayıt altına alınmıştır.


🎓 Akademik Katkı

Projenin akademik açıdan temel katkısı, yalnızca bir anomaly detection modeli oluşturmak değildir.

Aynı zamanda:

Fixed Threshold
        vs
Autoencoder

yaklaşımlarının aynı telemetry sistemi içerisinde karşılaştırılmasıdır.

Böylece model seçimi yalnızca teorik avantajlara göre değil, ölçülen Precision, Recall, F1 ve false alarm davranışlarına göre değerlendirilmiştir.


📌 Bulguların Yorumu

Mevcut deneylerde Autoencoder:

• Daha yüksek Recall
• Daha yüksek F1
• Çok düşük false alarm

sonuçları üretmiştir.

Ancak bu sonuçların tüm fault türlerine genellenemeyeceği belirtilmiştir.

Özellikle:

• Short fault burst
• F5 performansı
• Calibration / held-out ayrımı
• Browser benchmark eksikliği

raporun sınırlamalar bölümünde açıkça yer almıştır.


🔭 Gelecek Çalışmalar

Sistemin ileride geliştirilmesi için aşağıdaki çalışmalar önerilmiştir:

• Gerçek endüstriyel cihaz verilerinin kullanılması
• Bağımsız held-out normal test seti oluşturulması
• Daha uzun fault senaryoları
• Daha fazla fault tipi
• Browser WASM benchmark
• Gerçek browser memory profiling
• Uzun süreli saha testi
• Gelişmiş channel attribution
• Alarm history
• Gerçek cihaz entegrasyonu


🏁 Gün Sonu

Gün 29 kapsamında proje boyunca elde edilen teknik çıktılar akademik rapor yapısına aktarılmıştır.

Sistem mimarisi, telemetry pipeline, Autoencoder, threshold calibration, alarm policy, deney sonuçları ve performans ölçümleri tek bir rapor yapısı altında birleştirilmiştir.

Ayrıca ölçülen sonuçlar ile henüz doğrulanmamış varsayımlar birbirinden ayrılmıştır.

Bu çalışma sayesinde proje final sunumu ve akademik değerlendirme için daha düzenli, savunulabilir ve takip edilebilir bir dokümantasyon yapısına kavuşmuştur.


➡️ SONRAKİ ADIM — GÜN 30

Son aşamada final proje kontrolü, teslimat hazırlığı, repository temizliği, çalıştırma testi, final demo senaryosu ve proje handoff dokümanları tamamlanacaktır.
