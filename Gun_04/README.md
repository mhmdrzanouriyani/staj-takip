# Gün 04 - Günlük Çalışma Raporu

**Tarih:** 6 Ağustos 2026  
**Konu:** Plant Model Geliştirilmesi, Simülasyon Testleri ve İlk Proje Güncellemesi

---

# 📋 Günün Özeti

Bugün deterministik telemetry simülatörünün en önemli bileşenlerinden biri olan Plant Model geliştirildi. Sistemin fiziksel davranışını temsil edecek matematiksel model oluşturularak sıcaklık, akım, voltaj, fan devri ve CPU yükü arasındaki ilişkiler tanımlandı.

Model tamamlandıktan sonra uzun süreli simülasyon testleri gerçekleştirildi ve üretilen verilerin beklenen aralıklar içerisinde olduğu doğrulandı. Ayrıca proje derlenerek lint ve TypeScript kontrolleri başarıyla tamamlandı. Gün sonunda proje GitHub deposuna güncellenerek yeni geliştirmeler yayınlandı.

---

# 🛠 Yapılan Çalışmalar

## 🌡️ 1. Plant Model Geliştirilmesi

Simulator içerisinde kullanılacak fiziksel sistem modeli geliştirildi.

Bu kapsamda;

- CPU Load davranışı oluşturuldu.
- Güç tüketimi modeli tanımlandı.
- Voltaj düşümü hesaplandı.
- Akım hesaplama modeli geliştirildi.
- Fan kontrol algoritması oluşturuldu.
- Core ve Ambient sıcaklık ilişkisi modellendi.
- Termal denge hesaplamaları eklendi.

Böylece sistem gerçek donanım davranışına daha yakın telemetry verileri üretmeye başladı.

---

## ⚙️ 2. Matematiksel Modelleme

Simülatörde kullanılan temel hesaplama modelleri oluşturuldu.

Bu süreçte;

- Thermal Resistance modeli
- Fan eğrisi
- Constant Power yaklaşımı
- Voltage Drop hesaplaması
- Exponential smoothing yöntemi

uygulanarak sistemin daha stabil çalışması sağlandı.

---

## 🧪 3. Simülasyon Testleri

Geliştirilen Plant Model farklı senaryolar altında test edildi.

Yapılan kontroller;

- Uzun süreli çalışma testi
- Kararlı durum analizi
- Sıcaklık değişim testi
- Voltaj değişimi kontrolü
- Fan davranışı doğrulaması
- Kanal limit kontrolleri

gerçekleştirildi.

Üretilen telemetry değerlerinin beklenen çalışma aralıklarında olduğu doğrulandı.

---

## ✅ 4. Kod Doğrulama

Geliştirme tamamlandıktan sonra proje yeniden doğrulandı.

Çalıştırılan komutlar;

- npm run build
- npm run lint
- npx tsc --noEmit
- npm run dev

Tüm kontroller hata vermeden başarıyla tamamlandı.

---

## ☁️ 5. Git ve GitHub Güncellemesi

Yapılan geliştirmeler sürüm kontrol sistemine aktarıldı.

Bu kapsamda;

- Yeni dosyalar commit edildi.
- Simulator güncellemeleri eklendi.
- Dashboard mimarisi güncellendi.
- README düzenlemeleri yapıldı.
- Proje GitHub deposuna başarıyla push edildi.

---

# 📚 Gün Sonunda Öğrenilenler

Bugün sonunda;

- Fizik tabanlı telemetry üretim modeli geliştirildi.
- Deterministik simülasyon mantığı daha iyi anlaşıldı.
- Gerçek zamanlı telemetry üretim süreci test edildi.
- Yazılan kodların doğrulama süreçleri uygulandı.
- Git tabanlı sürüm yönetimi pratiği geliştirildi.
- Proje ilk önemli geliştirme aşamasını tamamladı.

---

# 💻 Kullanılan Teknolojiler

- Next.js 14
- React 18
- TypeScript
- Tailwind CSS
- Git & GitHub
- Node.js

---

# 📌 Sonuç

Dördüncü günün sonunda simülatörün temelini oluşturan Plant Model başarıyla geliştirildi ve sistemin ürettiği telemetry verileri doğrulandı. Proje derleme, lint ve tip kontrollerinden sorunsuz geçti. Son olarak tüm geliştirmeler GitHub deposuna aktarılmış ve proje bir sonraki geliştirme aşamasına hazır hale getirilmiştir.
