---
theme: default
background: /background.jpg
title: Bullet-Proof Java Containers
drawings:
  persist: false
transition: none
mdc: true
shiki: { theme: "nord" }
---

# Mission Possible: The 45-Minute Path to Bullet-Proof Java Container Images

<img src="/bellsoft.png" width="200px" class="absolute right-10px bottom-5px"/>

---
layout: image-right
image: "/cat.jpg"
---

## 'whoami'

🥑 Developer Advocate at BellSoft

😍 Java developer

👩‍💻 Tech writer

👾 CyberJAR Channel co-host (@cbrjar)

---

## BellSoft
<br/>

Member of:
- JCP Executive Committee
- OpenJDK Vulnerability Group
- GraalVM Advisory Board
- Linux Foundation
- Cloud Native Computing Foundation


---

## BellSoft
<br/>

Products:
- Liberica JDK
- Liberica Native Image Kit
- Alpaquita Linux
- BellSoft Hardened Images


---
layout: image
image: /room.png
---

---
layout: image
image: /room.png
---

## You’re sealed in a dark room. Water’s pouring from the ceiling. Somewhere in that water is a bomb. You’ve got 45 minutes.

---
layout: image
image: /room.png
---

- Room = Your container image
- Water = The constant CVE flow
- Bomb = Exploitable CVE in your context

---
layout: image
image: /room.png
---

## Ok, no panic, let's assess the situation first. <br> How bad is it? <br> <v-click>If we translate picture into code...</v-click>

---

## Initial Dockerfile

```dockerfile
FROM eclipse-temurin:25-jdk as builder
WORKDIR /app
COPY . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package

FROM eclipse-temurin:25-jre
WORKDIR /app
COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

Final image: 441MB

<v-click>Looks innocent enough, until...</v-click>

---

## ...Until we set the scanners loose

```bash
osv-scanner scan image neurowatch:latest
```

```bash

Total 13 packages affected by 18 known vulnerabilities (0 Critical, 4 High, 11 Medium, 3 Low, 0 Unknown) from 1 ecosystem.
╭─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Source:os:/var/lib/dpkg/status                                                                                                          │
├────────────────┬─────────────────────────┬──────────────────┬────────────┬─────────────────────────┬──────────────────┬─────────────────┤
│ SOURCE PACKAGE │ INSTALLED VERSION       │ FIX AVAILABLE    │ VULN COUNT │ BINARY PACKAGES (COUNT) │ INTRODUCED LAYER │ IN BASE IMAGE   │
├────────────────┼─────────────────────────┼──────────────────┼────────────┼─────────────────────────┼──────────────────┼─────────────────┤
│ coreutils      │ 9.4-3ubuntu6.1          │ No fix available │          2 │ coreutils               │ # 4 Layer        │ ubuntu          │
│ expat          │ 2.6.1-2ubuntu0.4        │ No fix available │          2 │ libexpat1               │ # 9 Layer        │ eclipse-temurin │
│ glibc          │ 2.39-0ubuntu8.7         │ No fix available │          1 │ libc-bin, libc6... (3)  │ # 4 Layer        │ ubuntu          │
│ gnupg2         │ 2.4.4-2ubuntu17.4       │ No fix available │          2 │ gpgv                    │ # 4 Layer        │ ubuntu          │
│ gnutls28       │ 3.8.3-1.1ubuntu3.4      │ Fix Available    │          1 │ libgnutls30t64          │ # 4 Layer        │ ubuntu          │
│ libgcrypt20    │ 1.10.3-2build1          │ No fix available │          1 │ libgcrypt20             │ # 4 Layer        │ ubuntu          │
│ lz4            │ 1.9.4-1build1.1         │ No fix available │          1 │ liblz4-1                │ # 4 Layer        │ ubuntu          │
│ ncurses        │ 6.4+20240113-1ubuntu2   │ No fix available │          1 │ libncursesw6... (4)     │ # 4 Layer        │ ubuntu          │
│ openssl        │ 3.0.13-0ubuntu3.7       │ No fix available │          2 │ libssl3t64, openssl     │ # 4 Layer        │ ubuntu          │
│ pam            │ 1.5.3-5ubuntu5.5        │ No fix available │          2 │ libpam-modules... (4)   │ # 4 Layer        │ ubuntu          │
│ shadow         │ 1:4.13+dfsg1-4ubuntu3.2 │ No fix available │          1 │ login, passwd           │ # 4 Layer        │ ubuntu          │
│ tar            │ 1.35+dfsg-3build1       │ No fix available │          1 │ tar                     │ # 4 Layer        │ ubuntu          │
│ zlib           │ 1:1.3.dfsg-3.1ubuntu2.1 │ No fix available │          1 │ zlib1g                  │ # 4 Layer        │ ubuntu          │
╰────────────────┴─────────────────────────┴──────────────────┴────────────┴─────────────────────────┴──────────────────┴─────────────────╯

```

---

## ...Until we set the scanners loose

```bash
trivy image neurowatch:latest
```

```bash

Report Summary

┌─────────────────────────────────────────────┬────────┬─────────────────┬─────────┐
│                   Target                    │  Type  │ Vulnerabilities │ Secrets │
├─────────────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ neurowatch-neurowatch:latest (ubuntu 24.04) │ ubuntu │       14        │    -    │
├─────────────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ app/app.jar                                 │  jar   │        2        │    -    │
└─────────────────────────────────────────────┴────────┴─────────────────┴─────────┘

```

---

## Could be worse...

```bash
trivy image openjdk:25-ea
```

```bash

Report Summary

┌────────────────────────────┬────────┬─────────────────┬─────────┐
│           Target           │  Type  │ Vulnerabilities │ Secrets │
├────────────────────────────┼────────┼─────────────────┼─────────┤
│ openjdk:25-ea (oracle 9.6) │ oracle │       69        │    -    │
└────────────────────────────┴────────┴─────────────────┴─────────┘

```

```bash
trivy image openjdk:21-ea
```

```bash

Report Summary

┌────────────────────────────┬────────┬─────────────────┬─────────┐
│           Target           │  Type  │ Vulnerabilities │ Secrets │
├────────────────────────────┼────────┼─────────────────┼─────────┤
│ openjdk:21-ea (oracle 8.8) │ oracle │       131       │    -    │
└────────────────────────────┴────────┴─────────────────┴─────────┘

```

---

## Anything else?

- Runs as root
- Includes a package manager: malicious packages can be installed
- No data on what's in the image, who built it, and where it's coming from

<br>

> Kubernetes and Docker security standards strongly advise running containers as non-root or rootless to limit the blast surface if the container is compromised.

<br>
<v-click>⚠️ If you don’t set the user, Docker runs as root by default!</v-click>

---

## Ok, time is running out, what's the plan?

Mission objective:<br><br>
Low-noise, locked-down Java container on a hardened base with zero unmanaged risk

Rules:<br><br>
No chasing after every CVE<br>
No leaving the room

---

## Ok, time is running out, what's the plan?

Actions:<br><br>
Harden the base<br>
Shrink privileges<br>
Generate provenance<br>
Scan<br>
Automate updates monitoring<br>

---

## Step 1: Pick a hardened base

TODO

---
layout: two-cols-header
---

## Hardened images: new container security standard
<br/>

<div class="hi-grid">
  <div class="hb">Low to zero CVEs</div>
  <div class="arrow">➡️</div>
  <div class="hb">Safer base from the start</div>

  <div class="hb">No package manager, minimalistic base</div>
  <div class="arrow">➡️</div>
  <div class="hb">Minimized attack surface</div>

  <div class="hb">Immutable component set</div>
  <div class="arrow">➡️</div>
  <div class="hb">No tampering with container at runtime</div>

  <div class="hb">SBOM, digital signature</div>
  <div class="arrow">➡️</div>
  <div class="hb">You know what's in the image and who made it</div>

  <div class="hb">Continuous patching</div>
  <div class="arrow">➡️</div>
  <div class="hb">The image stays safe</div>
</div>

<style>
.hi-grid{
  display:grid;
  grid-template-columns: 1fr 56px 1fr;
  row-gap: 14px; column-gap: 18px;
  align-items:center;
  max-width: 1100px;
}
.hb{
  background: #0f1424;
  border: 2px solid #00e6ff;
  border-radius: 12px;
  padding: 16px 18px;
  min-height: 64px;
  display:flex; align-items:center;
  font-size: 18px; line-height:1.3;
}
.arrow{
  text-align:center; font-size: 38px; line-height:1; font-weight: 700;
  color:#00e6ff; opacity:0.95;
}
</style>

---

## New Dockerfile

```dockerfile
FROM bellsoft/hardened-liberica-runtime-container:jdk-25-glibc as builder

WORKDIR /app
ADD my-app /app/my-app
RUN cd my-app && ./mvnw package

FROM bellsoft/hardened-liberica-runtime-container:jre-25-glibc

WORKDIR /app
COPY --from=builder /app/my-app/target/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

---

### Let's scan it to make sure we are on the right way

```bash

```

---
layout: cover
---

## Great, we cleaned up the mess

### But how do we protect the container from moles sneaking in?...


---

## Step 2: Shrink privileges

🤩 Most hardened images run as non-root by default.

If not, do it manually:


```dockerfile
USER 1234:1234
```

Or

```dockerfile
RUN groupadd -r myuser && useradd -r -g myuser myuser
USER myuser
```

---

## Step 2: Shrink privileges

No --privileged flag unless you absolutely need it:

```bash
docker run --privileged
```

Better: limit the granted privileges only to those needed by the container:
```bash
docker run --cap-drop all --cap-add <required-privilege>
```

Prevent escalation of privileges at runtime:
```bash
--security-opt no-new-privileges
```

---

## Step 3: Ensure provenance

---

## Step 4: Scan wisely

---

## Step 5: Enable automated updates monitoring