# Request Flow Architecture

## Full Stack

```text
                                INTERNET
                                   │
                                   ▼
Cloudflare
(DNS / Proxy / TLS / DDoS / WAF / Bot Filtering)
                                   │
                                   │
                                   │ Real Client IP
                                   ▼
Traefik
(EntryPoints :80 / :443)
                                   │
                                   │ Trusted Forwarded Headers
                                   ▼
CrowdSec Bouncer
(Global EntryPoint Middleware)
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                    │ Malicious / Banned IP       │ Clean IP
                    ▼                             ▼
                HTTP 403                    Continue Request
                                                  │
                                                  ▼
                                          Traefik Router
                                      (Host / Path Matching)
                                                  │
                             ┌────────────────────┴────────────────────┐
                             │                                         │
                             │ Protected Route                         │ Standard Route
                             ▼                                         ▼
                        Anubis Check                              Direct Backend
                   (ForwardAuth Middleware)                             │
                             │                                         │
                 ┌───────────┴───────────┐                             │
                 │                       │                             │
                 │ Challenge Failed      │ Challenge Passed            │
                 ▼                       ▼                             ▼
           Challenge Page          Backend Service              Backend Service
```

---

## Real Request Flow

```text
Visitor
   │
   ▼
Cloudflare
   │
   ▼
Traefik HTTPS EntryPoint
   │
   ▼
CrowdSec Middleware
   │
   ├── Blocked → 403 Forbidden
   │
   ▼
Router Match
   │
   ▼
Anubis Middleware (if enabled)
   │
   ├── Failed → Challenge Page
   │
   ▼
Backend Container
```

---

## Docker Network Layout

```text
                        Docker Host
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Traefik (:80 / :443)                                       │
│        │                                                    │
│        ├──────────────► CrowdSec Bouncer (:8922)            │
│        │                                                    │
│        ├──────────────► Anubis (:8923)                      │
│        │                                                    │
│        └──────────────► Backend Containers                  │
│                         ├── Grafana                         │
│                         ├── Prometheus                      │
│                         ├── Apps                            │
│                         └── APIs                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Middleware Order

```text
1. Cloudflare
2. Traefik EntryPoint
3. CrowdSec
4. Traefik Router Match
5. Anubis
6. Backend Service
```

---

## Security Layers

| Layer | Purpose |
|-------|---------|
| Cloudflare | DDoS, bot filtering, edge protection |
| CrowdSec | IP reputation, brute force, threat intelligence |
| Anubis | Browser verification, anti-bot challenge |
| Backend Auth | Application authentication |

---

## Result

```text
Internet
   ↓
Cloudflare
   ↓
Traefik
   ↓
CrowdSec
   ↓
Router
   ↓
Anubis
   ↓
Application
```