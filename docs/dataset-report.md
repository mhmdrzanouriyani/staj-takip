# 📊 SpikeEdge Telemetry — Dataset Validation Report

**Demo #2** · Gün 10 · Veri Seti Doğrulama  
**Seed:** 1337 · **Örnekleme:** 10 Hz · **İş yükü:** `variable`

Bu rapor Day 09 CSV dosyalarından **hesaplanan** istatistikleri içerir. Değerler elle yazılmamıştır. Anomali tespiti veya makine öğrenmesi bu aşamada uygulanmamıştır.

## 1. Amaç

Gün 10'un resmi hedefi:

> Veri setini doğrula: dağılım grafikleri, kanallar arası korelasyon matrisi → Demo #2

Gelecekteki anomali tespiti çalışmasından önce veri setinin temiz, etiketlerin tutarlı ve fiziksel ilişkilerin PlantModel ile uyumlu olup olmadığı doğrulanmalıdır.

Kritik hijyen kuralı:

```text
EĞİTİM  →  yalnızca normal telemetri
TEST    →  normal telemetri + arıza telemetrisi
```

Gün 10'da model eğitilmez. Autoencoder veya anomali detektörü eklenmez.

## 2. Veri Setleri

| Veri seti | Dosya | Rol | Satır | normal | fault |
|-----------|-------|-----|------:|-------:|------:|
| Normal | `data/day09-normal.csv` | Eğitim (yalnızca normal) | 800 | 800 | 0 |
| Arıza etiketli | `data/day09-faults.csv` | Test (karışık) | 800 | 500 | 300 |

Paylaşılan üretim parametreleri (Day 09 üreteci, mevcut Simulator):

- Seed: **1337**
- Örnekleme hızı: **10 Hz** (100 ms)
- İş yükü profili: **variable**
- Hedef satır sayısı: **800**
- Gürültü: Day 09 ile aynı seeded `NoisePipeline`

Normal dosya FaultEngine kapalı üretilmiştir. Arıza dosyası Day 08 `TELEMETRY_FAULTS` senaryosunu kullanır; pencereler arasında kalan kareler `normal` kalır.

## 3. Veri Kalitesi Kontrolleri

Şema, Day 09 dışa aktarıcısının sabit başlığıdır:

`timestamp,sequence,temp_core,temp_ambient,voltage_in,current_draw,fan_rpm,cpu_load,label,fault_id`

| Kontrol | Normal | Arıza dosyası |
|---------|--------|----------------|
| Gerekli sütunlar | PASS | PASS |
| Eksik / geçersiz sayı | 0 | 0 |
| NaN / Infinity | 0 | 0 |
| Boş satır | 0 | 0 |
| Yinelenen sequence | 0 | 0 |
| Geçersiz etiket | 0 | 0 |
| Geçersiz fault_id | 0 | 0 |
| CHANNEL_BOUNDS dışı | 0 | 0 |
| Sequence bütünlüğü (10 Hz) | PASS | PASS |

Eğitim/normal veri setinde arıza etiketi bulunmadı: **evet**.

## 4. İstatistiksel Dağılım

Aşağıdaki özet **eğitim/normal** veri setindendir (arıza pencereleri PlantModel ilişkilerini bozmasın diye).

| Kanal | Min | Max | Ortalama | Std. Sapma | Sample |
|---|---:|---:|---:|---:|---:|
| temp_core (Çekirdek Sıcaklığı) | 23.450 | 46.900 | 41.885 | 5.804 | 800 |
| temp_ambient (Ortam Sıcaklığı) | 23.450 | 23.750 | 23.587 | 0.072 | 800 |
| voltage_in (Giriş Gerilimi) | 12.070 | 12.270 | 12.157 | 0.061 | 800 |
| current_draw (Akım) | 0.800 | 1.310 | 1.066 | 0.174 | 800 |
| fan_rpm (Fan Devir) | 882.000 | 2516.000 | 1986.852 | 546.226 | 800 |
| cpu_load (CPU Yükü) | 26.200 | 63.300 | 46.413 | 14.157 | 800 |

### Histogram (10 eşit genişlikte kutu, kutu sayıları)

| Kanal | Kutu sayıları (düşük → yüksek) |
|-------|--------------------------------|
| temp_core | 22, 22, 22, 25, 26, 31, 38, 53, 237, 324 |
| temp_ambient | 7, 208, 0, 170, 0, 152, 143, 0, 111, 9 |
| voltage_in | 149, 191, 36, 43, 36, 28, 12, 117, 168, 20 |
| current_draw | 140, 148, 24, 18, 28, 32, 27, 87, 190, 106 |
| fan_rpm | 115, 19, 19, 23, 26, 30, 36, 45, 258, 229 |
| cpu_load | 160, 138, 18, 15, 16, 28, 29, 35, 143, 218 |

Dağılım SVG dosyaları: `docs/dataset/distribution-temperature.svg`, `distribution-power.svg`, `distribution-system.svg`.

## 5. Korelasyon Matrisi

Pearson çarpım-moment korelasyonu, **normal eğitim verisi** üzerinde, altı telemetri kanalı için hesaplanmıştır. Matris simetriktir; köşegen 1'dir.

| | temp_core | temp_ambient | voltage_in | current_draw | fan_rpm | cpu_load |
|---|---:|---:|---:|---:|---:|---:|
| temp_core | 1.0000 | 0.5866 | 0.3754 | -0.3379 | 0.9593 | -0.3796 |
| temp_ambient | 0.5866 | 1.0000 | 0.8454 | -0.8359 | 0.6771 | -0.8550 |
| voltage_in | 0.3754 | 0.8454 | 1.0000 | -0.9853 | 0.5153 | -0.9893 |
| current_draw | -0.3379 | -0.8359 | -0.9853 | 1.0000 | -0.4827 | 0.9961 |
| fan_rpm | 0.9593 | 0.6771 | 0.5153 | -0.4827 | 1.0000 | -0.5219 |
| cpu_load | -0.3796 | -0.8550 | -0.9893 | 0.9961 | -0.5219 | 1.0000 |

Isı haritası: `docs/dataset/correlation-matrix.svg`.

## 6. Fiziksel İlişkilerin Yorumu

Yorumlar PlantModel'in beklediği yönlerle **hesaplanan r** karşılaştırılarak yazılmıştır. Zayıf korelasyon abartılmamıştır.

- CPU Load ile Core Temperature arasında zayıf negatif korelasyon hesaplanmıştır (r = -0.3796); anlık Pearson katsayısı beklenen işaretle aynı değildir. PlantModel'de sıcaklık ve fan, CPU yükünü zaman sabitleriyle gecikmeli izler; 80 saniyelik pencerede anlık korelasyonun zayıf veya negatif çıkması bu gecikmeyle uyumlu olabilir, ancak tek başına nedensellik kanıtı değildir.
- CPU Load ile Current arasında güçlü pozitif korelasyon gözlemlenmiştir (r = 0.9961).
- CPU Load ile Fan RPM arasında orta negatif korelasyon hesaplanmıştır (r = -0.5219); anlık Pearson katsayısı beklenen işaretle aynı değildir. PlantModel'de sıcaklık ve fan, CPU yükünü zaman sabitleriyle gecikmeli izler; 80 saniyelik pencerede anlık korelasyonun zayıf veya negatif çıkması bu gecikmeyle uyumlu olabilir, ancak tek başına nedensellik kanıtı değildir.
- Current ile Voltage arasında güçlü negatif korelasyon gözlemlenmiştir (r = -0.9853).
- Core Temperature ile Fan RPM arasında güçlü pozitif korelasyon gözlemlenmiştir (r = 0.9593).

Not: `temp_ambient` yavaş bir oda döngüsüdür; 80 saniyelik pencerede diğer kanallarla güçlü bir bağ beklenmez. Termal kütle (`temp_core` zaman sabiti) CPU yükü değişimlerini geciktirir; bu da yük–sıcaklık korelasyonunu zayıflatabilir.

## 7. Fault Distribution

Arıza dosyası CSV'si taranmıştır. Var sayılmamıştır. Özet: F1 PASS, F2 PASS, F3 PASS, F4 PASS, F5 PASS.

| Fault ID | Fault Type | Samples/Events | Status |
|---|---|---:|---|
| F1-temperature-spike | temperature_spike (Sıcaklık sıçraması) | 50 | PASS |
| F2-voltage-sag | voltage_sag (Gerilim düşümü) | 40 | PASS |
| F3-current-surge | current_surge (Akım yükselişi) | 50 | PASS |
| F4-fan-degradation | fan_degradation (Fan bozulması) | 60 | PASS |
| F5-sensor-drift | sensor_drift (Sensör kayması) | 100 | PASS |

## 8. Data Leakage

- Normal veri setinde fault etiketi bulunmadığı doğrulandı: **evet**
- Datasetler arasında tespit edilen veri sızıntısı: **0**
- Aynı sequence’de, rampa başlangıcında sayısal olarak örtüşen arıza etiketli kare (F4/F5 intensity ≈ 0): **10**
- Sonuç: **PASS**

Her iki CSV aynı seed, startTimeMs ve sequence aralığını kullanır; sequence eşitliği tek başına sızıntı değildir. Şemada UUID yoktur. F4/F5 doğrusal rampanın ilk karelerinde intensity ≈ 0 olduğu için (ve nicemleme nedeniyle) aynı sequence’de kanal değerleri temiz koşu ile örtüşebilir. Bu, eğitim dosyasına arıza etiketi karışması değildir; etiket penceresi ile fiziksel etkinin başlangıcı arasındaki sınır etkisidir.

## 9. Sonuç

Day 10 doğrulaması: **PASS**.

Veri seti, ileride yalnızca normal davranış öğrenecek bir anomali tespiti çalışması için hazırlanmıştır. Eğitim kümesi yalnızca normal örnekler içerir; F1–F5 etiketleri test dosyasında mevcuttur; dağılım ve korelasyon özetleri üretilmiştir. Anomali tespiti henüz uygulanmamıştır.
