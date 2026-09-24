# MUCCORE Internet Passport

**Public Surface Intelligence for domains and Internet-facing services.**

[![Status](https://img.shields.io/badge/status-active-6C63FF)](https://passport.muccore.com)
[![Platform](https://img.shields.io/badge/platform-MUCCORE-111827)](https://passport.muccore.com)

**Live platform:** https://passport.muccore.com  
**Türkçe:** [README.tr.md](README.tr.md)

MUCCORE Internet Passport turns a public domain into an evidence-backed view of its Internet-facing security posture. A single assessment correlates web, DNS, email, TLS, infrastructure and behavioral evidence, adds an isolated visual preview, and presents the result through one surface-intelligence console.

> This repository contains **public product documentation only**. The production source code, deployment configuration, internal endpoints and operational secrets are intentionally not published here.

## What MUCCORE analyzes

| Surface | Coverage |
|---|---|
| **Web Security** | HTTPS behavior, redirects, security headers, HSTS, CSP, framing protections, cookie attributes and browser-facing policy controls |
| **DNS Security** | Core DNS records, DNSSEC, CAA, authoritative nameserver posture and DNS evidence |
| **Mail Security** | MX topology, SPF, DMARC, MTA-STS, TLS-RPT and SMTP/STARTTLS transport evidence |
| **TLS & Certificates** | Protocol capability, certificate trust and hostname checks, chain/expiry evidence, cipher observations, legacy TLS and forward secrecy |
| **Infrastructure** | Public IPv4/IPv6 relationships, MX infrastructure, PTR/rDNS and Internet-facing topology evidence |
| **Behavior Signals** | Passive signals such as redirects, forms, password fields, iframes, external scripts and related observable behavior |
| **Safe Preview** | Isolated browser rendering, static screenshot evidence and detection/classification of access challenges or protection pages |

MUCCORE also provides coverage-aware scoring, structured findings, technical evidence, JSON export, scan history and comparison with previous observations.

## Product model

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

## High-level architecture

The public architecture intentionally describes **responsibilities**, not private implementation details.

```mermaid
flowchart TB
    U([Analyst]) -->|HTTPS| UI["MUCCORE Internet Passport<br/>Surface Intelligence Console"]

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

The **control and analysis plane** validates targets, coordinates modules, normalizes evidence, calculates coverage-aware results and maintains historical observations. The **isolated measurement plane** performs measurements that require a real network or browser runtime. This separation keeps the public application layer distinct from active measurement execution.

## End-to-end topology & data flow

This view combines the product modules with the actual service/provider boundaries. It shows **where a scan originates, which platform performs each part, where evidence travels, and where results return** without exposing private addresses, credentials or deployment secrets.

```mermaid
flowchart TB
    USER(["Analyst / Browser"])

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

    subgraph INTERNET["External / Target-side Providers"]
        DNS["Authoritative / Recursive DNS"]
        WEB["Target Web Infrastructure<br/>Origin · CDN · WAF"]
        MX["Target Mail Providers / MX"]
        PKI["Public PKI / TLS Endpoints"]
    end

    USER -->|"1 · Start scan / view result"| EDGE
    GH -.->|"Production delivery"| WORKER
    GH -.->|"Sanitized docs"| DOC
    EDGE -->|"2 · API request"| WORKER
    WORKER -->|"3 · Coordinate"| ENGINES
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
    CHROME <-->|"6c · Isolated page rendering"| WEB

    SMTP -->|"7 · SMTP execution request"| GCS
    GCS <-->|"8 · TCP/25 · EHLO · STARTTLS"| MX
    GCS -->|"9 · SMTP/TLS evidence"| SMTP

    API -->|"10 · Measurement result"| TUNNEL
    TUNNEL --> WORKER
    ENGINES -->|"11 · Normalize / correlate / score"| WORKER
    WORKER -->|"12 · Persist canonical result"| D1
    WORKER -->|"13 · Completed / partial result"| EDGE
    EDGE -->|"14 · Intelligence view"| USER

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

### Provider responsibilities and traffic

| Provider / zone | Role in MUCCORE | Data sent | Data returned |
|---|---|---|---|
| **Analyst browser** | Starts assessments and consumes intelligence | Public target/domain and scan request | Findings, scores, evidence summaries, history and preview |
| **Cloudflare** | Public edge, Worker compute, orchestration, storage and secure service path | Measurement jobs and public-target requests | Worker analysis, persisted evidence and Research Node responses |
| **MUCCORE Research Node** | Dedicated network/browser measurement plane | HTTPS/TLS probes, preview requests and SMTP jobs | TLS, certificate, SMTP and isolated-render evidence |
| **Google Cloud Shell** | SMTP executor used when direct outbound TCP/25 is unavailable from the Research Node environment | MX host/SMTP measurement job | SMTP banner, EHLO, STARTTLS and transport/TLS evidence |
| **Target DNS providers** | Supply public DNS posture | DNS queries | DNS records, DNSSEC/CAA/authority evidence |
| **Target web/CDN/WAF providers** | Serve the target's public HTTP surface | HTTP/HTTPS requests and isolated browser navigation | Headers, redirects, content/render state and protection/challenge evidence |
| **Target mail providers** | Expose public MX/SMTP transport | SMTP connection, EHLO and STARTTLS negotiation | SMTP capabilities and transport TLS evidence |
| **GitHub** | Source control and delivery source; separate public documentation repository | Versioned project changes | Deployment source and public documentation |

The key distinction is that **Cloudflare is the control/data plane**, while the **MUCCORE Research Node is the dedicated measurement plane**. Google Cloud Shell is not the main application backend; it is a specialized execution path for SMTP measurements where outbound TCP/25 connectivity is required. Cloudflare D1 and KV remain on the Cloudflare side; scan evidence does not use Google Cloud Shell as application storage.

## Evidence-aware scoring

MUCCORE does not turn missing telemetry into an automatic security failure.

- A module is scored from evidence that was actually measured.
- `UNKNOWN` means evidence was insufficient or unavailable; it is not automatically a failure.
- `NOT_APPLICABLE` does not create a penalty.
- Informational observations are separated from score-impacting findings.
- When required coverage is missing, MUCCORE can withhold a numeric conclusion rather than invent confidence.

This makes the score a summary of observed posture, while the underlying findings and evidence remain available for interpretation.

## Safe Preview

Safe Preview is designed to provide visual context without embedding the live target into the analyst's browser.

```mermaid
flowchart LR
    URL["Public URL"] --> ISO["Isolated Browser Capture"]
    ISO --> OBS["Observe Render / Protection State"]
    OBS --> IMG["Static Screenshot"]
    OBS --> META["Capture Evidence"]
    IMG --> UI["MUCCORE Console"]
    META --> UI

    classDef core fill:#17152b,stroke:#7c5cff,color:#fff,stroke-width:2px;
    classDef safe fill:#101b1a,stroke:#29c995,color:#fff;
    class URL,ISO,OBS core;
    class IMG,META,UI safe;
```

Modern pages may execute JavaScript inside the isolated capture environment so they can render correctly. The analyst receives static visual/evidence output rather than the target's live interactive page. Observed CAPTCHA, challenge, access-denied or bot-protection states can be classified as evidence; MUCCORE does not attempt to solve CAPTCHA or bypass site-specific protections.

## History & change intelligence

A single scan is a snapshot. Repeated observations turn that snapshot into a timeline. MUCCORE can compare a new assessment with previous stored evidence to surface meaningful posture changes such as security-policy changes, mail-routing changes, TLS capability changes, certificate lifecycle changes and module coverage/score changes.

Current product policy retains scan evidence for **30 days**. Safe Preview screenshots use a shorter **24-hour** retention window.

## Security philosophy

MUCCORE treats every target as untrusted input and is designed around public-surface, low-impact observation. Private/reserved network destinations are outside the intended target space, and the platform applies outbound-target controls before measurement.

The platform is **not** intended for exploit delivery, brute force, credential testing, destructive requests, broad directory fuzzing or general-purpose port scanning.

## Current scope

MUCCORE is actively developed. Current production scope focuses on public web, DNS, mail, TLS, infrastructure, passive behavior signals, isolated preview, evidence-aware scoring and historical comparison. Richer registration/provider intelligence and additional enrichment can be added as the platform evolves.

---

**MUCCORE Internet Passport** — evidence first, surface aware, built for understanding what the Internet can see.
