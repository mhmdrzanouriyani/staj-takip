🚀 GÜN 27 — PERFORMANS ÖLÇÜMÜ VE INFERENCE OPTİMİZASYONU

🎯 Çalışmanın Amacı

Gün 27 kapsamında, Autoencoder tabanlı anomali tespit sisteminin çalışma performansı detaylı şekilde incelenmiştir.

Gün 26'da modelin doğruluk ve alarm sonuçları değerlendirilirken, bu aşamada sistemin:

• Inference süresi
• Model yükleme süresi
• Bellek kullanımı
• Warm inference performansı
• Gerçek zamanlı çalışmaya uygunluğu
• Browser-side AI mimarisi

üzerinde durulmuştur.

Amaç, geliştirilen modelin yalnızca doğru sonuç üretmesini değil, aynı zamanda gerçek zamanlı telemetry akışı içerisinde yeterli performansla çalışabilmesini değerlendirmektir.


🏗️ Performans Açısından Sistem Yapısı

Final sistemde AI işlemleri server üzerinde sürekli çalışan bir AI servisine bağımlı değildir.

Temel yapı:

Telemetry
    ↓
WebSocket
    ↓
Ring Buffer
    ↓
64-Frame Window
    ↓
Normalization
    ↓
Autoencoder
    ↓
Reconstruction MSE
    ↓
EWMA
    ↓
Alarm Policy


Bu yapı sayesinde anomaly scoring işleminin browser tarafında gerçekleştirilmesi hedeflenmiştir.


🧠 Browser-Side AI

Autoencoder modeli TensorFlow.js formatına dönüştürülerek web uygulamasında kullanılabilir hale getirilmiştir.

Model:

384 → 64 → 16 → 64 → 384

Model dosyası:

📦 yaklaşık 212 KB

Bu boyut, browser tabanlı bir prototip için küçük ve taşınabilir bir model yapısı sağlamaktadır.


⚙️ Inference Backend

TensorFlow.js inference mimarisinde WASM backend desteklenmiştir.

Kullanılan yaklaşım:

Browser
   ↓
Anomaly Worker
   ↓
TensorFlow.js
   ↓
WASM Backend
   ↓
Autoencoder
   ↓
MSE


WASM tercihinin temel nedeni, model inference işleminin browser içerisinde GPU/WebGL'e bağımlı olmadan çalıştırılabilmesidir.

Burada Three.js WebGL ile TensorFlow.js inference backend birbirinden ayrı değerlendirilmiştir.

Three.js WebGL:

→ 3D Digital Twin rendering

TensorFlow.js:

→ AI inference


Bu iki teknoloji aynı browser içerisinde çalışsa da farklı görevleri yerine getirmektedir.


🧵 Web Worker Kullanımı

Inference işleminin ana browser thread'ini gereksiz şekilde meşgul etmemesi için Web Worker yapısı kullanılmıştır.

Temel yapı:

Main Thread
    ↓
Anomaly Worker
    ↓
TensorFlow.js
    ↓
Autoencoder


Worker'ın görevi:

• 384 elemanlı input almak
• Model inference gerçekleştirmek
• Reconstruction MSE hesaplamak
• Sonucu main thread'e göndermek


Bu yapı sayesinde AI hesaplamalarının UI rendering işlemlerinden ayrılması hedeflenmiştir.


📊 Performans Ölçümleri

Node CPU ortamında yapılan benchmark sonuçları:

Model Load:

23.18 ms


İlk Inference:

14.43 ms


Warm Inference:

P50 → 0.457 ms
P90 → 0.988 ms
P95 → 1.221 ms


Yaklaşık throughput:

≈ 1686 inference / second


İlk inference ile warm inference arasında belirgin fark bulunmaktadır.

Bunun temel nedeni model yükleme, runtime initialization ve ilk çalıştırma maliyetleridir.

Model bir kez hazırlandıktan sonra sonraki inference işlemleri çok daha kısa sürede tamamlanmıştır.


🔥 Warm Inference Nedir?

Model ilk kez çalıştırıldığında runtime initialization ve çeşitli hazırlık işlemleri gerçekleşebilir.

Bu nedenle:

First Inference
→ daha yüksek latency

Warm Inference
→ daha düşük latency


Gerçek zamanlı sistem açısından sürekli ilk inference maliyeti ödenmediği için warm inference performansı daha anlamlı bir ölçümdür.


💾 Bellek Kullanımı

Benchmark sırasında yaklaşık:

Heap Delta:

+7.7 MiB

RSS:

≈ 180 MiB


Bu değerler Node CPU benchmark ortamında gözlemlenmiştir.


⚠️ Browser Performans Sınırı

Bu ölçümlerin doğrudan gerçek browser performansı olarak yorumlanmaması gerekir.

Çünkü benchmark:

• Node CPU ortamında yapılmıştır.
• Gerçek browser WASM latency ölçülmemiştir.
• WebGL ile doğrudan benchmark yapılmamıştır.
• Gerçek browser FPS etkisi ölçülmemiştir.
• Browser tab memory profili çıkarılmamıştır.


Bu nedenle raporda bu sonuçlar:

"Node CPU benchmark"

olarak belirtilmiştir.


📈 SVG Chart Kullanımı

Dashboard içerisindeki grafikler bounded SVG yapısı kullanmaktadır.

Reconstruction MSE chart:

≈ 90 sample

Telemetry chart:

≈ 100 sample


Grafiklerde gösterilen veri miktarı sınırlandırıldığı için sürekli büyüyen bir DOM/SVG yapısı oluşturulmamaktadır.

Bu aşamada SVG'nin performans açısından gerçek bir problem oluşturduğunu gösteren ölçüm bulunmadığından Canvas'a geçiş yapılmamıştır.


🔄 Inference Cache

Modelin her telemetry frame'inde tekrar yüklenmesini engellemek için model loading işlemi cache edilmiştir.

Beklenen davranış:

İlk kullanım
↓
Model Load
↓
Cache
↓
Sonraki inference'lar
↓
Aynı model instance


Bu yapı gereksiz network ve initialization maliyetlerini azaltmaktadır.


🪟 Window İşleme

Model her tek telemetry sample'ını doğrudan değerlendirmemektedir.

Bunun yerine:

64 frame
×
6 channel
=
384 input value


oluşturulmaktadır.

10 Hz telemetry hızında:

64 / 10
≈
6.4 saniye


Bu nedenle model yaklaşık 6.4 saniyelik telemetry davranışını birlikte değerlendirmektedir.


🔐 Frozen Configuration

Performans ve deterministik davranış için aşağıdaki değerler runtime sırasında değiştirilmemektedir:

• Window = 64
• Channels = 6
• Input = 384
• Latent = 16
• Frozen normalization μ/σ
• Frozen threshold τ
• EWMA α = 0.2
• Alarm policy = 3-of-5


Bu yapı deneylerin tekrarlanabilir olmasını sağlamaktadır.


🧪 Test Sonuçları

Gün 27 kapsamında aşağıdaki kontroller gerçekleştirilmiştir:

✅ TypeScript check
✅ ESLint
✅ Production build
✅ Model loading
✅ Autoencoder inference
✅ Frozen normalization
✅ Reconstruction MSE
✅ Day14–22 tests
✅ Day26 validation
✅ Performance benchmark


Mevcut Node CPU benchmark başarılı şekilde tamamlanmıştır.


📌 Performans Sonucunun Yorumu

Ölçülen warm inference latency değerleri, modelin hesaplama açısından küçük bir yapıya sahip olduğunu göstermektedir.

P95:

≈ 1.221 ms


10 Hz telemetry akışında iki sample arasındaki süre:

100 ms


olduğundan, Node CPU benchmark sonucu açısından inference hesaplaması telemetry örnekleme aralığına göre oldukça kısa kalmaktadır.

Ancak bu sonuç gerçek browser performansının garantisi değildir.

Browser WASM performansının ayrıca ölçülmesi gerektiği açıkça belirtilmiştir.


🏭 Gerçek Zamanlı Sistem Açısından

Final sistemin performans hedefi:

Telemetry → Window → AI Inference → MSE → Alarm

işlemlerinin sürekli telemetry akışı altında UI'ı gereksiz şekilde bloklamamasıdır.

Web Worker kullanımı:

AI Calculation
      ↓
Background Worker

UI Rendering
      ↓
Main Thread


şeklinde ayrım sağlamaktadır.


⚠️ Mevcut Sınırlamalar

Gün 27 sonunda aşağıdaki noktalar açıkça kayıt altına alınmıştır:

❌ Gerçek browser WASM benchmarkı yapılmadı.

❌ WebGL ve WASM arasında ölçümlü bir hız karşılaştırması yapılmadı.

❌ Gerçek browser FPS etkisi ölçülmedi.

❌ Uzun süreli browser memory profiling yapılmadı.

❌ Gerçek endüstriyel cihaz üzerinde performans testi yapılmadı.


Bu nedenle ölçülmeyen değerler performans sonucu olarak raporlanmamıştır.


🧩 Teknik Kazanımlar

Gün 27 sonunda sistemde:

✅ TensorFlow.js browser inference
✅ WASM backend desteği
✅ Web Worker inference
✅ Model caching
✅ Bounded chart data
✅ Frozen model configuration
✅ Performance benchmark
✅ Memory observation
✅ Deterministic inference

yapısı oluşturulmuştur.


🏁 Gün Sonu

Gün 27 kapsamında Autoencoder tabanlı anomaly detection sisteminin performans karakteristiği incelenmiştir.

Node CPU benchmark sonucunda:

Load → 23.18 ms

First Inference → 14.43 ms

Warm P50 → 0.457 ms

Warm P95 → 1.221 ms

olarak ölçülmüştür.

Bu sonuçlar modelin hesaplama açısından hafif bir yapıya sahip olduğunu göstermektedir.

Aynı zamanda gerçek browser ortamında WASM, FPS ve memory ölçümlerinin henüz yapılmadığı açıkça kayıt altına alınmıştır.

Bu yaklaşım sayesinde proje içerisinde ölçülmüş sonuçlar ile henüz doğrulanmamış varsayımlar birbirinden ayrılmıştır.


➡️ SONRAKİ ADIM — GÜN 28

Bir sonraki aşamada proje dokümantasyonu, teknik kararlar, kurulum adımları, sistem mimarisi ve üçüncü taraf bir kullanıcının projeyi çalıştırabilmesi için gerekli README ve dokümantasyon yapısı finalize edilecektir.
