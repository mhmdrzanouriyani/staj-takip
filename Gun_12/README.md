Gün 12 - WebSocket Client ve Telemetry Transport

Tarih: 14 Ağustos 2026

Konu:
WebSocket Client, Ring Buffer, Backpressure, Automatic Reconnect ve Clock Synchronization


1. Günün Amacı

Bugün bir önceki gün oluşturduğum WebSocket Server'ın client tarafını
geliştirdim.

Amacım server tarafından gönderilen telemetry verilerini client tarafında
düzgün bir şekilde almak, gelen verileri kontrol etmek, sınırlı bir buffer
içerisinde tutmak ve bağlantı problemleri olduğunda sistemin tekrar
bağlanabilmesini sağlamaktı.

Bu kapsamda TelemetryClient, Ring Buffer, backpressure, automatic
reconnect, sequence tracking, clock synchronization ve subscriber
mekanizmalarını geliştirdim.

Bu gün Dashboard'u WebSocket'e bağlamadım. Öncelikle client tarafındaki
transport yapısını bağımsız olarak geliştirip test ettim.


2. TelemetryClient

Gün 12'nin ana geliştirmesi TelemetryClient oldu.

Dosya:

src/lib/telemetry/TelemetryClient.ts

TelemetryClient'ın görevi WebSocket Server'dan gelen telemetry verilerini
almak ve client tarafında işlemek.

Genel akış:

WebSocket Server
       ↓
TelemetryClient
       ↓
Frame Validation
       ↓
Ring Buffer
       ↓
Consumer


Client yeni bir Simulator oluşturmuyor ve kendisi telemetry üretmiyor.
Mevcut WebSocket stream'ini kullanıyor.

Bağlantı durumlarını da takip edecek şekilde geliştirildi:

- disconnected
- connecting
- connected
- reconnecting
- closed

Aynı anda gereksiz birden fazla WebSocket bağlantısı oluşturulmasını da
engelledim.


3. TelemetryFrame Validation

Server'dan gelen mesajları direkt kullanmak yerine önce JSON olarak
parse edip mevcut TelemetryFrame yapısına uygun olup olmadığını kontrol
ettim.

Geçersiz bir JSON veya hatalı telemetry frame geldiğinde client'ın
çökmesini engelledim. Bu mesajlar reject edilip ilgili sayaçlara
ekleniyor.

Test sonucu:

Telemetry JSON: PASS
Frame validation: PASS


4. Ring Buffer

Telemetry verilerinin sürekli birikerek memory kullanımını artırmaması
için bounded Ring Buffer kullandım.

Client tarafındaki buffer kapasitesini:

256 frame

olarak belirledim.

Buradaki amaç buffer'ın hiçbir zaman sınırsız şekilde büyümemesini
sağlamak.

Test sonucu:

Ring buffer store: PASS
Bounded capacity: PASS


5. DROP_OLDEST

Buffer tamamen dolduğunda yeni gelen veriyi kaybetmek yerine en eski
frame'i çıkartıp yeni frame'i ekleyecek şekilde DROP_OLDEST politikasını
kullandım.

Mantık:

Buffer Full
    ↓
Oldest Frame Removed
    ↓
Newest Frame Inserted
    ↓
droppedFrames++

Böylece gerçek zamanlı sistemde mümkün olduğunca güncel telemetry
verilerini tutmayı amaçladım.

Ayrıca kaç frame'in düşürüldüğünü takip etmek için droppedFrames
sayacını ekledim.

Test sonuçları:

Overflow DROP_OLDEST: PASS
Dropped frame counter: PASS


6. Backpressure

Client'ın telemetry verilerini tüketme hızı gelen verilerden daha düşük
olursa buffer'ın dolabileceğini göz önünde bulundurdum.

Bu nedenle buffer kullanım oranını takip eden basit bir backpressure
mekanizması oluşturdum.

Kullandığım seviyeler:

NORMAL
utilization < 70%

WARNING
utilization >= 70%

FULL
size === capacity

Buffer FULL olduğunda DROP_OLDEST uygulanıyor.

Ayrıca aşağıdaki bilgileri istatistiklerde tutuyorum:

- buffer size
- buffer capacity
- buffer utilization
- dropped frame count
- backpressure level

Test sonucu:

Backpressure stats: PASS


7. Automatic Reconnect

WebSocket bağlantısı beklenmedik şekilde kapanırsa client'ın kendisini
otomatik olarak tekrar bağlamasını sağladım.

Reconnect sırasında bounded exponential backoff kullandım:

250 ms
500 ms
1000 ms
2000 ms
4000 ms
5000 ms

Maksimum bekleme süresini 5000 ms ile sınırlandırdım.

Bağlantı başarılı olduğunda reconnect attempt sayacı tekrar
sıfırlanıyor.

Test sonuçları:

Automatic reconnect: PASS
Duplicate reconnect timer: PASS


8. Intentional Disconnect

Burada önemli bir ayrım yaptım.

Eğer uygulama bilinçli olarak disconnect() çağırırsa client'ın tekrar
bağlanmasını istemiyorum.

Bu nedenle:

Intentional disconnect
        ↓
No reconnect

Beklenmedik bağlantı kaybında ise:

Unexpected connection loss
        ↓
Automatic reconnect

şeklinde çalışıyor.

Test sonucu:

Intentional disconnect: PASS


9. Sequence Tracking

Mevcut TelemetryFrame içerisindeki seq değerini client tarafında takip
ettim.

Böylece frame'lerin sırasını ve olası sequence gap durumlarını
gözlemleyebiliyorum.

Sequence gap durumunu doğrudan fault olarak kabul etmedim. Çünkü bir gap
network problemi, reconnect veya buffer overflow gibi farklı nedenlerden
oluşabilir.

Test sonucu:

Sequence tracking: PASS


10. Clock Synchronization

Client ile server arasındaki yaklaşık zaman farkını görebilmek için
basit bir clock offset hesabı ekledim.

Kullandığım yaklaşım:

offset ≈ frame.t - clientReceiveTime

Tek bir değere güvenmek yerine son 16 örneği kullanarak rolling mean
hesapladım.

Bu yapı gerçek bir NTP sistemi değil. Network latency, jitter ve
processing delay ayrı ayrı hesaplanmıyor.

Amaç bu aşamada client ile server arasındaki yaklaşık zaman farkını
takip edebilmek.

Test sonucu:

Clock offset: PASS


11. Subscriber API

TelemetryClient içerisine yeni gelen telemetry frame'lerini dinlemek
için subscriber mekanizması ekledim.

Yapı genel olarak:

TelemetryClient
      ↓
Subscriber
      ↓
TelemetryFrame

şeklinde çalışıyor.

Bu yapıyı React'e bağımlı oluşturmadım. Böylece ilerleyen aşamada
Dashboard'u TelemetryClient'a bağlamak daha kolay olacak.

Test sonucu:

Subscriber: PASS


12. Client Statistics

Client'ın durumunu takip edebilmek için çeşitli istatistikler ekledim.

Takip edilen bilgiler:

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

Bu bilgileri ilerleyen aşamada Dashboard üzerinde bağlantı durumu,
buffer durumu ve telemetry sağlığı gibi bilgileri göstermek için
kullanabilirim.


13. Eklenen Dosyalar

Gün 12'de aşağıdaki dosyaları ekledim:

src/lib/telemetry/TelemetryClient.ts
src/lib/telemetry/day12SelfTest.ts
scripts/run-day12-self-test.ts
Gun_12/README.md


14. Değiştirdiğim Dosyalar

Gün 12 kapsamında:

src/lib/telemetry/FrameHistoryBuffer.ts
package.json
README.md

dosyalarında değişiklik yaptım.

FrameHistoryBuffer içerisinde push() işleminin DROP_OLDEST sonucunda
bir frame'in düşürülüp düşürülmediğini bildirecek şekilde düzenleme
yaptım.

package.json içerisine:

npm run test:day12

script'ini ekledim.

Root README içerisinde de Day 12 durumunu güncelledim.


15. Day 12 Self-Test

Day 12 için özel bir self-test hazırladım.

Çalıştırdığım komut:

npm run test:day12

Sonuç:

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

Bütün Day 12 self-test kontrolleri başarıyla tamamlandı.


16. Validation

Day 12 sonunda aşağıdaki kontroller de başarılı oldu:

TypeScript Type Check: PASS
ESLint: PASS
Production Build: PASS
Day 08 Self-Test: PASS
Day 09 Self-Test: PASS
Day 10 Self-Test: PASS
Day 11 Self-Test: PASS
Day 12 Self-Test: PASS

Kullandığım temel komutlar:

npx tsc --noEmit
npm run lint
npm run build
npm run test:day08
npm run test:day09
npm run test:day10
npm run test:day11
npm run test:day12


17. Dashboard Entegrasyonu

Bu gün Dashboard'u WebSocket'e bağlamadım.

Dashboard halen mevcut süreç içi TelemetryService yapısını kullanıyor.

Öncelikle TelemetryClient'ın bağımsız olarak çalıştığından ve bağlantı,
buffer, reconnect ve telemetry işlemlerinin doğru olduğundan emin olmak
istedim.

Dashboard entegrasyonunu sonraki aşamaya bıraktım.


18. Sınırlamalar

Clock synchronization şu anda yaklaşık bir offset hesabıdır. Gerçek bir
NTP veya benzeri zaman senkronizasyon protokolü kullanılmadı.

Client tarafında Node ws paketi kullanılıyor. Henüz React browser
WebSocket API'sine bağlanmadı.

Sequence gap'leri takip ediliyor ancak otomatik olarak fault olarak
sınıflandırılmıyor.

Geçersiz JSON veya TelemetryFrame mesajları reject ediliyor ve
istatistiklerde tutuluyor. Client'ın çökmesine neden olmuyor.


19. Sonuç

Bugünkü çalışma sonunda WebSocket Server'ın client tarafındaki transport
katmanını tamamladım.

TelemetryClient ile server'a bağlanmayı, gelen telemetry verilerini
doğrulamayı ve bunları 256 frame kapasiteli Ring Buffer içerisinde
tutmayı sağladım.

Buffer dolduğunda DROP_OLDEST kullanılıyor ve düşürülen frame sayısı
takip ediliyor.

Ayrıca backpressure durumu, automatic reconnect, sequence tracking,
clock offset ve subscriber mekanizmalarını ekledim.

Day 12 self-test içerisindeki bütün kontroller PASS oldu.

TypeScript, ESLint, production build ve önceki günlerin self-test
kontrolleri de başarılı şekilde tamamlandı.


20. Bir Sonraki Aşama

Bir sonraki aşamada TelemetryClient'ı Dashboard ile entegre etmeyi
planlıyorum.

Hedeflediğim yapı:

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

Böylece WebSocket üzerinden gelen gerçek telemetry akışı Dashboard
tarafında kullanılabilecek ve bağlantı ile buffer durumu kullanıcıya
gösterilebilecek.

Gün 12 tamamlandı. ✅
