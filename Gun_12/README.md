# 📊 Gün 12 - WebSocket Client ve Telemetry Transport

## 📅 Tarih

14 Ağustos 2026

## 📝 Konu

WebSocket Client, Ring Buffer, Backpressure, Automatic Reconnect ve Clock Synchronization

---

## 1. Günün Amacı

Gün 12 kapsamında, mevcut WebSocket Server üzerine client-side telemetry
transport katmanı geliştirildi.

Amaç; WebSocket üzerinden gelen telemetry verilerinin client tarafında
güvenilir şekilde alınması, doğrulanması, sınırlı bir buffer içerisinde
tutulması ve bağlantı problemlerinin yönetilmesidir.

Bu kapsamda aşağıdaki bileşenler geliştirildi:

- TelemetryClient
- Ring Buffer
- Backpressure
- Automatic Reconnect
- Sequence Tracking
- Clock Synchronization
- Subscriber API

Dashboard bu aşamada WebSocket'e geçirilmedi.

---

## 2. TelemetryClient

Day 12'nin ana bileşeni:

```text
src/lib/telemetry/TelemetryClient.ts
```

TelemetryClient, WebSocket üzerinden gelen telemetry verilerini alır ve
client-side processing katmanına aktarır.

Client yeni bir Simulator oluşturmaz ve telemetry üretmez.

Veri akışı:

```text
WebSocket Server
       ↓
TelemetryClient
       ↓
Frame Validation
       ↓
Ring Buffer
       ↓
Consumer
```

Client bağlantı durumunu da takip eder.

Kullanılan temel durumlar:

```text
disconnected
connecting
connected
reconnecting
closed
```

Aynı anda birden fazla aktif WebSocket bağlantısının oluşturulması
engellenmiştir.

---

## 3. TelemetryFrame Validation

WebSocket üzerinden gelen mesajlar doğrudan kullanılmamaktadır.

Öncelikle mesaj JSON olarak parse edilir ve mevcut TelemetryFrame yapısına
uygunluğu kontrol edilir.

Geçersiz JSON veya geçersiz telemetry frame client'ın çalışmasını
bozmadan reject edilir ve ilgili sayaçlara eklenir.

Self-test sonucu:

```text
Telemetry JSON: PASS
Frame validation: PASS
```

---

## 4. Ring Buffer

Telemetry verilerinin sınırsız şekilde memory içerisinde birikmesini
önlemek için bounded Ring Buffer kullanıldı.

Client buffer kapasitesi:

```text
256 frames
```

Bu kapasite:

```text
DEFAULT_CLIENT_BUFFER_CAPACITY = 256
```

olarak belirlenmiştir.

Ring Buffer'ın temel amacı gerçek zamanlı telemetry akışında memory
kullanımının kontrol altında tutulmasıdır.

Self-test sonuçları:

```text
Ring buffer store: PASS
Bounded capacity: PASS
```

---

## 5. DROP_OLDEST Overflow Policy

Buffer kapasitesi dolduğunda yeni gelen telemetry frame'in eklenebilmesi
için en eski frame çıkarılır.

İşlem:

```text
Buffer Full
    ↓
Oldest Frame Removed
    ↓
Newest Frame Inserted
    ↓
droppedFrames++
```

Bu politika sayesinde daha güncel telemetry verileri buffer içerisinde
korunur.

Self-test sonuçları:

```text
Overflow DROP_OLDEST: PASS
Dropped frame counter: PASS
```

---

## 6. Backpressure

Client tarafında buffer kullanım oranı takip edilmektedir.

Backpressure seviyeleri:

```text
NORMAL
utilization < 70%

WARNING
utilization >= 70%

FULL
size === capacity
```

Buffer FULL olduğunda `DROP_OLDEST` politikası uygulanır.

Client istatistiklerinde aşağıdaki bilgiler tutulmaktadır:

- buffer size
- buffer capacity
- buffer utilization
- dropped frame count
- backpressure level

Bu yapı sayesinde consumer yavaşladığında buffer'ın sınırsız şekilde
büyümesi engellenmiştir.

Self-test:

```text
Backpressure stats: PASS
```

---

## 7. Automatic Reconnect

WebSocket bağlantısı beklenmedik şekilde kapandığında client otomatik
olarak yeniden bağlanmaktadır.

Reconnect için bounded exponential backoff kullanılmıştır:

```text
250 ms
500 ms
1000 ms
2000 ms
4000 ms
5000 ms
```

Maksimum reconnect gecikmesi:

```text
5000 ms
```

olarak sınırlandırılmıştır.

Başarılı bağlantı sonrasında reconnect attempt counter sıfırlanmaktadır.

Self-test:

```text
Automatic reconnect: PASS
Duplicate reconnect timer: PASS
```

---

## 8. Intentional Disconnect

Uygulama tarafından bilinçli olarak `disconnect()` çağrıldığında otomatik
reconnect yapılmamaktadır.

Davranış:

```text
Intentional disconnect
        ↓
No reconnect

Unexpected connection loss
        ↓
Automatic reconnect
```

Self-test:

```text
Intentional disconnect: PASS
```

---

## 9. Sequence Tracking

Mevcut TelemetryFrame içerisindeki `seq` alanı client tarafından takip
edilmektedir.

Sequence gap'leri gözlemlenmekte ve istatistiklerde tutulmaktadır.

Sequence gap otomatik olarak fault olarak kabul edilmemektedir.

Bunun nedeni bir gap'in network problemi, reconnect veya buffer overflow
gibi farklı nedenlerden kaynaklanabilmesidir.

Self-test:

```text
Sequence tracking: PASS
```

---

## 10. Clock Synchronization

Client ve server arasındaki yaklaşık zaman farkını hesaplamak için
hafif bir clock offset mekanizması geliştirildi.

Kullanılan yaklaşım:

```text
offset ≈ frame.t - clientReceiveTime
```

Son 16 örnek kullanılarak rolling mean hesaplanmaktadır.

```text
Clock samples = 16
```

Bu yapı gerçek bir NTP protokolü değildir.

Network latency, jitter ve processing delay ayrı ayrı hesaplanmamaktadır.

Self-test:

```text
Clock offset: PASS
```

---

## 11. Subscriber API

TelemetryClient içerisinde yeni telemetry frame'lerini dinleyebilecek
subscriber mekanizması oluşturuldu.

Temel yapı:

```text
TelemetryClient
      ↓
Subscriber
      ↓
TelemetryFrame
```

Bu yapı React'e bağımlı değildir ve ilerleyen aşamada Dashboard'un
TelemetryClient'a bağlanmasını kolaylaştıracaktır.

Self-test:

```text
Subscriber: PASS
```

---

## 12. Client Statistics

TelemetryClient aşağıdaki istatistikleri takip etmektedir:

```text
receivedFrames
acceptedFrames
rejectedFrames
droppedFrames
sequenceGaps
reconnectCount
bufferSize
bufferCapacity
bufferUtilization
lastSequence
clockOffsetMs
clockSyncSamples
connectionState
```

Bu bilgiler ilerleyen aşamada telemetry bağlantısının ve buffer
durumunun Dashboard üzerinde gösterilmesi için kullanılabilir.

---

## 13. Eklenen Dosyalar

Gün 12 kapsamında aşağıdaki dosyalar oluşturuldu:

```text
src/lib/telemetry/TelemetryClient.ts
src/lib/telemetry/day12SelfTest.ts
scripts/run-day12-self-test.ts
Gun_12/README.md
```

---

## 14. Değiştirilen Dosyalar

Aşağıdaki dosyalarda Gün 12 için değişiklik yapıldı:

```text
src/lib/telemetry/FrameHistoryBuffer.ts
package.json
README.md
```

`FrameHistoryBuffer.ts` içerisinde `push()` işleminin `DROP_OLDEST`
sonucunda bir frame'in düşürülüp düşürülmediğini bildirebilmesi sağlandı.

`package.json` içerisine aşağıdaki test script'i eklendi:

```bash
npm run test:day12
```

---

## 15. Day 12 Self-Test

Self-test aşağıdaki komut ile çalıştırıldı:

```bash
npm run test:day12
```

Sonuç:

```text
Day 12 Telemetry Client Self-Test
---------------------------------
Client module: PASS
Connect to Day 11 server: PASS
Telemetry JSON: PASS
Frame validation: PASS
Ring buffer store: PASS
Bounded capacity: PASS
Overflow DROP_OLDEST: PASS
Dropped frame counter: PASS
Backpressure stats: PASS
Sequence tracking: PASS
Automatic reconnect: PASS
Duplicate reconnect timer: PASS
Intentional disconnect: PASS
Clock offset: PASS
Subscriber: PASS

All Day 12 checks passed.
```

Tüm Day 12 self-test kontrolleri başarıyla tamamlandı.

---

## 16. Validation

Day 12 sonunda aşağıdaki kontroller PASS sonucuyla tamamlandı:

| Kontrol | Sonuç |
| :--- | :---: |
| TypeScript Type Check | ✅ PASS |
| ESLint | ✅ PASS |
| Production Build | ✅ PASS |
| Day 08 Self-Test | ✅ PASS |
| Day 09 Self-Test | ✅ PASS |
| Day 10 Self-Test | ✅ PASS |
| Day 11 Self-Test | ✅ PASS |
| Day 12 Self-Test | ✅ PASS |

---

## 17. Dashboard Entegrasyonu

Gün 12 kapsamında Dashboard WebSocket'e geçirilmedi.

Dashboard halen mevcut süreç içi `TelemetryService` yapısını kullanmaktadır.

Bu günün amacı öncelikle WebSocket client transport katmanını bağımsız
olarak geliştirmek ve test etmektir.

Dashboard entegrasyonu bir sonraki aşamada gerçekleştirilecektir.

---

## 18. Sınırlamalar

### Clock Synchronization

Clock offset yaklaşık bir tahmindir.

Gerçek bir NTP veya benzeri zaman senkronizasyon protokolü uygulanmamıştır.

### Node WebSocket Client

Client tarafında Node `ws` paketi kullanılmaktadır.

Henüz React browser WebSocket API'sine bağlanmamıştır.

### Sequence Gaps

Sequence gap'leri takip edilmektedir ancak otomatik olarak fault olarak
sınıflandırılmamaktadır.

### Malformed Messages

Geçersiz JSON veya TelemetryFrame mesajları reject edilmekte ve
istatistiklerde sayılmaktadır.

Bu mesajlar client'ın crash olmasına neden olmamaktadır.

---

## 19. Sonuç

Gün 12 sonunda SpikeEdge Telemetry projesinin WebSocket client-side
transport katmanı tamamlandı.

Bu kapsamda:

- TelemetryClient geliştirildi.
- TelemetryFrame validation eklendi.
- 256 frame kapasiteli bounded Ring Buffer oluşturuldu.
- DROP_OLDEST overflow policy uygulandı.
- Backpressure seviyeleri oluşturuldu.
- Automatic reconnect geliştirildi.
- Reconnect için bounded exponential backoff kullanıldı.
- Duplicate reconnect timer engellendi.
- Intentional disconnect davranışı eklendi.
- Sequence tracking geliştirildi.
- Clock offset hesaplama eklendi.
- Subscriber API oluşturuldu.
- Client statistics oluşturuldu.

Day 12 self-test içerisindeki tüm kontroller başarıyla tamamlandı.

---

## 20. Bir Sonraki Aşama

Bir sonraki aşamada TelemetryClient'ın Dashboard ile entegre edilmesi
planlanmaktadır.

Hedeflenen yapı:

```text
WebSocket Server
       ↓
TelemetryClient
       ↓
Ring Buffer
       ↓
Dashboard
       ↓
Live Charts
       ↓
Connection State
       ↓
Backpressure State
```

Bu aşamada WebSocket üzerinden gelen telemetry verilerinin Dashboard
tarafından kullanılması ve bağlantı ile buffer durumlarının kullanıcıya
gösterilmesi hedeflenmektedir.

---

## 21. Gün 12 Durumu

```text
TelemetryClient              ✅
WebSocket Connection         ✅
TelemetryFrame Validation    ✅
Ring Buffer                  ✅
Bounded Capacity             ✅
DROP_OLDEST                  ✅
Backpressure                 ✅
Automatic Reconnect          ✅
Reconnect Protection         ✅
Intentional Disconnect      ✅
Sequence Tracking            ✅
Clock Offset                 ✅
Subscriber API               ✅
Client Statistics             ✅
Day 12 Self-Test              ✅
```

**Gün 12 tamamlandı. ✅**
