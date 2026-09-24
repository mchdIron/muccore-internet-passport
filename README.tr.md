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
