# Gün 14 — Günlük Çalışma Raporu

**Tarih:** 16 Ağustos 2026  
**Konu:** Sabit Eşik Baseline ve Alarm Mekanizması

---

## 1. Günün amacı

Bugün gelişmiş bir anomali modeli yazmadım. Amacım, Day 15 değerlendirmesinden önce anlaşılır ve deterministik bir **sabit eşik baseline** koymaktı.

Soru şu: “Bu telemetri değeri, normal çalışma aralığının içinde mi?”  
Soru şu değil: “Bu F1 mi, F3 mü?”

Bu yüzden detektör arıza kimliği döndürmüyor.

---

## 2. Yapılan çalışma

Mevcut Day 09 normal CSV’sini ve Day 10 istatistik yardımcılarını kullandım. Yeni bir Simulator, WebSocket sunucusu veya telemetri kaynağı açmadım.

Akış şöyle:

```text
day09-normal.csv
        ↓
min / max + hysteresis
        ↓
ThresholdDetector
        ↓
kanal durumu / alarm
```

Canlı hat aynı kaldı: WebSocket → TelemetryClient → ring buffer → Dashboard. Eşik katmanı bunun üzerine binen bir analiz.

---

## 3. Normal dataset’ten baseline oluşturulması

Baseline’ı yalnızca `data/day09-normal.csv` üzerinden hesapladım. Bu dosyadaki 800 satırın hepsi `normal` etiketli.

`createBaselineFromRows` yine de `label === "normal"` filtresi uyguluyor. Karışık bir dizi gelse bile arıza satırları min/max’a girmez.

Fault dosyasını (`day09-faults.csv`) öğrenme için kullanmadım. Onu sadece self-test’te değerlendirme girdisi olarak okudum.

---

## 4. Kanal bazlı min/max

Projede zaten altı kanal var; kullanıcı listesindeki beş isme yenilerini eklemedim. `temp_ambient` de aynı şemada olduğu için baseline’a girdi.

Her kanal için Day 10’daki `allChannelStatistics` ile gözlenen min ve max alındı. `CHANNEL_BOUNDS` (simülatör kelepçesi) baseline değil; onu ezmedim.

---

## 5. Hysteresis mekanizması

Eşiğin hemen altında/üstünde titremesin diye her kanala, gözlenen aralığın **%5**’i kadar hysteresis koydum:

```text
üst alarm:        value > max
üst toparlanma:   value < max − hysteresis

alt alarm:        value < min
alt toparlanma:   value > min + hysteresis
```

Örnek: max = 46, hysteresis = 1 → 47 alarm, 45.4 hâlâ alarm (`recovering`), 44.9 normal.

Karmaşık bir durum makinesi yazmadım. Kanal başına bir mandal var: `none` / `high` / `low`.

---

## 6. Threshold detector

Ana API `src/lib/threshold-detector.ts` içinde:

- `createBaselineFromNormalCsv(csv)`
- `createBaselineFromRows(rows)`
- `ThresholdDetector.evaluateFrame(frame)`
- `ThresholdDetector.evaluateRow(row)`
- `classifyFrame` / `classifyValue` (mandalsız, anlık kontrol)

Day 15 bunları doğrudan çağırabilir. Baseline nesnesi değerlendirme sırasında kopyalanır; fault satırları onu değiştirmez.

---

## 7. Alarm üretimi

Her kanal için yapı:

- `channel`, `value`, `state` (`NORMAL` | `ALARM`)
- `reason`: `within_baseline` | `above_max` | `below_min` | `recovering`
- `threshold`, `min`, `max`, `hysteresis`

`WARNING` eklemedim; şu anki iş için gerekmedi. F1–F5 kimliği yok.

Dashboard’da kartlara küçük bir Türkçe ipucu koydum: **Normal** / **Alarm · üst sınır** / **Alarm · alt sınır**. Mevcut İngilizce kanal adlarına dokunmadım.

---

## 8. F1–F5 ile uyumluluk

Detektör FaultEngine’i değiştirmiyor. F1–F5 satırlarını yalnızca mevcut `fault_id` ile okuyup ilgili kanalı değerlendiriyor.

Self-test’te F1 sıcaklık sıçramasının `temp_core` üst eşiğini, F2 gerilim çökmesinin `voltage_in` alt eşiğini **geçebildiği** görüldü. F3–F5 için zorunlu “hepsi yakalandı” şartı koymadım. Bu baseline’ın sınırını gizlemek istemedim.

---

## 9. Veri sızıntısının önlenmesi

Üç kontrol yaptım:

1. Baseline yalnızca normal etiketli satırlardan üretiliyor.
2. Tüm fault satırları değerlendirildikten sonra min/max/hysteresis aynı kalıyor.
3. Normal satırlara fault satırları eklenip tekrar öğrenilince sonuç değişmiyor.

Yani test kümesi eğitimi kirletmiyor.

---

## 10. Self-test

Komut: `npm run test:day14`

```text
Day 14 Threshold Baseline Self-Test
-----------------------------------
Baseline schema: PASS
Normal dataset: PASS

temp_core baseline: PASS
temp_ambient baseline: PASS
voltage_in baseline: PASS
current_draw baseline: PASS
fan_rpm baseline: PASS
cpu_load baseline: PASS

Min/max validation: PASS
Hysteresis: PASS
Normal classification: PASS
Upper threshold alarm: PASS
Lower threshold alarm: PASS
Hysteresis recovery: PASS
Determinism: PASS
Leakage protection: PASS

F1 evaluation: PASS
F2 evaluation: PASS
F3 evaluation: PASS
F4 evaluation: PASS
F5 evaluation: PASS

All Day 14 checks passed.
```

---

## 11. Validation

| Komut | Sonuç |
|-------|--------|
| `npx tsc --noEmit` | PASS |
| `npm run lint` | PASS |
| `npm run build` | PASS |
| `test:day08`–`test:day13` | PASS |
| `test:day14` | PASS |

Tarayıcıda ayrı bir görsel test bu oturumda yapılmadı.

---

## 12. Eklenen dosyalar

- `src/lib/threshold-detector.ts`
- `src/lib/threshold/day14SelfTest.ts`
- `scripts/run-day14-self-test.ts`
- `Gun_14/README.md`

---

## 13. Değiştirilen dosyalar

- `package.json` — `test:day14`
- `src/app/page.tsx` — normal CSV’den baseline okuma
- `src/components/dashboard/LiveTelemetryPanel.tsx` — küçük eşik ipucu
- `README.md` — Gün 14 satırı

Day 08–13 davranışına dokunulmadı. Dataset yeniden üretilmedi.

---

## 14. Sınırlamalar

Bu yöntem yalnızca “normal aralığın dışı”nı görür. Yavaş drift, aralık içinde kalan akım/fan değişimi veya korelasyon bozulması kaçabilir. F3, F4 ve F5 için bu beklenen bir zayıflık; Day 15 bunu sayıyla gösterecek.

Hysteresis oranı sabit (%5). Fault verisine bakarak ayarlamadım.

Dashboard ipucu anlık (mandalsız) sınıflandırma kullanıyor; mandallı `ThresholdDetector` değerlendirme API’sinde duruyor.

---

## 15. Sonuç

Bugün normal dataset’ten türetilmiş, açıklanabilir bir min/max + hysteresis baseline’ı ekledim. Self-test geçti, önceki günler bozulmadı. ML yok.

---

## 16. Day 15’e hazırlık

Day 15’in çağıracağı yüzey hazır:

```ts
const baseline = createBaselineFromNormalCsv(normalCsv);
const detector = new ThresholdDetector(baseline);
const result = detector.evaluateFrame(frame);
```

Yarın bu çıktı üzerinden F1–F5 için dürüst bir değerlendirme yazılabilir. Bugün o metrikleri şişirmedim.
Güncellenmiş metin burada: `Gun_14/Gun_14_18_GitHub.txt`

GitHub’a yapıştırmaya uygun, Gün 14–18 birleşik sürüm.

**Ne değişti**
- Üstte tek bir hero (badges + 14→19 ilerleme tablosu)
- İçindekiler kapanır menü
- Her gün kendi renkli başlığı ve özeti
- Gün 14 teknik hikâye aynı (min/max, %5 hysteresis, sızıntı)
- Gün 15–18 tam bölüm: metrikler, μ/σ, mimari, TF.js boyutu

**Kısa özet**

| Gün | Konu | Sonuç |
|:---:|------|--------|
| 14 | Fixed min/max + hysteresis | Detector hazır, fault ID yok |
| 15 | Frozen evaluation | F1 86% · F2/F3 100% · F4 **0%** · F5 27% · FPR 0% |
| 16 | Z-score + pencere 64 | 737 × 384, yalnızca normal |
| 17 | Keras `384→16→384` | val MSE ≈ 0.01711 |
| 18 | TF.js export | `public/models/ae-v1/` · **207.2 KB** |

Sonraki hedef metinde **Day 19** (tarayıcı `loadLayersModel`). Eski `Gun_14_GitHub_guncel.txt` duruyor; yapıştırmak için yeni dosyayı kullan.
