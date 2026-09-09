# Gün 22 — Alarm Policy: EWMA + M-Consecutive Windows

## 1. Günün Amacı

Day 22 kapsamında mevcut canlı Reconstruction MSE pipeline üzerine alarm kararlılığı katmanı eklendi.

Amaç, tek seferlik veya kısa süreli MSE yükselmelerinin doğrudan alarm oluşturmasını önlemektir.

Pipeline:

Reconstruction MSE
→ EWMA smoothing
→ Frozen τ comparison
→ M consecutive anomalous windows
→ Alarm State

## 2. Day 21 Threshold

Day 21'de belirlenen threshold değiştirilmedi.

**τ = 0.025459141133630285**

Method: P99.5 of reconstruction MSE over 737 frozen normal windows.

Artifact: `ml/day21-threshold.json`

Runtime: `src/lib/ml/frozenAeThreshold.ts`

Day 21 classification remains:

- `MSE <= τ` → NORMAL
- `MSE > τ` → ANOMALY

## 3. EWMA Smoothing

Day 22'de Reconstruction MSE'nin daha kararlı değerlendirilmesi için EWMA uygulandı.

**α = 0.2**

Constant: `EWMA_ALPHA`

Formula:

`EWMA_t = α × MSE_t + (1 − α) × EWMA_(t−1)`

Initialization:

`EWMA_0 = MSE_0`

Alpha, mevcut 10 Hz telemetry stream yapısı ve Week 5 planındaki 0.2 aday değeri temel alınarak seçildi. F1–F5 sonuçlarıyla tune edilmedi.

## 4. Alarm Confirmation Window

Tek bir anomalous window sonucunda alarmın aktifleşmesini önlemek için consecutive-window confirmation rule eklendi.

**M = 3**

Constant: `ALARM_CONFIRMATION_WINDOWS`

10 Hz akışta M=3 yaklaşık 300 ms ardışık confirmation anlamına gelir.

Kural:

- `EWMA > τ` → anomaly streak artırılır.
- `EWMA <= τ` → streak sıfırlanır.
- Streak `>= 3` olduğunda alarm ACTIVE olur.

## 5. Alarm State Machine

Üç durum kullanılır:

- **NORMAL**
- **CANDIDATE**
- **ALARM**

| Condition | State | UI |
|---|---|---|
| EWMA ≤ τ | NORMAL, streak = 0 | NORMAL |
| EWMA > τ and streak < 3 | CANDIDATE | PENDING |
| EWMA > τ and streak ≥ 3 | ALARM | ACTIVE |

Örnek:

1. anomalous window → CANDIDATE, streak 1/3  
2. anomalous window → CANDIDATE, streak 2/3  
3. anomalous window → ALARM, streak 3/3

## 6. Recovery

`EWMA <= τ` olduğunda:

- anomaly streak = 0
- state = NORMAL
- active alarm kapanır

Day 22'de ekstra hysteresis uygulanmadı.

Day 14 hysteresis ayrı tutuldu ve değiştirilmedi.

## 7. Raw MSE ve EWMA

Day 21 detection logic değişmedi:

`MSE <= τ` → NORMAL

`MSE > τ` → ANOMALY

Day 22 alarm policy ise EWMA üzerinden karar verir:

`EWMA <= τ` → no anomaly candidate

`EWMA > τ` → anomaly candidate

بنابراین kısa süreli raw MSE spike'larının doğrudan alarm oluşturması engellenir.

## 8. Live Telemetry Integration

Yeni telemetry veya inference pipeline oluşturulmadı.

Mevcut Day 20 pipeline yeniden kullanıldı:

Simulator
→ WebSocket
→ TelemetryClient
→ Ring Buffer
→ 64-frame Window
→ Frozen μ/σ
→ TensorFlow.js Autoencoder
→ Reconstruction MSE
→ EWMA
→ Frozen τ
→ M-consecutive Alarm Policy

## 9. AI Monitor

AI Monitor genişletildi ve şu bilgiler gösteriliyor:

- Reconstruction MSE
- EWMA
- Frozen Threshold τ
- Anomaly Streak
- Alarm State

Örnek:

```text
RECONSTRUCTION MSE
0.02710

EWMA
0.02631

THRESHOLD
0.02546

ANOMALY STREAK
2 / 3

ALARM
PENDING
```

Üç consecutive anomalous window sonrasında:

`ALARM ACTIVE`

EWMA threshold altına döndüğünde:

`ALARM NORMAL`

## 10. Reconstruction MSE Chart

Mevcut `ReconstructionMseChart` korunmuştur.

Day 21'de eklenen frozen τ reference line korunmuştur.

Chart üzerinde:

- Reconstruction MSE
- τ reference line
- EWMA overlay

bulunmaktadır.

## 11. Day 14 Detector

Day 14 sistemi değiştirilmedi.

Day 14:

Fixed per-channel threshold + 5% hysteresis

olarak bağımsız çalışmaya devam etmektedir.

Day 22:

Multivariate Autoencoder Reconstruction MSE
+
EWMA
+
M-consecutive alarm confirmation

olarak ayrı bir detection/alarm path oluşturmaktadır.

## 12. Fault Data ve Tuning

Day 22'de:

- τ değiştirilmedi.
- α fault data ile tune edilmedi.
- M fault data ile tune edilmedi.

Synthetic deterministic sequences yalnızca EWMA ve state-machine davranışını test etmek için kullanıldı.

## 13. Tests

Yeni test command:

`npm run test:day22`

Testler doğruladı:

- Normal EWMA → NORMAL
- Tek anomalous window → CANDIDATE / PENDING
- M-1 anomalous windows → ALARM değil
- M anomalous windows → ALARM ACTIVE
- Normal window → streak reset
- Active alarm + normal EWMA → NORMAL
- Frozen τ değişmedi
- EWMA formula ile uyumlu
- Deterministic replay aynı sonucu üretiyor

## 14. Validation

```text
npx tsc --noEmit       PASS
npm run lint           PASS
npm run build          PASS

npm run test:day14     PASS
npm run test:day15     PASS
npm run test:day16     PASS
npm run test:day17     PASS
npm run test:day18     PASS
npm run test:day19     PASS
npm run test:day20     PASS
npm run test:day21     PASS
npm run test:day22     PASS
```

## 15. Browser Validation Limitation

AI Monitor live click-through browser validation bu ortamda çalıştırılmadı çünkü browser interaction tools mevcut değildi.

Buna rağmen synthetic Day 22 tests, EWMA calculation, state transitions, threshold wiring, TypeScript compilation ve production build başarıyla doğrulandı.

## 16. Files Created

- `src/lib/ml/day22/ewma.ts`
- `src/lib/ml/day22/alarmPolicy.ts`
- `src/lib/ml/day22SelfTest.ts`
- `scripts/run-day22-self-test.ts`
- `Gun_22/README.md`
- `ml/day22-validation.json`

## 17. Files Changed

- `src/lib/ml/day20/liveInference.ts` — live inference snapshot fields
- `src/lib/ml/useLiveReconstructionMse.ts` — policy step only on new sequence
- `src/components/dashboard/AiMonitorPanel.tsx` — MSE, EWMA, τ, streak, alarm
- `src/components/dashboard/ReconstructionMseChart.tsx` — τ line retained and EWMA overlay added
- `package.json` — `test:day22`
- `README.md` — Day 22 progress row

## 18. Systems Not Added

Day 22 deliberately does NOT include:

- Channel attribution
- SQLite alarm history
- AE vs baseline comparison
- Demo #5
- New simulator
- New Autoencoder
- New window pipeline
- New telemetry source

These are reserved for later Week 5 days.

## 19. Day 22 Result

Day 22 successfully added alarm stability to the existing Autoencoder reconstruction pipeline.

Final configuration:

- Frozen τ: `0.025459141133630285`
- EWMA α: `0.2`
- Alarm confirmation M: `3 windows`

Final logic:

Reconstruction MSE
→ EWMA
→ compare with frozen τ
→ consecutive anomaly counter
→ Alarm

## 20. Preparation for Day 23

Day 23 will focus on Channel Attribution.

Expected direction:

Autoencoder reconstruction
→ channel-wise reconstruction error
→ channel attribution
→ identify suspicious channel(s)

Day 23 must build on the existing Day 21/22 pipeline without changing the frozen threshold or retraining the model.

## 21. Summary

Day 22 transformed the Day 21 anomaly boundary into a more stable alarm decision policy.

The system now has:

Telemetry
→ 64-frame window
→ Frozen normalization
→ Autoencoder
→ Reconstruction MSE
→ EWMA
→ Frozen P99.5 threshold
→ M-consecutive confirmation
→ Alarm State

This provides the foundation for Day 23 channel attribution and the later alarm history layer.
