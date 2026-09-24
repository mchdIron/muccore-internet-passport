# MUCCORE Internet Passport

**Domain ve internete açık servisler için Public Surface Intelligence platformu.**

[![Durum](https://img.shields.io/badge/durum-aktif-6C63FF)](https://passport.muccore.com)
[![Platform](https://img.shields.io/badge/platform-MUCCORE-111827)](https://passport.muccore.com)

**Canlı platform:** https://passport.muccore.com  
**English:** [README.md](README.md)

MUCCORE Internet Passport, public bir domaini internete açık güvenlik duruşunun evidence-backed bir görünümüne dönüştürür. Tek değerlendirmede Web, DNS, Mail, TLS, Infrastructure ve Behavior evidence bir araya getirilir; izole Safe Preview ile görsel bağlam eklenir ve sonuç tek bir surface-intelligence konsolunda sunulur.

> Bu repository yalnızca **public ürün dokümantasyonu** içerir. Production source code, deployment configuration, internal endpoint'ler ve operasyonel secret'lar burada yayınlanmaz.

## MUCCORE neleri analiz ediyor?

| Yüzey | Kapsam |
|---|---|
| **Web Security** | HTTPS davranışı, redirect'ler, security header'lar, HSTS, CSP, framing korumaları, cookie attribute'ları ve browser-facing policy kontrolleri |
| **DNS Security** | Temel DNS kayıtları, DNSSEC, CAA, authoritative nameserver duruşu ve DNS evidence |
| **Mail Security** | MX topolojisi, SPF, DMARC, MTA-STS, TLS-RPT ve SMTP/STARTTLS transport evidence |
| **TLS & Certificates** | Protocol capability, certificate trust/hostname kontrolleri, chain/expiry evidence, cipher gözlemleri, legacy TLS ve forward secrecy |
| **Infrastructure** | Public IPv4/IPv6 ilişkileri, MX infrastructure, PTR/rDNS ve internete açık topology evidence |
| **Behavior Signals** | Redirect, form, password field, iframe, external script ve benzeri gözlemlenebilir davranış sinyalleri |
| **Safe Preview** | İzole browser rendering, statik screenshot evidence ve erişim challenge/protection sayfalarının tespit/sınıflandırılması |

Bunlara ek olarak MUCCORE coverage-aware scoring, structured findings, technical evidence, JSON export, scan history ve önceki gözlemlerle değişim karşılaştırması sunar.

## Ürün nasıl çalışıyor?

```mermaid
flowchart LR
    T["Public Target"] --> C["MUCCORE<br/>Collection & Analysis"]
    C --> W["Web"]
    C --> D["DNS"]
    C --> M["Mail"]
    C --> L["TLS"]
    C --> I["Infrastructure"]
    C --> B["Behavior"]
    C --> P["Safe Preview"]

    W --> E["Evidence Layer"]
    D --> E
    M --> E
    L --> E
    I --> E
    B --> E
    P --> E

    E --> F["Findings + Coverage"]
    F --> S["Security Posture Scores"]
    S --> H["History + Change Intelligence"]
    H --> R["Internet Passport"]

    classDef core fill:#17152b,stroke:#7c5cff,color:#fff,stroke-width:2px;
    classDef module fill:#0e1420,stroke:#53657d,color:#fff;
    classDef evidence fill:#101b1a,stroke:#29c995,color:#fff;
    class T,C,R core;
    class W,D,M,L,I,B,P module;
    class E,F,S,H evidence;
```

## Üst seviye mimari

Public dokümantasyondaki mimari bilinçli olarak **sorumlulukları** gösterir; private implementation ayrıntılarını yayınlamaz.

```mermaid
flowchart TB
    U([Analist]) -->|HTTPS| UI["MUCCORE Internet Passport<br/>Surface Intelligence Console"]

    subgraph CP["Edge Control & Analysis Plane"]
        O["Scan Orchestration"]
        A["Analysis Engines"]
        S[("Evidence & History")]
        O --> A
        O <--> S
    end

    UI --> O

    subgraph MP["Isolated Measurement Plane"]
        N["MUCCORE Research Node"]
        TP["Transport Security Probes"]
        SP["SMTP / STARTTLS Measurements"]
        BP["Isolated Browser Capture"]
        N --> TP
        N --> SP
        N --> BP
    end

    O -->|"Authenticated measurement jobs"| N
    TP --> NET((Public Internet))
    SP --> NET
    BP --> NET
    A --> NET

    classDef edge fill:#17152b,stroke:#7c5cff,color:#fff,stroke-width:2px;
    classDef plane fill:#0e1420,stroke:#53657d,color:#fff;
    classDef data fill:#101b1a,stroke:#29c995,color:#fff;
    class UI,O,A,N,TP,SP,BP edge;
    class U,NET plane;
    class S data;
```

**Control & Analysis Plane** hedef doğrulama, modül koordinasyonu, evidence normalization, coverage-aware sonuç üretimi ve historical observation yönetimini üstlenir. **Isolated Measurement Plane** ise gerçek network veya browser runtime gerektiren ölçümleri gerçekleştirir. Böylece public application katmanı ile aktif measurement execution birbirinden ayrılır.

## Uçtan uca topoloji ve veri akışı

Bu görünüm ürün modüllerini gerçek servis/üretici sınırlarıyla birleştirir. Yani yalnızca “hangi modül var?” değil; **scan nereden başlıyor, hangi üretici hangi işi yapıyor, veri nereye gidiyor ve sonuç nereden geri geliyor?** sorularını da gösterir. Private adres, credential ve deployment secret'ları bilinçli olarak gösterilmez.

```mermaid
flowchart TB
    USER(["Analist / Browser"])

    subgraph GITHUB["GitHub · Source & Delivery"]
        GH["Private Production Repository"]
        DOC["Public Product Documentation"]
    end

    subgraph CF["Cloudflare · Edge / Control / Data Plane"]
        EDGE["passport.muccore.com<br/>Public UI + API"]
        WORKER["Cloudflare Worker<br/>Scan Orchestration"]
        ENGINES["Worker Analysis Engines<br/>Web · DNS · Mail Policy<br/>Infrastructure · Behavior · Scoring"]
        D1[("Cloudflare D1<br/>Scan Evidence · Findings<br/>History · Deltas")]
        KV[("Cloudflare KV<br/>Ephemeral Scan State / Cache")]
        TUNNEL["Cloudflare Tunnel<br/>Authenticated Service Path"]
    end

    subgraph NODE["MUCCORE Research Node · Measurement Plane"]
        API["Node.js Research Service"]
        TLS["HTTPS / TLS Probe"]
        SMTP["SMTP / STARTTLS Probe"]
        CHROME["Playwright / Chromium<br/>Safe Preview"]
    end

    subgraph GOOGLE["Google Cloud · SMTP Execution Path"]
        GCS["Google Cloud Shell<br/>SMTP Executor"]
    end

    subgraph INTERNET["Harici / Target Tarafındaki Üreticiler"]
        DNS["Authoritative / Recursive DNS"]
        WEB["Target Web Infrastructure<br/>Origin · CDN · WAF"]
        MX["Target Mail Providers / MX"]
        PKI["Public PKI / TLS Endpoints"]
    end

    USER -->|"1 · Scan başlat / sonucu görüntüle"| EDGE
    GH -.->|"Production delivery"| WORKER
    GH -.->|"Sanitized docs"| DOC
    EDGE -->|"2 · API request"| WORKER
    WORKER -->|"3 · Orchestrate"| ENGINES
    WORKER <-->|"State"| KV
    WORKER <-->|"Evidence / history"| D1

    ENGINES <-->|"4a · DNS / HTTP evidence"| DNS
    ENGINES <-->|"4b · Web evidence"| WEB

    WORKER -->|"5 · Authenticated measurement job"| TUNNEL
    TUNNEL --> API
    API --> TLS
    API --> SMTP
    API --> CHROME

    TLS <-->|"6a · TLS handshake / certificate evidence"| PKI
    TLS <-->|"6b · HTTPS measurement"| WEB
    CHROME <-->|"6c · İzole page rendering"| WEB

    SMTP -->|"7 · SMTP execution request"| GCS
    GCS <-->|"8 · TCP/25 · EHLO · STARTTLS"| MX
    GCS -->|"9 · SMTP/TLS evidence"| SMTP

    API -->|"10 · Measurement result"| TUNNEL
    TUNNEL --> WORKER
    ENGINES -->|"11 · Normalize / correlate / score"| WORKER
    WORKER -->|"12 · Canonical sonucu sakla"| D1
    WORKER -->|"13 · COMPLETED / PARTIAL result"| EDGE
    EDGE -->|"14 · Intelligence görünümü"| USER

    classDef cf fill:#17152b,stroke:#7c5cff,color:#fff,stroke-width:2px;
    classDef node fill:#101827,stroke:#38bdf8,color:#fff,stroke-width:2px;
    classDef google fill:#182313,stroke:#8bc34a,color:#fff,stroke-width:2px;
    classDef ext fill:#171717,stroke:#6b7280,color:#fff;
    classDef store fill:#10201b,stroke:#29c995,color:#fff;
    classDef user fill:#25173a,stroke:#c084fc,color:#fff,stroke-width:2px;
    class USER user;
    class EDGE,WORKER,ENGINES,TUNNEL cf;
    class D1,KV store;
    class API,TLS,SMTP,CHROME node;
    class GCS google;
    class DNS,WEB,MX,PKI ext;
```

### Üretici bazında görev ve trafik

| Üretici / katman | MUCCORE'daki görevi | Giden veri | Geri gelen veri |
|---|---|---|---|
| **Analist browser'ı** | Scan başlatır ve intelligence sonucunu görüntüler | Public target/domain ve scan isteği | Findings, score, evidence özeti, history ve preview |
| **Cloudflare** | Public edge, Worker compute, orchestration, storage ve güvenli servis yolu | Measurement job'ları ve public-target request'leri | Worker analizleri, stored evidence ve Research Node cevapları |
| **MUCCORE Research Node** | Ayrılmış network/browser measurement plane | HTTPS/TLS probe, preview request ve SMTP job | TLS, certificate, SMTP ve isolated-render evidence |
| **Google Cloud Shell** | Research Node ortamından doğrudan outbound TCP/25 mümkün olmadığında kullanılan SMTP executor | MX host / SMTP measurement job | SMTP banner, EHLO, STARTTLS ve transport/TLS evidence |
| **Target DNS üreticileri** | Public DNS posture'u sağlar | DNS query | DNS records, DNSSEC/CAA/authority evidence |
| **Target web/CDN/WAF üreticileri** | Target'ın public HTTP yüzeyini sunar | HTTP/HTTPS request ve isolated browser navigation | Header, redirect, content/render state ve protection/challenge evidence |
| **Target mail üreticileri** | Public MX/SMTP transport'u sunar | SMTP connection, EHLO ve STARTTLS negotiation | SMTP capability ve transport TLS evidence |
| **GitHub** | Source control/deployment source ve ayrı public dokümantasyon | Versioned project changes | Deployment source ve public documentation |

Buradaki en önemli ayrım: **Cloudflare control/data plane**, **MUCCORE Research Node ise dedicated measurement plane** olarak çalışır. Google Cloud Shell ana backend değildir; outbound TCP/25 gereken SMTP ölçümlerinde kullanılan özel execution path'idir. Cloudflare D1 ve KV Cloudflare tarafında kalır; scan evidence uygulama storage'ı olarak Google Cloud Shell'e yazılmaz.

## Evidence-aware scoring

MUCCORE ölçülemeyen telemetry'yi otomatik güvenlik hatasına dönüştürmez.

- Her modül gerçekten ölçülmüş evidence üzerinden değerlendirilir.
- `UNKNOWN`, evidence'ın yetersiz veya ulaşılamaz olduğunu belirtir; otomatik failure değildir.
- `NOT_APPLICABLE` score cezası oluşturmaz.
- Informational observation'lar score-impacting finding'lerden ayrılır.
- Gerekli coverage eksikse MUCCORE yapay bir kesinlik üretmek yerine numeric sonucu withheld edebilir.

Böylece score, gözlemlenen security posture'un özeti olur; alttaki finding ve evidence yorumlanabilir biçimde korunur.

## Safe Preview

Safe Preview, canlı target sayfasını analistin browser'ına embed etmeden görsel bağlam üretmek için tasarlanmıştır.

```mermaid
flowchart LR
    URL["Public URL"] --> ISO["Isolated Browser Capture"]
    ISO --> OBS["Render / Protection State Gözlemi"]
    OBS --> IMG["Static Screenshot"]
    OBS --> META["Capture Evidence"]
    IMG --> UI["MUCCORE Console"]
    META --> UI

    classDef core fill:#17152b,stroke:#7c5cff,color:#fff,stroke-width:2px;
    classDef safe fill:#101b1a,stroke:#29c995,color:#fff;
    class URL,ISO,OBS core;
    class IMG,META,UI safe;
```

Modern sayfaların doğru render edilebilmesi için izole capture ortamında JavaScript çalışabilir. Analiste target'ın canlı interaktif sayfası yerine statik visual/evidence çıktısı verilir. Gözlemlenen CAPTCHA, challenge, access-denied veya bot-protection durumları evidence olarak sınıflandırılabilir; MUCCORE CAPTCHA çözmeye veya siteye özel korumaları bypass etmeye çalışmaz.

## History & Change Intelligence

Tek scan bir snapshot'tır. Tekrarlanan gözlemler bu snapshot'ı timeline'a dönüştürür. MUCCORE yeni assessment'ı önceki stored evidence ile karşılaştırarak security policy, mail routing, TLS capability, certificate lifecycle ve module coverage/score gibi anlamlı posture değişikliklerini gösterebilir.

Mevcut ürün politikasında scan evidence **30 gün**, Safe Preview screenshot'ları ise **24 saat** tutulur.

## Güvenlik yaklaşımı

MUCCORE her target'ı untrusted input olarak kabul eder ve public-surface, low-impact observation yaklaşımıyla tasarlanmıştır. Private/reserved network destination'lar hedef kapsamının dışındadır ve measurement öncesinde outbound-target kontrolleri uygulanır.

Platform exploit delivery, brute force, credential testing, destructive request, broad directory fuzzing veya genel amaçlı port scanning için tasarlanmamıştır.

## Mevcut kapsam

MUCCORE aktif olarak geliştirilmektedir. Production kapsamı şu anda public Web, DNS, Mail, TLS, Infrastructure, passive Behavior Signals, isolated Preview, evidence-aware scoring ve historical comparison üzerine odaklanır. Daha zengin registration/provider intelligence ve ek enrichment özellikleri ürün geliştikçe eklenebilir.

---

**MUCCORE Internet Passport** — evidence first, surface aware, Internet'in ne gördüğünü anlamak için tasarlandı.
