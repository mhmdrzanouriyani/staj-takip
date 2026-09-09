Gün 21 — Threshold Calibration / Eşik Kalibrasyonu

1. Günün Amacı

Day 21 kapsamında Autoencoder modelinin Reconstruction MSE çıktıları üzerinden veri odaklı ve sabit bir anomaly threshold (τ) belirlendi.

Amaç, normal çalışma verilerindeki reconstruction error dağılımını kullanarak güvenilir bir sınır oluşturmaktır.

Bu çalışma sırasında Autoencoder yeniden eğitilmedi ve threshold belirlemek için fault/anomaly verileri kullanılmadı.

Belirlenen threshold yöntemi:

τ = Normal Reconstruction MSE değerlerinin P99.5 (99.5. yüzdelik) değeri


2. Kullanılan Veri

Calibration işlemi Day 16'da oluşturulan frozen normal window dataset'i kullanılarak gerçekleştirildi.

Toplam:

737 normal window

Her window:

64 frame × 6 kanal = 384 değer

Kullanılan preprocessing Day 16 ile aynı tutuldu:

- Frozen μ (mean)
- Frozen σ (standard deviation)
- Aynı 6 kanal sırası
- 64-frame sliding window
- 384 elemanlı flattening
- Aynı frozen Autoencoder modeli

Fault dataset threshold seçimi sırasında kullanılmadı.


3. Autoencoder

Day 17'de eğitilen ve Day 18'de TensorFlow.js formatına export edilen mevcut Autoencoder kullanıldı.

Architecture:

384 → Dense(64, ReLU) → Dense(16, ReLU) → Dense(64, ReLU) → Dense(384, Linear)

Model değiştirilmedi ve yeniden eğitilmedi.

Day 21 yalnızca mevcut modelin normal veriler üzerindeki reconstruction error dağılımını analiz etti.


4. Reconstruction MSE

737 normal window için Autoencoder reconstruction MSE hesaplandı.

Elde edilen dağılım:

Minimum:
0.006776828366356903

Maximum:
0.026106650070300424

Mean:
0.01678161337683763

Median:
0.016389263470095495

Population Standard Deviation:
0.00470236551537309

P90:
0.023143280950906286

P95:
0.023833261652531912

P97.5:
0.02441142281680357

P99:
0.025032012648036152

P99.5:
0.025459141133630285


5. Ek İstatistikler

Mean + 1σ:
0.021483978892210723

Mean + 2σ:
0.02618634440758381

Mean + 3σ:
0.0308887099229569

Bu değerler yalnızca dağılımı incelemek ve threshold'un normal MSE dağılımındaki konumunu görmek amacıyla hesaplandı.


6. Frozen Threshold

Week 5 planına uygun olarak resmi threshold:

τ = P99.5

Sonuç:

τ = 0.025459141133630285

Threshold artık frozen olarak kabul edilmektedir.

Classification:

MSE <= τ
→ NORMAL

MSE > τ
→ ANOMALY

Threshold runtime'da tekrar hesaplanmaz.

Browser tarafı frozen threshold değerini kullanır.


7. Threshold Artifact

Frozen threshold aşağıdaki artifact içerisinde saklandı:

ml/day21-threshold.json

Runtime tarafında threshold:

src/lib/ml/frozenAeThreshold.ts

üzerinden kullanılmaktadır.

Browser P99.5 hesabını tekrar yapmaz ve live telemetry üzerinden threshold'u değiştirmez.


8. False Positive Rate

Threshold normal calibration windows üzerinde test edildi.

Threshold'un üzerinde kalan normal window sayısı:

4 / 737

False Positive Rate:

4 / 737 = 0.005427408412483039

Yaklaşık:

0.543%

Bu değer gerçek ölçüm sonucudur.

FPR değeri zorla %0.5 yapılmamıştır.

Threshold P99.5 olarak belirlendikten sonra ortaya çıkan gerçek calibration-set FPR raporlanmıştır.


9. Determinism / Tekrarlanabilirlik

Calibration işlemi iki kez çalıştırıldı.

İki çalıştırmada aşağıdaki değerlerin tamamı aynı çıktı:

- Mean
- Median
- Standard deviation
- P90
- P95
- P97.5
- P99
- P99.5
- Threshold τ
- FPR

Bu sonuç Day 21 calibration işleminin deterministic olduğunu doğrulamaktadır.


10. AI Monitor

AI Monitor paneli Day 21 kapsamında genişletildi.

Artık aşağıdaki değerler gösterilmektedir:

- Reconstruction MSE
- Frozen Threshold τ
- Detection Status

Classification:

MSE <= τ → NORMAL

MSE > τ → ANOMALY


11. Reconstruction MSE Chart

Mevcut Reconstruction MSE chart korunarak threshold reference line eklendi.

Chart üzerinde frozen τ için yatay reference line bulunmaktadır.

Bu sayede canlı MSE değerinin threshold'a göre konumu görsel olarak takip edilebilmektedir.


12. Day 14 ile İlişki

Day 14 detector değiştirilmedi.

Day 14:

Fixed per-channel min/max threshold + 5% hysteresis

olarak çalışmaya devam etmektedir.

Day 21 ise ayrı bir multivariate Autoencoder detection signal'i oluşturmaktadır.

Day 14 ve Day 21 bu aşamada birleştirilmemiştir.


13. Post-hoc Fault Validation

Fault verileri threshold seçimi sırasında kullanılmadı.

Threshold freeze edildikten sonra yalnızca post-hoc validation amacıyla fault windows incelendi.

F1–F4 fault burst'leri 64 frame'den kısa olduğu için pure window-level label oluşturulamadı.

F5 için:

37 pure windows

6 / 37 window threshold'u aştı.

Sonuç:

16.2%

Bu sonuç threshold tuning amacıyla kullanılmadı.

F4 için Autoencoder'ın problemi çözdüğü iddia edilmemektedir.

Day 15 baseline sonucunda F4 = 0% idi ve Day 21'de F4 için geçerli 64-frame pure window label bulunmamaktadır.


14. Validation

Aşağıdaki kontroller başarıyla tamamlandı:

npx tsc --noEmit
PASS

npm run lint
PASS

npm run build
PASS

npm run test:day14
PASS

npm run test:day15
PASS

npm run test:day16
PASS

npm run test:day17
PASS

npm run test:day18
PASS

npm run test:day19
PASS

npm run test:day20
PASS

npm run test:day21
PASS


15. Browser Validation Limitation

Live dashboard click-through testi bu ortamda gerçekleştirilemedi çünkü browser interaction tool bulunmamaktadır.

Buna rağmen:

- Threshold wiring
- Classification logic
- Frozen threshold behavior
- Day 21 self-test
- Production build

başarıyla doğrulanmıştır.


16. Files Created

src/lib/ml/day21/*

src/lib/ml/frozenAeThreshold.ts

src/lib/ml/day21SelfTest.ts

scripts/run-day21-self-test.ts

ml/day21-threshold.json

Gun_21/README.md


17. Files Updated

package.json
(test:day21 script)

src/lib/ml/constants.ts

live MSE snapshot

src/components/dashboard/AiMonitorPanel.tsx

src/components/dashboard/ReconstructionMseChart.tsx

README.md


18. Files / Systems Not Changed

ml/autoencoder.keras
→ unchanged

Day 16 μ/σ
→ unchanged

64-frame window contract
→ unchanged

Day 14 detector
→ unchanged

EWMA alarm policy
→ not implemented yet

M-consecutive alarm policy
→ not implemented yet

Channel attribution
→ not implemented yet

SQLite alarm history
→ not implemented yet


19. Day 21 Result

Day 21 successfully established a frozen Autoencoder anomaly boundary using only normal data.

Final threshold:

τ = 0.025459141133630285

Method:

P99.5

Normal calibration windows:

737

Measured FPR:

0.543%

The threshold is deterministic, frozen, and loaded at runtime without recalculation.

The AI Monitor now exposes the Reconstruction MSE, threshold, and NORMAL / ANOMALY state.


20. Preparation for Day 22

Day 22 will build the alarm stability layer on top of the frozen Day 21 threshold.

Planned:

Reconstruction MSE
→ EWMA smoothing
→ frozen τ comparison
→ M consecutive anomalous windows
→ Alarm decision

Day 22 should not recalibrate τ.

Day 22 should not retrain the Autoencoder.

Day 22 should not implement channel attribution or SQLite history.

Day 21 threshold remains:

τ = 0.025459141133630285


21. Summary

Day 21 completed the transition from raw Reconstruction MSE to a frozen anomaly decision boundary.

The system now has:

Normal telemetry
→ 64-frame window
→ frozen normalization
→ Autoencoder
→ Reconstruction MSE
→ frozen P99.5 threshold
→ NORMAL / ANOMALY

This provides the foundation for the Day 22 alarm stability logic.
