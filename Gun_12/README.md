# Gün 12 — WebSocket Client ve Telemetry Transport

## Tarih
14 Ağustos 2026

## Konu
WebSocket Client, Ring Buffer, Backpressure, Automatic Reconnect ve Clock Synchronization


## 1. Günün Genel Çalışması

Bugün, bir önceki gün oluşturduğum WebSocket Server'ın client tarafını
geliştirmeye odaklandım.

Day 11'de telemetry verilerinin WebSocket üzerinden yayınlanmasını
sağlamıştım. Bugün ise bu verileri karşılayan client tarafını oluşturdum.

Amacım sadece WebSocket bağlantısı kurmak değil; gelen telemetry
verilerinin güvenli şekilde alınması, kontrol edilmesi, belirli bir
kapasitede tutulması ve bağlantı sırasında oluşabilecek problemlerin
yönetilmesiydi.

Bu nedenle client tarafına Ring Buffer, Backpressure, Automatic
Reconnect, Sequence Tracking, Clock Synchronization ve Subscriber
mekanizmalarını ekledim.

Bu aşamada Dashboard'u WebSocket'e bağlamadım. Öncelikle oluşturduğum
client transport katmanının bağımsız ve güvenilir şekilde çalıştığından
emin olmak istedim.


## 2. Oluşturulan TelemetryClient

Günün ana geliştirmesi `TelemetryClient` oldu.

Dosya:

```text
src/lib/telemetry/TelemetryClient.ts
```

TelemetryClient'ın temel görevi WebSocket Server tarafından gönderilen
TelemetryFrame verilerini almak ve bunları client tarafında yönetmek.

Veri akışını şu şekilde oluşturdum:

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

Burada önemli nokta, TelemetryClient'ın yeni bir Simulator oluşturmaması
ve telemetry verisi üretmemesidir. Mevcut WebSocket stream'i doğrudan
client tarafında işlenmektedir.

Ayrıca bağlantının hangi durumda olduğunu takip edebilmek için aşağıdaki
connection state'leri kullanıldı:

```text
disconnected
connecting
connected
reconnecting
closed
```

Böylece client'ın bağlantı durumunu daha sonra Dashboard tarafında da
gösterebilecek bir temel hazırlanmış oldu.


## 3. Gelen Telemetry Verilerinin Kontrolü

WebSocket üzerinden gelen mesajları doğrudan kullanmak yerine önce
JSON olarak parse edip mevcut TelemetryFrame yapısına uygunluğunu kontrol
ettim.

Geçersiz bir mesaj geldiğinde client'ın çalışmasını durdurmak yerine
mesajı reject edip ilgili istatistiklere ekleyecek şekilde tasarladım.

Bu sayede hatalı bir mesajın bütün telemetry client'ı etkilemesinin
önüne geçildi.

Self-test sonucunda:

```text
Telemetry JSON: PASS
Frame validation: PASS
```


## 4. Ring Buffer

Telemetry verilerinin sürekli olarak birikmesini ve memory kullanımının
kontrolsüz şekilde artmasını önlemek için bounded Ring Buffer kullandım.

Client tarafındaki buffer kapasitesi:

```text
256 frames
```

olarak belirlendi.

Bu yapı sayesinde buffer'ın belirlenen kapasitenin üzerine çıkması
engellenmiş oldu.

Ring Buffer'ın temel amacı özellikle consumer tarafı gelen verileri
yeterince hızlı işleyemediğinde telemetry akışının kontrol altında
tutulmasıdır.

Test sonuçları:

```text
Ring buffer store: PASS
Bounded capacity: PASS
```


## 5. Buffer Dolduğunda DROP_OLDEST

Buffer tamamen dolduğunda yeni gelen veriyi doğrudan kaybetmek yerine
en eski frame'i çıkartıp yeni frame'i ekleyecek şekilde
`DROP_OLDEST` politikasını kullandım.

Çalışma mantığı:

```text
Buffer Full
    ↓
Oldest Frame Removed
    ↓
Newest Frame Inserted
    ↓
droppedFrames++
```

Buradaki amaç gerçek zamanlı telemetry akışında mümkün olduğunca güncel
verileri korumaktır.

Ayrıca kaç frame'in buffer'dan çıkarıldığını takip etmek için
`droppedFrames` sayacı kullanıldı.

Self-test sonuçları:

```text
Overflow DROP_OLDEST: PASS
Dropped frame counter: PASS
```


## 6. Backpressure Yönetimi

Telemetry producer ile consumer'ın aynı hızda çalışmayabileceğini göz
önünde bulundurdum.

Örneğin server'dan gelen telemetry verileri client'ın işleyebileceği
hızdan daha hızlı gelirse buffer zamanla dolabilir.

Bu durumu takip etmek için basit ve kontrollü bir backpressure yapısı
oluşturdum.

Kullanılan seviyeler:

```text
NORMAL
utilization < 70%

WARNING
utilization >= 70%

FULL
size === capacity
```

Buffer FULL seviyesine ulaştığında `DROP_OLDEST` politikası devreye
giriyor.

Ayrıca aşağıdaki değerleri takip ediyorum:

- buffer size
- buffer capacity
- buffer utilization
- dropped frame count
- backpressure level

Self-test:

```text
Backpressure stats: PASS
```


## 7. Automatic Reconnect

WebSocket bağlantısının beklenmedik şekilde kapanabileceğini göz önünde
bulundurarak automatic reconnect mekanizması geliştirdim.

Bağlantı koptuğunda client hemen sürekli olarak tekrar bağlanmaya
çalışmak yerine bounded exponential backoff kullanıyor.

Bekleme süreleri:

```text
250 ms
500 ms
1000 ms
2000 ms
4000 ms
5000 ms
```

Maksimum bekleme süresi 5000 ms ile sınırlandırıldı.

Bağlantı tekrar başarılı olduğunda reconnect attempt sayacı sıfırlanıyor.

Ayrıca aynı anda birden fazla reconnect timer oluşturulmasını da
engelledim.

Test sonuçları:

```text
Automatic reconnect: PASS
Duplicate reconnect timer: PASS
```


## 8. Intentional Disconnect

Reconnect mekanizmasında özellikle dikkat ettiğim noktalardan biri,
bilinçli olarak kapatılan bağlantı ile beklenmedik şekilde kopan
bağlantıyı birbirinden ayırmaktı.

Eğer uygulama `disconnect()` çağırarak bağlantıyı bilinçli şekilde
kapatırsa:

```text
Intentional disconnect
        ↓
No reconnect
```

Beklenmedik bir bağlantı kaybında ise:

```text
Unexpected connection loss
        ↓
Automatic reconnect
```

şeklinde çalışıyor.

Bu davranış self-test ile doğrulandı:

```text
Intentional disconnect: PASS
```


## 9. Sequence Tracking

TelemetryFrame içerisinde bulunan `seq` değerini client tarafında
takip edecek bir yapı ekledim.

Böylece gelen frame'lerin sırası hakkında bilgi sahibi olabiliyorum ve
sequence gap durumlarını takip edebiliyorum.

Sequence gap'i doğrudan bir fault olarak değerlendirmedim.

Çünkü bir sequence gap network problemi, reconnect veya buffer
overflow gibi farklı nedenlerden kaynaklanabilir.

Bu nedenle bu aşamada gap'leri gözlemleyip istatistiklerde tutmayı
tercih ettim.

Self-test:

```text
Sequence tracking: PASS
```


## 10. Clock Synchronization

Client ile server arasındaki zaman farkını yaklaşık olarak görebilmek
için basit bir clock offset hesaplama mekanizması geliştirdim.

Kullanılan yaklaşım:

```text
offset ≈ frame.t - clientReceiveTime
```

Tek bir örneğe bağlı kalmak yerine son 16 örneğin rolling mean değerini
kullanıyorum.

```text
Clock samples = 16
```

Bu yapının gerçek bir NTP sistemi olmadığını özellikle belirtmek
gerekir. Network latency, jitter ve processing delay burada ayrı ayrı
hesaplanmıyor.

Bu aşamadaki amaç, client ile server arasındaki yaklaşık zaman farkını
takip edebilecek temel bir yapı oluşturmaktı.

Self-test:

```text
Clock offset: PASS
```


## 11. Subscriber API

TelemetryClient içerisine yeni telemetry frame'lerini dinleyebilmek
için subscriber mekanizması ekledim.

Temel yapı:

```text
TelemetryClient
      ↓
Subscriber
      ↓
TelemetryFrame
```

Bu yapıyı React'e doğrudan bağımlı oluşturmadım.

Böylece ilerleyen aşamada Dashboard'un TelemetryClient'a bağlanması
daha kolay olacak.

Self-test:

```text
Subscriber: PASS
```


## 12. Client İstatistikleri

Client'ın çalışma durumunu daha kolay takip edebilmek için çeşitli
istatistikler ekledim.

Takip edilen temel bilgiler:

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

Bu bilgiler ilerleyen aşamada Dashboard üzerinde bağlantı durumu,
buffer kullanımı ve telemetry akışının durumu gibi bilgileri göstermek
için kullanılabilir.


## 13. Eklenen Dosyalar

Bugünkü çalışma kapsamında aşağıdaki dosyaları ekledim:

```text
src/lib/telemetry/TelemetryClient.ts
src/lib/telemetry/day12SelfTest.ts
scripts/run-day12-self-test.ts
Gun_12/README.md
```


## 14. Değiştirilen Dosyalar

Aşağıdaki dosyalarda Day 12 için değişiklik yaptım:

```text
src/lib/telemetry/FrameHistoryBuffer.ts
package.json
README.md
```

`FrameHistoryBuffer.ts` içerisinde `push()` işlemini DROP_OLDEST
sonucunda bir frame'in düşürülüp düşürülmediğini bildirecek şekilde
düzenledim.

`package.json` içerisine Day 12 self-test komutunu ekledim:

```bash
npm run test:day12
```

Root README içerisinde de Day 12 ilerleme durumunu güncelledim.


## 15. Day 12 Self-Test

Geliştirdiğim yapının gerçekten çalıştığını kontrol etmek için Day 12
için ayrı bir self-test hazırladım.

Çalıştırdığım komut:

```bash
npm run test:day12
```

Test sonucu:

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

Bütün Day 12 kontrolleri başarıyla tamamlandı.


## 16. Genel Validation

Day 12 sonunda proje üzerinde yapılan validation kontrollerinin tamamı
başarılı oldu.

```text
TypeScript Type Check: PASS
ESLint: PASS
Production Build: PASS
Day 08 Self-Test: PASS
Day 09 Self-Test: PASS
Day 10 Self-Test: PASS
Day 11 Self-Test: PASS
Day 12 Self-Test: PASS
```

Kullanılan komutlar:

```bash
npx tsc --noEmit
npm run lint
npm run build
npm run test:day08
npm run test:day09
npm run test:day10
npm run test:day11
npm run test:day12
```


## 17. Dashboard Entegrasyonu

Bu gün Dashboard tarafında herhangi bir WebSocket değişikliği yapmadım.

Dashboard halen mevcut süreç içi `TelemetryService` yapısını kullanıyor.

Bunun nedeni öncelikle WebSocket client tarafındaki bağlantı yönetimi,
buffer, reconnect ve telemetry işleme mekanizmalarını bağımsız olarak
tamamlamak ve test etmekti.

Dashboard entegrasyonunu sonraki aşamaya bıraktım.


## 18. Sınırlamalar

Bu aşamada clock synchronization yalnızca yaklaşık bir clock offset
hesabıdır. Gerçek bir NTP veya benzeri zaman senkronizasyon protokolü
uygulanmadı.

Client tarafında Node `ws` paketi kullanılıyor. Henüz React browser
WebSocket API'sine doğrudan bağlanmadı.

Sequence gap'leri takip ediliyor ancak otomatik olarak fault olarak
sınıflandırılmıyor.

Geçersiz JSON veya TelemetryFrame mesajları reject ediliyor ve
istatistiklerde tutuluyor. Bu mesajlar client'ın crash olmasına neden
olmuyor.


## 19. Sonuç

Bugünkü çalışma sonunda SpikeEdge Telemetry projesinin WebSocket
client-side transport katmanını tamamladım.

Server'dan gelen telemetry verilerini alabilen ve doğrulayabilen bir
TelemetryClient oluşturdum.

Ayrıca 256 frame kapasiteli bounded Ring Buffer, DROP_OLDEST overflow
politikası, backpressure takibi, automatic reconnect, sequence
tracking, clock offset hesabı ve subscriber mekanizmasını ekledim.

Reconnect mekanizmasını kontrollü tutmak için exponential backoff
kullandım ve aynı anda birden fazla reconnect timer oluşmasını
engelledim.

Geliştirdiğim tüm Day 12 self-test kontrolleri PASS oldu. TypeScript,
ESLint, production build ve önceki günlerin testleri de başarılı şekilde
tamamlandı.

Bu çalışma ile telemetry verisinin sadece server tarafında yayınlanması
değil, client tarafında güvenilir şekilde alınması ve yönetilmesi için de
temel altyapı hazırlanmış oldu.


## 20. Bir Sonraki Aşama

Bir sonraki aşamada geliştirdiğim TelemetryClient'ı Dashboard'a
entegre etmeyi planlıyorum.

Hedeflediğim yapı:

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

Böylece Dashboard doğrudan WebSocket üzerinden gelen telemetry akışını
kullanabilecek ve bağlantı ile buffer durumlarını kullanıcıya
gösterebilecek.

Gün 12 tamamlandı. ✅
