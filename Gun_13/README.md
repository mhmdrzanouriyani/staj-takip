# Gün 13 — Dashboard ve WebSocket Entegrasyonu

## Tarih

15 Ağustos 2026

## Konu

TelemetryClient'ın Dashboard'a entegre edilmesi, canlı telemetry akışının
WebSocket üzerinden Dashboard'a aktarılması ve bağlantı/buffer durumlarının
arayüzde gösterilmesi.


## 1. Günün Genel Çalışması

Bugün bir önceki gün geliştirdiğim `TelemetryClient` yapısını mevcut
Dashboard'a entegre ettim.

Day 11'de oluşturulan WebSocket Server telemetry verilerini yayınlıyor,
Day 12'de geliştirdiğim TelemetryClient ise bu verileri client tarafında
alıp yönetiyordu.

Bugün bu iki yapıyı Dashboard ile birleştirdim.

Oluşturduğum genel akış şu hale geldi:

```text
Day 11 WebSocket Server
          ↓
TelemetryClient
          ↓
Ring Buffer
          ↓
DashboardTelemetryBridge
          ↓
useTelemetryClient
          ↓
Dashboard
          ↓
KPI Cards + Live Charts
```

Buradaki önemli nokta, React tarafına WebSocket veya reconnect mantığını
taşımamam oldu. Bu işlemler TelemetryClient içerisinde tutuldu ve
Dashboard sadece hazırlanan client abstraction üzerinden telemetry
verilerini kullanıyor.


## 2. Dashboard Entegrasyonunun Amacı

Bugünkü çalışmada amacım Dashboard'un daha önce kullandığı süreç içi
telemetry akışını WebSocket üzerinden gelen gerçek telemetry stream'i ile
birleştirmekti.

Bunu yaparken ikinci bir Simulator oluşturmadım.

Aynı şekilde Dashboard içerisinde ikinci bir WebSocket bağlantısı veya
ayrı bir Ring Buffer oluşturmadım.

Sonuç olarak tek telemetry kaynağı kullanılmaya devam ediyor:

```text
Simulator
    ↓
WebSocket Server
    ↓
TelemetryClient
    ↓
Dashboard
```

Mevcut `TelemetryService` projede bırakıldı ancak Dashboard artık onu
başlatmıyor. Böylece aynı anda iki farklı telemetry stream çalışmasının
önüne geçildi.


## 3. DashboardTelemetryBridge

Dashboard ile TelemetryClient arasında bir bridge katmanı oluşturdum.

Dosya:

```text
src/lib/telemetry/dashboardClient.ts
```

Bu katmanın amacı TelemetryClient'ı Dashboard'dan ayırmak ve Dashboard
tarafında kullanılabilecek ortak bir client instance sağlamaktır.

`DashboardTelemetryBridge` tek bir paylaşılan `TelemetryClient`
instance'ını yönetiyor.

Böylece Dashboard'un farklı component'leri tarafından tekrar tekrar
yeni WebSocket client oluşturulmasının önüne geçildi.


## 4. React Integration

Dashboard tarafında TelemetryClient'a bağlanmak için:

```text
src/lib/telemetry/useTelemetryClient.ts
```

dosyasını oluşturdum.

Bu hook'un görevi WebSocket mantığını tekrar yazmak değil, mevcut
`DashboardTelemetryBridge` üzerinden gelen state ve telemetry
değişikliklerini React tarafına taşımak.

React tarafında:

```text
Dashboard
    ↓
useTelemetryClient
    ↓
DashboardTelemetryBridge
    ↓
TelemetryClient
```

şeklinde bir yapı kullanılıyor.

Bu sayede React component'leri WebSocket bağlantısının nasıl yönetildiğini
bilmek zorunda kalmıyor.


## 5. Shared Client Instance

React geliştirme ortamında Strict Mode gibi durumlar aynı component'in
mount/unmount işlemlerini birden fazla kez tetikleyebilir.

Bu nedenle birden fazla WebSocket client oluşturulmasını engelleyecek
bir yapı kullandım.

`retain()` mekanizması reference-counted şekilde çalışıyor.

Son aktif kullanım kaldırıldığında:

```text
Last release
    ↓
disconnect()
    ↓
listeners removed
```

işlemleri gerçekleştiriliyor.

Böylece gereksiz bağlantıların ve listener'ların açık kalmasının önüne
geçiliyor.

Self-test sonucu:

```text
Shared client instance: PASS
No duplicate subscription: PASS
Cleanup / unsubscribe: PASS
```


## 6. Live Telemetry Dashboard

Dashboard'daki mevcut telemetry kartlarını WebSocket üzerinden gelen
verileri kullanacak şekilde bağladım.

Genel akış:

```text
WebSocket
    ↓
TelemetryClient
    ↓
Validated TelemetryFrame
    ↓
Dashboard Bridge
    ↓
Dashboard State
    ↓
KPI Cards / Charts
```

Dashboard'da mevcut telemetry kanalları kullanılmaya devam ediyor.

Yeni bir telemetry schema oluşturmadım.


## 7. Live Charts

Mevcut canlı grafik bileşenlerini de TelemetryClient üzerinden gelen
verilere bağladım.

Değiştirilen dosyalardan biri:

```text
src/components/charts/LiveTelemetryCharts.tsx
```

Grafikler Ring Buffer içerisindeki son 100 frame'i kullanıyor.

Client tarafındaki Ring Buffer kapasitesi ise:

```text
256 frames
```

Bu nedenle React içerisinde sürekli büyüyen ve sınırsız bir array
oluşturulmadı.

Akış:

```text
TelemetryClient
    ↓
256-frame Ring Buffer
    ↓
last 100 frames
    ↓
Live Charts
```

Bu yaklaşım memory kullanımının kontrol altında tutulmasına yardımcı
oluyor.


## 8. Connection Status

Dashboard'a WebSocket bağlantı durumunu göstermek için yeni bir component
oluşturdum:

```text
src/components/dashboard/TelemetryTransportStatus.tsx
```

Arayüzde bağlantı durumları Türkçe olarak gösteriliyor:

```text
Bağlanıyor
Bağlandı
Yeniden bağlanıyor
Bağlantı kesildi
Kapatıldı
```

Ayrıca bağlantıyı yönetmek için:

```text
Bağlan
Bağlantıyı kes
```

butonları eklendi.


## 9. Backpressure Durumu

Day 12'de oluşturduğum backpressure mekanizmasını Dashboard tarafında
da görünür hale getirdim.

Gösterilen durumlar:

```text
Normal
Uyarı
Dolu
```

Ayrıca buffer kullanımını:

```text
Buffer Kullanımı
size / 256 (%)
```

şeklinde gösteriyorum.

Böylece telemetry client'ın buffer durumunu sadece kod içerisinden
değil, Dashboard üzerinden de takip etmek mümkün hale geldi.


## 10. Telemetry Statistics

TelemetryClient tarafından sağlanan bazı istatistikleri Dashboard
entegrasyonunda kullanılabilir hale getirdim.

Bunlar arasında:

```text
receivedFrames
droppedFrames
reconnectCount
bufferSize
bufferCapacity
bufferUtilization
sequenceGaps
```

gibi bilgiler bulunuyor.

Bu değerler ilerleyen aşamada telemetry bağlantısının sağlığı ve
Dashboard monitoring özellikleri için kullanılabilir.


## 11. TelemetryClient Değişikliği

Day 13 kapsamında:

```text
src/lib/telemetry/TelemetryClient.ts
```

dosyasında da gerekli uyumluluk değişikliklerini yaptım.

Browser tarafında kullanılabilmesi için Node'a özel `ws` import'u
kullanılmayacak şekilde düzenlendi.

Ayrıca Dashboard entegrasyonu için:

```text
getHistory()
subscribeState()
```

gibi ek erişim noktaları oluşturuldu.

Bu değişiklikler mevcut Day 12 davranışını değiştirmeden Dashboard
entegrasyonunu mümkün hale getirmek amacıyla yapıldı.


## 12. WebSocket URL Configuration

WebSocket adresini ayrı bir configuration alanına taşıdım.

Dosya:

```text
src/config/telemetry.ts
```

Burada kullanılan WebSocket adresi:

```text
ws://127.0.0.1:8787
```

olarak tanımlanıyor.

Bu sayede ilerleyen aşamalarda WebSocket endpoint'ini Dashboard
kodunun farklı bölümlerini değiştirmeden yönetmek daha kolay olacak.


## 13. Eklenen Dosyalar

Day 13 kapsamında aşağıdaki dosyaları oluşturdum:

```text
src/lib/telemetry/dashboardClient.ts
src/lib/telemetry/useTelemetryClient.ts
src/components/dashboard/TelemetryTransportStatus.tsx
src/lib/telemetry/day13SelfTest.ts
scripts/run-day13-self-test.ts
Gun_13/README.md
```


## 14. Değiştirilen Dosyalar

Aşağıdaki dosyalarda Day 13 kapsamında değişiklik yaptım:

```text
src/lib/telemetry/TelemetryClient.ts
src/components/dashboard/LiveTelemetryPanel.tsx
src/components/charts/LiveTelemetryCharts.tsx
src/lib/telemetry/index.ts
src/config/telemetry.ts
package.json
README.md
```

Mevcut `TelemetryService` projede tutulmaya devam ediyor ancak Dashboard
artık onu başlatmıyor.

Böylece Dashboard tarafında iki farklı canlı telemetry stream
oluşturulmasının önüne geçildi.


## 15. Day 13 Self-Test

Geliştirdiğim Dashboard entegrasyonunun temel davranışlarını kontrol
etmek için Day 13'e özel self-test hazırladım.

Çalıştırdığım komut:

```bash
npm run test:day13
```

Test sonucu:

```text
Day 13 Dashboard Integration Self-Test
--------------------------------------
Integration module: PASS
TelemetryClient used by bridge: PASS
Subscriber updates: PASS
Latest telemetry state: PASS
Connection state: PASS
Backpressure state: PASS
Statistics: PASS
Cleanup / unsubscribe: PASS
No duplicate subscription: PASS
Shared client instance: PASS

All Day 13 checks passed.
```

Tüm otomatik Day 13 kontrolleri başarıyla tamamlandı.


## 16. Validation

Day 13 kapsamında aşağıdaki validation kontrolleri başarıyla tamamlandı:

```text
TypeScript Type Check: PASS
ESLint: PASS
Production Build: PASS
Day 08 Self-Test: PASS
Day 09 Self-Test: PASS
Day 10 Self-Test: PASS
Day 11 Self-Test: PASS
Day 12 Self-Test: PASS
Day 13 Self-Test: PASS
```

Kullanılan temel komutlar:

```bash
npx tsc --noEmit
npm run lint
npm run build
npm run test:day08
npm run test:day09
npm run test:day10
npm run test:day11
npm run test:day12
npm run test:day13
```


## 17. Manuel Browser Testi

Bu çalışma sırasında manuel browser testi gerçekleştirilmedi.

Bu nedenle Dashboard'un browser içerisindeki gerçek bağlantı davranışını
PASS olarak işaretlemedim.

Manuel kontrol için kullanılabilecek akış:

```bash
npm run server:telemetry
npm run dev
```

Daha sonra Dashboard browser üzerinden açılarak WebSocket bağlantısı,
telemetry akışı, grafikler ve reconnect davranışı kontrol edilebilir.

Bu test gerçekleştirilmediği için sonucu bu raporda PASS olarak
göstermiyorum.


## 18. Dashboard ve WebSocket Mimarisi

Day 13 sonunda ilgili yapı şu şekilde oldu:

```text
                Simulator
                    ↓
            WebSocket Server
                    ↓
            TelemetryClient
                    ↓
        DashboardTelemetryBridge
                    ↓
          useTelemetryClient
                    ↓
               Dashboard
              /                      ↓           ↓
        KPI Cards     Live Charts
                         ↓
                    Last 100 Frames
```

TelemetryClient içerisindeki 256 frame kapasiteli Ring Buffer korunuyor
ve grafikler bu buffer içerisinden son 100 frame'i kullanıyor.


## 19. Sınırlamalar

### WebSocket Server

Dashboard'ın WebSocket üzerinden telemetry alabilmesi için Day 11
server'ın şu adreste çalışıyor olması gerekiyor:

```text
ws://127.0.0.1:8787
```

### Server Kapalı Olduğunda

Server çalışmıyorsa Dashboard sahte telemetry üretmiyor ve ikinci bir
Simulator başlatmıyor.

Client bağlantıyı yeniden kurmayı deniyor ve UI bağlantı durumunu
gösteriyor.

### Clock Offset

Day 12'de oluşturulan clock offset bilgisi client statistics içerisinde
bulunuyor ancak Day 13 Dashboard panelinde gösterilmiyor.

### Channel Labels

Mevcut channel kartlarının bazı label'ları halen İngilizce
`CHANNEL_DISPLAY` yapısından geliyor.

Day 13 kapsamında eklenen yeni bağlantı ve transport label'ları ise
Türkçe olarak oluşturuldu.


## 20. Sonuç

Bugünkü çalışma ile Day 12'de hazırladığım TelemetryClient'ı mevcut
Dashboard'a entegre ettim.

Artık telemetry akışının genel yolu:

```text
Simulator
    ↓
WebSocket Server
    ↓
TelemetryClient
    ↓
Dashboard
```

şeklinde çalışacak şekilde hazırlandı.

Dashboard tarafında yeni bir Simulator oluşturmadım ve React içerisine
ayrı bir WebSocket veya reconnect mekanizması yazmadım.

Bunun yerine mevcut TelemetryClient ve Ring Buffer yapısını kullanarak
temiz bir Dashboard entegrasyonu oluşturdum.

Ayrıca bağlantı durumu, backpressure durumu ve bazı telemetry
istatistiklerini Dashboard tarafında gösterecek altyapıyı hazırladım.

Day 13 self-test içerisindeki bütün kontroller PASS oldu ve TypeScript,
ESLint, production build ile önceki günlerin testleri de başarılı şekilde
tamamlandı.

Manuel browser testi ise bu çalışma sırasında gerçekleştirilmedi.


## 21. Bir Sonraki Aşama

Bir sonraki aşamada Dashboard üzerindeki telemetry verilerinin daha
kullanışlı hale getirilmesi ve gerçek zamanlı monitoring özelliklerinin
geliştirilmesi planlanabilir.

Özellikle:

```text
Live Telemetry
      ↓
Thresholds
      ↓
Alarms
      ↓
Fault / Anomaly Indicators
      ↓
Better Offline / Connection UX
```

gibi özellikler değerlendirilebilir.

Bunun yanında projeyi ilerleyen aşamalarda daha sektörel hale getirmek
için 3D modelleme ve mevcut 3D çalışmaların telemetry altyapısı ile
birleştirilmesi de ayrı bir geliştirme yönü olarak değerlendirilebilir.

Gün 13 tamamlandı. ✅
