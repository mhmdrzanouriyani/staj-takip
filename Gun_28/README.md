🚀 GÜN 28 — PROJE DOKÜMANTASYONU VE TESLİMATA HAZIRLIK

🎯 Çalışmanın Amacı

Gün 28 kapsamında, proje boyunca geliştirilen sistemin teknik dokümantasyonu düzenlenmiş ve final teslimata uygun hale getirilmiştir.

Bu aşamadaki temel amaç:

• Projenin nasıl çalıştığını açıklamak
• Kurulum adımlarını standartlaştırmak
• Sistem mimarisini belgelemek
• AI pipeline'ını açıklamak
• Teknik kararların nedenlerini kayıt altına almak
• Yeni bir kullanıcının projeyi kendi ortamında çalıştırabilmesini sağlamak
• Final rapor ve sunum çalışmalarına temel oluşturmak

olmuştur.


📚 Dokümantasyon Yapısı

Proje dokümantasyonu aşağıdaki ana bölümlere ayrılmıştır:

📁 docs/
│
├── technical-decisions.md
│
├── report/
│   ├── README.md
│   ├── 01-...
│   ├── 02-...
│   ├── 03-...
│   └── ...
│
└── eval/
    └── <run_id>/
        └── NOTES.md


📖 Ana README

README içerisinde projenin temel kullanım bilgileri açıklanmıştır.

Dokümante edilen başlıklar:

• Projenin amacı
• Sistem mimarisi
• Kullanılan teknolojiler
• Kurulum
• Çalıştırma
• Telemetry server
• Dashboard
• AI Monitor
• 3D Viewer
• Model dosyaları
• Test komutları
• Proje yapısı


⚙️ Kurulum Akışı

Üçüncü taraf bir kullanıcının projeyi çalıştırabilmesi için temel kurulum akışı standartlaştırılmıştır.

İlk adım:

npm install


Telemetry server:

npm run server:telemetry


Development server:

npm run dev


Daha sonra browser üzerinden dashboard açılarak sistem kontrol edilebilir.

Bu yapı sayesinde proje yalnızca geliştiricinin kendi bilgisayarında çalışan bir yapı olmaktan çıkarılarak tekrar kurulabilir bir prototip haline getirilmiştir.


🏗️ Sistem Mimarisi Dokümantasyonu

Final sistem mimarisi:

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
AI Monitor


Ayrıca 3D tarafı ayrı bir görselleştirme katmanı olarak ele alınmıştır.

Telemetry
   ↓
Digital Twin / 3D Viewer

AI pipeline ve 3D rendering birbirinden bağımsız görevler olarak dokümante edilmiştir.


🧠 AI Pipeline Dokümantasyonu

Autoencoder sisteminin temel parametreleri kayıt altına alınmıştır:

Window:

64 frame

Telemetry:

10 Hz

Window duration:

≈ 6.4 saniye

Channel:

6

Input:

384

Latent:

16

Loss:

MSE


Model:

384
 ↓
64
 ↓
16
 ↓
64
 ↓
384


Model yalnızca normal çalışma verileri kullanılarak eğitilmiştir.


📊 Anomaly Score

Anomaly score üretim süreci:

64-frame telemetry window
        ↓
Normalization
        ↓
Autoencoder Reconstruction
        ↓
MSE
        ↓
EWMA


Final threshold:

τ = 0.025459141133630285

Threshold, normal reconstruction MSE dağılımındaki P99.5 seviyesinden belirlenmiştir.


🚨 Alarm Policy

Final alarm mekanizması:

EWMA > τ

koşulunun son 5 window içerisinde en az 3 kez gerçekleşmesine dayanmaktadır.

Durumlar:

🟢 NORMAL
🟡 PENDING
🔴 ACTIVE


Bu yapı ile tek bir anlık threshold aşımının doğrudan alarm oluşturması engellenmiştir.


⚖️ Teknik Kararların Dokümantasyonu

Gün 28 kapsamında önemli teknik kararlar ayrıca belgelenmiştir.

Örneğin:

TensorFlow.js
→ Browser-side inference

WASM
→ TensorFlow.js inference backend

Web Worker
→ AI hesaplamalarının UI thread'inden ayrılması

Three.js WebGL
→ 3D Digital Twin rendering

Bounded SVG
→ Dashboard grafiklerinde sınırlı veri gösterimi


Bu kararların yalnızca ne olduğu değil, proje içerisindeki görevleri de açıklanmıştır.


🧵 Web Worker Dokümantasyonu

Inference Worker'ın görevi:

• Input window almak
• Model inference gerçekleştirmek
• Reconstruction MSE hesaplamak
• Sonucu main thread'e göndermek


Main thread ise:

• EWMA
• Alarm policy
• UI state
• Dashboard rendering

işlemlerini yönetmektedir.

Böylece AI hesaplama ve UI işlemleri arasında daha net bir sorumluluk ayrımı oluşturulmuştur.


📁 Model Dosyalarının Dokümantasyonu

TensorFlow.js modeli:

public/models/ae-v1/


İçeriği:

model.json
group1-shard1of1.bin


Model boyutu yaklaşık:

212 KB


Model dosyaları projenin browser-side inference yapısının temel bileşenidir.


🧪 Test ve Validation Dokümantasyonu

Projede kullanılan test komutları ve validation süreçleri README içerisinde açıklanmıştır.

Örnek:

npm run test:day21
npm run test:day22
npm run test:day26
npm run test:day27


Ayrıca model, threshold ve alarm policy gibi önemli aşamalar için validation çıktıları ayrı dosyalarda tutulmuştur.


📝 Experiment Notes

Deney sonuçlarının kaybolmaması için evaluation klasör yapısı kullanılmıştır.

Örnek:

docs/eval/
└── ae_v1_w64_b16_20260914/
    └── NOTES.md


Experiment notes içerisinde:

• run_id
• detector
• model
• test set
• threshold
• persistence policy
• result
• fault-by-fault evaluation
• performance
• observations

gibi bilgiler kayıt altına alınmıştır.


🔐 Reproducibility

Projenin tekrar üretilebilir olması için önemli parametreler frozen olarak tutulmuştur.

Bunlar:

• Normalization μ/σ
• Autoencoder model
• Window size
• Channel order
• Threshold
• EWMA α
• Alarm policy


Bu değerlerin runtime sırasında değiştirilmemesi, farklı testlerde daha tutarlı sonuçlar elde edilmesini sağlamaktadır.


📦 Repository Düzeni

Projenin ana klasör yapısı da daha anlaşılır hale getirilmiştir.

Örnek:

src/
├── components/
├── lib/
├── workers/
│   └── anomaly.worker.ts
│
config/
│   └── device-map.ts
│
server/
│   └── telemetry-ws.js
│
sim/
│
ml/
│
public/
│   └── models/
│       └── ae-v1/
│
docs/


Bu yapı sayesinde frontend, worker, telemetry server, simulation, machine learning ve documentation bölümleri birbirinden ayrılmıştır.


🧹 Gereksiz Dosyaların Kontrolü

Repository içerisinde gereksiz veya ağır dosyaların bulunmaması için kontrol yapılmıştır.

Özellikle:

❌ node_modules
❌ .next
❌ Gereksiz büyük binary dosyaları
❌ Ağır .blend dosyaları
❌ Gereksiz raw dataset kopyaları

repository dışında tutulacak şekilde düzenlenmiştir.


📐 Model Dosyası Kontrolü

Autoencoder modelinin toplam boyutu:

≈ 212 KB

Bu boyut, proje için belirlenen küçük ve browser-friendly model hedefiyle uyumludur.


🔍 Final Kullanıcı Akışı

Dokümantasyonda kullanıcı açısından temel çalışma akışı da açıklanmıştır:

1️⃣ Projeyi kur

npm install

2️⃣ Telemetry server'ı başlat

npm run server:telemetry

3️⃣ Dashboard'u başlat

npm run dev

4️⃣ Dashboard'a bağlan

5️⃣ Telemetry akışını kontrol et

6️⃣ 3D Viewer'ı aç

7️⃣ AI Monitor'ü kontrol et

8️⃣ MSE / EWMA / Threshold değerlerini gözlemle

9️⃣ Alarm durumunu incele


📊 Final Dashboard'da Görülebilen Bilgiler

AI Monitor içerisinde temel olarak:

• Reconstruction MSE
• EWMA
• Threshold
• Alarm state
• Window bilgisi
• Model bilgisi
• Inference durumu

gibi bilgiler gösterilmektedir.

Bu bilgiler sayesinde modelin karar süreci kullanıcı tarafından daha anlaşılır hale getirilmiştir.


⚠️ Dokümante Edilen Sınırlamalar

Final dokümantasyonda sistemin mevcut sınırları da açık şekilde belirtilmiştir.

Bunlar:

• Sistem simülasyon verisi kullanmaktadır.
• Gerçek endüstriyel cihaz entegrasyonu yapılmamıştır.
• F1–F4 fault'ları için pure 64-frame window bulunmamaktadır.
• Threshold calibration bağımsız held-out normal set değildir.
• Browser WASM performansı henüz ölçülmemiştir.
• Gerçek saha koşullarında uzun süreli test yapılmamıştır.


Bu sınırlamalar özellikle gizlenmemiş ve final rapor içerisinde açık şekilde belirtilmiştir.


🎓 Akademik Rapor İçin Hazırlık

Gün 28 kapsamında oluşturulan dokümantasyon, akademik rapor için temel oluşturacak şekilde düzenlenmiştir.

Raporda kullanılabilecek ana bölümler:

1. Introduction
2. Problem Definition
3. System Architecture
4. Telemetry Pipeline
5. Digital Twin
6. Anomaly Detection
7. Autoencoder Model
8. Threshold Calibration
9. Alarm Policy
10. Performance Evaluation
11. Limitations
12. Conclusion


📌 Gün Sonu Sonucu

Gün 28 sonunda proje dokümantasyonu önemli ölçüde finalize edilmiştir.

Artık proje için:

✅ Kurulum dokümantasyonu
✅ Çalıştırma adımları
✅ Sistem mimarisi
✅ AI pipeline açıklaması
✅ Teknik kararlar
✅ Model bilgileri
✅ Threshold bilgileri
✅ Alarm policy
✅ Test yapısı
✅ Evaluation notes
✅ Proje klasör yapısı
✅ Bilinen sınırlamalar

kayıt altına alınmıştır.


🏁 Genel Değerlendirme

Gün 28'in temel çıktısı yeni bir model geliştirmek değil, mevcut sistemin anlaşılabilir, tekrar kurulabilir ve teslim edilebilir hale getirilmesidir.

İyi bir teknik proje yalnızca çalışan koddan oluşmadığı için, geliştirilen sistemin nasıl kurulacağı, nasıl çalıştığı, hangi kararların neden alındığı ve hangi sınırlamalara sahip olduğu da açık şekilde belgelenmiştir.

Bu dokümantasyon final teslimat, akademik rapor ve proje sunumu için temel kaynak olarak kullanılacaktır.


➡️ SONRAKİ ADIM — GÜN 29

Bir sonraki aşamada akademik raporun hazırlanması, deney sonuçlarının rapor formatına aktarılması ve proje çıktılarının akademik bir çalışma formatında düzenlenmesi planlanmaktadır.
