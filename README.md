<div align="center">

<!-- ─────────────  BAŞLIK  ───────────── -->

<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=600&size=26&duration=3200&pause=900&color=2563EB&center=true&vCenter=true&width=700&lines=Merhaba%2C+ben+Mohammadreza+Nouriyani;Bilgisayar+M%C3%BChendisli%C4%9Fi+%C3%B6%C4%9Frencisi;End%C3%BCstriyel+AI+%26+Real-Time+Web+Sistemleri" alt="Mohammadreza Nouriyani" />

### Bilgisayar Mühendisliği Öğrencisi · Bursa Teknik Üniversitesi

**Endüstriyel AI · Real-Time Web Sistemleri · Backend & Altyapı**

<br>

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/mhmdrzanouriyani)
[![YouTube](https://img.shields.io/badge/MohixCode-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@mohixcodee)
[![E-posta](https://img.shields.io/badge/E--posta-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:mohammadrezanouriyani@gmail.com)

<img src="https://komarev.com/ghpvc/?username=mhmdrzanouriyani&style=flat-square&color=2563eb&label=Profil+g%C3%B6r%C3%BCnt%C3%BClenme" alt="profil görüntülenme" />

</div>

---

## 👋 Hakkımda

Bursa Teknik Üniversitesi Bilgisayar Mühendisliği öğrencisiyim. İlgi alanım, **makine öğrenmesini
gerçek zamanlı sistemlerin içine yerleştirmek**: sensör verisini toplayan, görselleştiren ve
üzerinde anlamlı karar üreten uçtan uca uygulamalar geliştiriyorum.

Çalışma biçimimi üç ilke belirliyor:

- **Ölçülebilirlik** — Bir sistemin "çalıştığını" söylemek yetmez; precision, recall, latency ve
  bellek kullanımı ile ölçerim.
- **Şeffaflık** — Projelerimin sınırlamalarını sonuçlarıyla aynı yerde belgelerim.
- **Devredilebilirlik** — Projeyi devralan kişinin kurabileceği, çalıştırabileceği ve mimariyi
  anlayabileceği dokümantasyon yazarım.

> 🎯 Şu anda **endüstriyel yazılım, gömülü AI ve backend geliştirme** alanlarında staj ve
> proje iş birliklerine açığım.

---

## 🏭 Öne Çıkan Proje — SpikeEdge

<div align="center">

### ⚡ Industrial Digital Twin & AI Anomaly Detection

**Endüstriyel telemetriyi gerçek zamanlı izleyen, Digital Twin üzerinde görselleştiren ve
anomaliyi tamamen tarayıcı içinde tespit eden uçtan uca prototip.**

![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![TensorFlow.js](https://img.shields.io/badge/TensorFlow.js-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)
![Three.js](https://img.shields.io/badge/Three.js-049EF4?style=flat-square&logo=threedotjs&logoColor=white)
![Keras](https://img.shields.io/badge/Keras-D00000?style=flat-square&logo=keras&logoColor=white)
![WebAssembly](https://img.shields.io/badge/WASM-654FF0?style=flat-square&logo=webassembly&logoColor=white)

</div>

```mermaid
flowchart LR
    S["📡 Telemetry<br/>10 Hz · 6 kanal"] --> W["🔌 WebSocket"]
    W --> B["🗂 Ring Buffer"]
    B --> T["🧊 Digital Twin<br/>Three.js"]
    B --> P["🪟 64-Frame Window<br/>384 değer"]
    P --> AE["🧠 Autoencoder<br/>TF.js / WASM Worker"]
    AE --> E["〰️ EWMA + P99.5 τ"]
    E --> A["🚨 3-of-5 Alarm<br/>NORMAL / PENDING / ACTIVE"]

    classDef a fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef b fill:#ede9fe,stroke:#7c3aed,color:#4c1d95
    classDef c fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    class S,W,B,T a
    class P,AE,E b
    class A c
```

### Teknik kararlar ve sonuçları

| Karar | Gerekçe | Ölçülen sonuç |
|---|---|---|
| Sabit eşik yerine **64 frame'lik davranışsal pencere** | Tek kanallı eşik, kanallar arası korelasyon bozulmasını göremiyor | Recall **%53.3 → %76.4** |
| **EWMA (α = 0.2) + 3-of-5 kalıcılık** politikası | Anlık gürültünün kalıcı alarma dönüşmesini engellemek | Precision **%100**, F1 **%86.6** |
| Inference'ın **Web Worker + WASM**'a taşınması | 3D render döngüsü ile GPU'da yarışmasın, UI thread bloklanmasın | Warm P50 **0.457 ms** |
| **Dondurulmuş μ / σ ve eşik** | Model çalışma anında kendi anomalisine adapte olmasın | Calibration FPR **≈ %0.543** |

<div align="center">

| Model | Latent | Boyut | Throughput | Alarm F1 |
|:-:|:-:|:-:|:-:|:-:|
| Dense Autoencoder `384 → 64 → 16 → 64 → 384` | 16 (24:1 sıkıştırma) | ≈ 212 KB | ≈ 1686 inference/s | **%86.6** |

</div>

> **Şeffaflık notu:** Sistem simülasyon verisi kullanır; saha testi yapılmamıştır ve eşik
> kalibrasyonu bağımsız held-out sete dayanmaz. Bu sınırlamalar proje dokümantasyonunda
> sonuçlarla birlikte açıkça raporlanmıştır.

<div align="center">

**→ [Projeyi ve tam teknik dokümantasyonu incele](https://github.com/mhmdrzanouriyani)**

</div>

---

## 🛠 Teknoloji Yığını

<table>
<tr><td valign="top" width="50%">

**Diller**

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![C](https://img.shields.io/badge/C-A8B9CC?style=flat-square&logo=c&logoColor=black)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=flat-square&logo=postgresql&logoColor=white)

**Yapay Zekâ & Veri**

![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?style=flat-square&logo=tensorflow&logoColor=white)
![Keras](https://img.shields.io/badge/Keras-D00000?style=flat-square&logo=keras&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=numpy&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat-square&logo=pandas&logoColor=white)

</td><td valign="top" width="50%">

**Frontend & 3D**

![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=black)
![Three.js](https://img.shields.io/badge/Three.js-049EF4?style=flat-square&logo=threedotjs&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white)

**Backend & Altyapı**

![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)
![Celery](https://img.shields.io/badge/Celery-37814A?style=flat-square&logo=celery&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black)

</td></tr>
</table>

---

## 📦 Diğer Çalışmalar

| Proje | Açıklama | Teknolojiler |
|---|---|---|
| **CloudDeploy** | Self-hosted VPS yönetim paneli — sunucu provizyonu, uzaktan komut yürütme, kimlik doğrulama | FastAPI · PostgreSQL · Redis · Celery · AsyncSSH · Docker · JWT + TOTP |
| **NetProbe++** | Python ile güvenilir UDP dosya transfer platformu — paket kaybı ve yeniden iletim yönetimi | Python · Socket · Bilgisayar Ağları |
| **MohixCode** | Farsça programlama ve yapay zekâ eğitim içerikleri — müfredat, ders materyalleri ve video üretimi | Next.js · İçerik üretimi · Teknik anlatım |
| **AI Agent çalışmaları** | ReAct tabanlı araştırma ajanı, Telegram otomasyon ajanı ve masaüstü otomasyon sistemi | FastAPI · SSE · LLM API'leri · Telethon |

---

## 🎓 Eğitim & Sertifikasyon

| Kurum / Alan | Ayrıntı |
|---|---|
| 🏛 **Bursa Teknik Üniversitesi** | Bilgisayar Mühendisliği — lisans |
| 🌐 **Cisco CCNA** | ITN (Introduction to Networks) — Modül 1–3 çalışması |
| ⚡ **Elektrik Devreleri** | Devre temelleri, mesh ve node analizi |
| 🗣 **Diller** | Farsça (ana dil) · Türkçe · İngilizce |

---

## 📊 GitHub İstatistikleri

<div align="center">

<img height="165" src="https://github-readme-stats.vercel.app/api?username=mhmdrzanouriyani&show_icons=true&hide_border=true&title_color=2563eb&icon_color=7c3aed&count_private=true" alt="GitHub istatistikleri" />
<img height="165" src="https://github-readme-stats.vercel.app/api/top-langs/?username=mhmdrzanouriyani&layout=compact&hide_border=true&title_color=2563eb&langs_count=8" alt="En çok kullanılan diller" />

<br><br>

<img src="https://github-readme-streak-stats.herokuapp.com/?user=mhmdrzanouriyani&hide_border=true&ring=2563eb&fire=7c3aed&currStreakLabel=2563eb" alt="Katkı serisi" />

</div>

---

## 📬 İletişim

Proje iş birliği, staj veya teknik bir soru için çekinmeden yazabilirsiniz.

<div align="center">

[![E-posta](https://img.shields.io/badge/mohammadrezanouriyani@gmail.com-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:mohammadrezanouriyani@gmail.com)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/)

<br>

<sub>💡 <i>"Bir sistemin değeri, çalışmasıyla değil; nasıl çalıştığının ölçülebilir ve
anlatılabilir olmasıyla belirlenir."</i></sub>

</div>
