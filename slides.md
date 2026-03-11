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
layout: image-right
image: "/members.png"
---

## BellSoft

<br/>

- Java and Linux experts with 15+ years of experience
- Members of various boards/committees
- Author of Alpine Linux Port

---

## BellSoft

<br/>

Products:

- Liberica JDK
- Liberica Native Image Kit
- Alpaquita Linux
- BellSoft Hardened Images

---

<SlidevVideo v-click autoplay controls>
  <source src="/room_video.mp4" type="video/mp4" />
  <autoplay>true</autoplay>
</SlidevVideo>

---
layout: image
image: /room.png
---

# You’re sealed in a dark room. <br>Water’s pouring from the ceiling.<br> Somewhere in that water is a bomb.<br><br> You’ve got 45 minutes.

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

## Let's assess the situation first. <br> How bad is it? <br><br> <v-click>If we translate picture into code...</v-click>

---

## NeuroWatch Demo
<br/>

- Petclinic on steroids
- Spring Boot 4
- Java 25
- Spring Security
- Spring AI
- MongoDB
- Vaadin

---

## Initial Dockerfile


```dockerfile {none|1|4|6|8,9,10}{maxHeight:'250px'}
FROM eclipse-temurin:25-jdk as builder
WORKDIR /app
COPY . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction package
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/neurowatch/target/*.jar"]
```

Final image: 1GB

<v-click>Looks innocent enough, until...</v-click>

---

## ...Until we set the scanners loose

```bash
osv-scanner scan image neurowatch:latest
```

```bash {all|2,3|all}{maxHeight:'300px'}

Total 19 packages affected by 46 known vulnerabilities (0 Critical, 12 High, 16 Medium, 8 Low, 10 Unknown) from 2 ecosystems.
5 vulnerabilities can be fixed.

Maven
╭─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Source:artifact:/app/app.jar                                                                                                │
├─────────────────────────────────────────┬───────────────────┬───────────────┬────────────┬──────────────────┬───────────────┤
│ PACKAGE                                 │ INSTALLED VERSION │ FIX AVAILABLE │ VULN COUNT │ INTRODUCED LAYER │ IN BASE IMAGE │
├─────────────────────────────────────────┼───────────────────┼───────────────┼────────────┼──────────────────┼───────────────┤
│ com.fasterxml.jackson.core:jackson-core │ 2.20.2            │ Fix Available │          1 │ # 17 Layer       │ --            │
│ org.yaml:snakeyaml                      │ 1.33              │ Fix Available │          1 │ # 17 Layer       │ --            │
│ tools.jackson.core:jackson-core         │ 3.0.3             │ Fix Available │          2 │ # 17 Layer       │ --            │
╰─────────────────────────────────────────┴───────────────────┴───────────────┴────────────┴──────────────────┴───────────────╯
Ubuntu:24.04
╭─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Source:os:/var/lib/dpkg/status                                                                                                          │
├────────────────┬─────────────────────────┬──────────────────┬────────────┬─────────────────────────┬──────────────────┬─────────────────┤
│ SOURCE PACKAGE │ INSTALLED VERSION       │ FIX AVAILABLE    │ VULN COUNT │ BINARY PACKAGES (COUNT) │ INTRODUCED LAYER │ IN BASE IMAGE   │
├────────────────┼─────────────────────────┼──────────────────┼────────────┼─────────────────────────┼──────────────────┼─────────────────┤
│ binutils       │ 2.42-4ubuntu2.8         │ No fix available │         22 │ binutils... (8)         │ # 9 Layer        │ eclipse-temurin │
│ coreutils      │ 9.4-3ubuntu6.1          │ No fix available │          2 │ coreutils               │ # 4 Layer        │ ubuntu          │
│ dpkg           │ 1.22.6ubuntu6.5         │ No fix available │          1 │ dpkg                    │ # 4 Layer        │ ubuntu          │
│ expat          │ 2.6.1-2ubuntu0.4        │ No fix available │          2 │ libexpat1               │ # 9 Layer        │ eclipse-temurin │
│ freetype       │ 2.13.2+dfsg-1build3     │ No fix available │          1 │ libfreetype6            │ # 9 Layer        │ eclipse-temurin │
│ gnupg2         │ 2.4.4-2ubuntu17.4       │ No fix available │          2 │ gpgv                    │ # 4 Layer        │ ubuntu          │
│ gnutls28       │ 3.8.3-1.1ubuntu3.4      │ Fix Available    │          1 │ libgnutls30t64          │ # 4 Layer        │ ubuntu          │
│ libgcrypt20    │ 1.10.3-2build1          │ No fix available │          1 │ libgcrypt20             │ # 4 Layer        │ ubuntu          │
│ libpng1.6      │ 1.6.43-5ubuntu0.5       │ No fix available │          1 │ libpng16-16t64          │ # 9 Layer        │ eclipse-temurin │
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
│ neurowatch-neurowatch:latest (ubuntu 24.04) │ ubuntu │       38        │    -    │
├─────────────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ app/app.jar                                 │  jar   │        4        │    -    │
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
│ openjdk:25-ea (oracle 9.6) │ oracle │       72        │    -    │
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
│ openjdk:21-ea (oracle 8.8) │ oracle │       132       │    -    │
└────────────────────────────┴────────┴─────────────────┴─────────┘

```

---
layout: image
image: /cve-count.svg
---


---

## Apart from that...
<br/>

- package manager,
- runs as root,
- random base,
- tons of packages,
- CVE noise,
- No provenance,
- irregular updates.


---

## Why do we care?
<br/>

- Some CVEs are real bombs (Remote Code Execution, RCE)
- You may have RCE vuln and not even know it!
- Failed audit = legal consequences

---

## Time is running out, what's the plan?
<br/>

Mission objective:<br><br>
Low-noise, locked-down Java container on a hardened base with zero unmanaged risk

Rules:<br><br>
No chasing after every CVE<br>
- Not all of them are exploitable in your case
- Constant re-testing -> too heavy load on the system

---

## Time is running out, what's the plan?
<br/>

Actions:<br><br>
Shrink privileges<br>
Scan<br>
Classify risk<br>
Pick the right base<br>
Generate provenance<br>

---
layout: cover
---

## Step 1: Shrink privileges

### Goal: Reduce blast radius in case of attack.

---

## Step 1.1: No Running Containers as Root
<br/>

Misconfigurations / kernel vulnerabilities = 
Attacker can gain root access on the host system when being root in container

<br/>

RCE + root on the host = BIG BOOM!

---

## Step 1.1: No Running Containers as Root
<br/>

Run containers as non-root:

```dockerfile
USER 1234:1234
```

Or

```dockerfile
RUN groupadd -r myuser && useradd -r -g myuser myuser
USER myuser
```

---

## Step 1.2: No excessive privileges
<br/>

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
layout: cover
---

## We reduced the risk of explosion taking down the whole building
### Now, we need to find that bomb (or bombs)

---
layout: cover
---

## Step 2: Reduce the number of packages
### Goal: Easier to assess what's left

---

## Step 2.1: JDK to JRE
<br/>

- Use JRE at the final stage

> JDK is for building and compiling<br/>
JRE is for running

- Use Docker multistage builds

---

## Step 2.1: JDK to JRE
<br/>

Change the Dockerfile

````md magic-move
```dockerfile
FROM eclipse-temurin:25-jdk as builder
WORKDIR /app
COPY . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction package
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/neurowatch/target/*.jar"]
```

```docker
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
````


---

## Step 2.2: Pick a Minimalistic Linux
<br/>


|                    | Alpaquita<br/>(musl) | Alpaquita<br/>(glibc) | Alpine<br/>(musl) | RHEL<br/>(Distroless UBI 10 Micro) | Ubuntu Jammy | Debian Slim |
|--------------------|----------------------|-----------------------|-------------------|------------------------------------|--------------|-------------|
| Size on Docker Hub | 3.8MB                | 11.7MB                | 3.45MB            | 7.37MB                             | 28.17MB      | 27.8MB      |

<br/>

<v-click>Spoiler: Not all minimal Linux base images are created equal... </v-click>


---

## Step 2.2: Pick a Minimalistic Linux
<br/>

````md magic-move
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

```docker
FROM eclipse-temurin:25-jdk-alpine as builder
RUN apk add --no-cache nodejs npm
WORKDIR /app
COPY . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction clean package

FROM eclipse-temurin:25-jre-alpine
WORKDIR /app
COPY --from=builder /app/neurowatch/target/neurowatch-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```
````

<br/>

<v-click> Size: 301MB (-70%) </v-click>

---
layout: cover
---

## Great, there's not much left to assess.
### How to tell the real threats and mere spooks apart?

---
layout: cover
---

## Step 3: Divide and conquer
### Goal: Prioritize exploitable risk with context-aware scanning, not CVE volume.

---

## Scan results for our demo
<br/>

```bash {maxHeight:'250px'}

┌──────────────────────────────────────────────┬────────┬─────────────────┬─────────┐
│                    Target                    │  Type  │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ neurowatch-neurowatch:latest (alpine 3.23.3) │ alpine │        3        │    -    │
├──────────────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ app/app.jar                                  │  jar   │        4        │    -    │
└──────────────────────────────────────────────┴────────┴─────────────────┴─────────┘

```

---

## Scan results for our demo
<br/>

```bash {maxHeight:'250px'}
neurowatch:latest (alpine 3.23.3)
Total: 3 (UNKNOWN: 0, LOW: 0, MEDIUM: 1, HIGH: 1, CRITICAL: 1)
┌─────────┬────────────────┬──────────┬────────┬───────────────────┬───────────────┬─────────────────────────────────────────────────────────────┐
│ Library │ Vulnerability  │ Severity │ Status │ Installed Version │ Fixed Version │                            Title                            │
├─────────┼────────────────┼──────────┼────────┼───────────────────┼───────────────┼─────────────────────────────────────────────────────────────┤
│ libpng  │ CVE-2026-25646 │ HIGH     │ fixed  │ 1.6.54-r0         │ 1.6.55-r0     │ libpng: LIBPNG has a heap buffer overflow in                │
│         │                │          │        │                   │               │ png_set_quantize                                            │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2026-25646                  │
├─────────┼────────────────┼──────────┤        ├───────────────────┼───────────────┼─────────────────────────────────────────────────────────────┤
│ zlib    │ CVE-2026-22184 │ CRITICAL │        │ 1.3.1-r2          │ 1.3.2-r0      │ zlib: zlib: Arbitrary code execution via buffer overflow in │
│         │                │          │        │                   │               │ untgz utility                                               │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2026-22184                  │
│         ├────────────────┼──────────┤        │                   │               ├─────────────────────────────────────────────────────────────┤
│         │ CVE-2026-27171 │ MEDIUM   │        │                   │               │ zlib: zlib: Denial of Service via infinite loop in CRC32    │
│         │                │          │        │                   │               │ combine functions...                                        │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2026-27171                  │
└─────────┴────────────────┴──────────┴────────┴───────────────────┴───────────────┴─────────────────────────────────────────────────────────────┘

```

---

## Scan results for our demo
<br/>

```bash
Java (jar)
Total: 4 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 4, CRITICAL: 0)
┌───────────────────────────────────────────────────┬─────────────────────┬──────────┬────────┬───────────────────┬───────────────────────┬──────────────────────────────────────────────────────────────┐
│                      Library                      │    Vulnerability    │ Severity │ Status │ Installed Version │     Fixed Version     │                            Title                             │
├───────────────────────────────────────────────────┼─────────────────────┼──────────┼────────┼───────────────────┼───────────────────────┼──────────────────────────────────────────────────────────────┤
│ com.fasterxml.jackson.core:jackson-core (app.jar) │ GHSA-72hv-8253-57qq │ HIGH     │ fixed  │ 2.20.2            │ 2.18.6, 2.21.1, 3.1.0 │ jackson-core: Number Length Constraint Bypass in Async       │
│                                                   │                     │          │        │                   │                       │ Parser Leads to Potential DoS...                             │
│                                                   │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-72hv-8253-57qq            │
├───────────────────────────────────────────────────┼─────────────────────┤          │        ├───────────────────┼───────────────────────┼──────────────────────────────────────────────────────────────┤
│ org.yaml:snakeyaml (app.jar)                      │ CVE-2022-1471       │          │        │ 1.33              │ 2.0                   │ SnakeYaml: Constructor Deserialization Remote Code Execution │
│                                                   │                     │          │        │                   │                       │ https://avd.aquasec.com/nvd/cve-2022-1471                    │
├───────────────────────────────────────────────────┼─────────────────────┤          │        ├───────────────────┼───────────────────────┼──────────────────────────────────────────────────────────────┤
│ tools.jackson.core:jackson-core (app.jar)         │ CVE-2026-29062      │          │        │ 3.0.3             │ 3.1.0                 │ jackson-core: jackson-core: Denial of Service via excessive  │
│                                                   │                     │          │        │                   │                       │ JSON nesting                                                 │
│                                                   │                     │          │        │                   │                       │ https://avd.aquasec.com/nvd/cve-2026-29062                   │
│                                                   ├─────────────────────┤          │        │                   ├───────────────────────┼──────────────────────────────────────────────────────────────┤
│                                                   │ GHSA-72hv-8253-57qq │          │        │                   │ 3.1.0, 2.21.1, 2.18.6 │ jackson-core: Number Length Constraint Bypass in Async       │
│                                                   │                     │          │        │                   │                       │ Parser Leads to Potential DoS...                             │
│                                                   │                     │          │        │                   │                       │ https://github.com/advisories/GHSA-72hv-8253-57qq            │
└───────────────────────────────────────────────────┴─────────────────────┴──────────┴────────┴───────────────────┴───────────────────────┴──────────────────────────────────────────────────────────────┘

```

---

## Step 3.1: Start with CVSS Scores
<br/>

CVSS - Common Vulnerability Scoring System (0.0–10.0)<br/>
A quick estimate of how bad could this be in a generic environment.

### What CVSS tells you

- Potential impact + ease of exploitation (in general)

<br/>

### What CVSS does not tell you

- Whether your service is affected
- Whether the vulnerable code is reachable
- Whether a patch exists

---
layout: image
image: /risk_model.svg
---

---

## Step 3.2: Do not live on CVSS alone

Use CVSS plus context:

- Which component is affected?
- Is it exploitable in this workload?
- Does a patch exist?
- Is it externally exposed?
- What is the blast radius?
- CVSS score / severity


---

## Example risk model

<br/>

<table>
  <thead>
    <tr>
      <th>CVE</th>
      <th>Affected component</th>
      <th>Exploitability</th>
      <th>Patch availability</th>
      <th>External exposure</th>
      <th>Blast radius</th>
      <th>CVSS</th>
      <th>Triage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        CVE ID 
      </td>
      <td>
        base /<br/> app dependency
      </td>
      <td>
        reachable /<br/> runtime path /<br/> attack preconditions
      </td>
      <td>
        yes / no
      </td>
      <td>
        internet-facing /<br/> internal
      </td>
      <td>
        single service /<br/> shared platform /<br/> lateral movement potential
      </td>
      <td>
severity signal
</td>
      <td>
patch now /<br/> with next update /<br/> exception
</td>
    </tr>
  </tbody>
</table>

Output:<br/>
A risk tier you can act on quickly

---
layout: image
image: /triage_outcomes.svg
---

---
layout: two-cols-header
---

## We found the bombs, let's get rid of them!

::left::

### App CVEs

- Update the deps

::right::

### Base CVEs

- Update the base image
<v-click>...and pray</v-click>


---

## We pulled the fresh base, let's scan it
<br/>

```bash {all|1,2|all}{maxHeight:'350px'}
eclipse-temurin:25-jre-alpine (alpine 3.23.3)
Total: 3 (UNKNOWN: 0, LOW: 0, MEDIUM: 1, HIGH: 1, CRITICAL: 1)
┌─────────┬────────────────┬──────────┬────────┬───────────────────┬───────────────┬─────────────────────────────────────────────────────────────┐
│ Library │ Vulnerability  │ Severity │ Status │ Installed Version │ Fixed Version │                            Title                            │
├─────────┼────────────────┼──────────┼────────┼───────────────────┼───────────────┼─────────────────────────────────────────────────────────────┤
│ libpng  │ CVE-2026-25646 │ HIGH     │ fixed  │ 1.6.54-r0         │ 1.6.55-r0     │ libpng: LIBPNG has a heap buffer overflow in                │
│         │                │          │        │                   │               │ png_set_quantize                                            │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2026-25646                  │
├─────────┼────────────────┼──────────┤        ├───────────────────┼───────────────┼─────────────────────────────────────────────────────────────┤
│ zlib    │ CVE-2026-22184 │ CRITICAL │        │ 1.3.1-r2          │ 1.3.2-r0      │ zlib: zlib: Arbitrary code execution via buffer overflow in │
│         │                │          │        │                   │               │ untgz utility                                               │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2026-22184                  │
│         ├────────────────┼──────────┤        │                   │               ├─────────────────────────────────────────────────────────────┤
│         │ CVE-2026-27171 │ MEDIUM   │        │                   │               │ zlib: zlib: Denial of Service via infinite loop in CRC32    │
│         │                │          │        │                   │               │ combine functions...                                        │
│         │                │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2026-27171                  │
└─────────┴────────────────┴──────────┴────────┴───────────────────┴───────────────┴─────────────────────────────────────────────────────────────┘
```

<v-click>The image contains the same CVEs, no updates yet</v-click>

---
layout: statement
---

## We need the base image patch urgently!

---
layout: statement
---

## Wait for a patch?
### No ETA, we don't have time

---
layout: statement
---

## Ask for a patch?
### Community project - no SLA

---
layout: statement
---

## All our efforts led us to the deadlock
### We know everything, but can't do anything

---
layout: statement
---

## There's got to be another way to defuse the bomb...

---
layout: cover
---

## Step 4: Pick a hardened base

### Goal: Delegate the bomb diffusion to pro deminers.

---

## Who said you must complete the mission alone?
<br/>

<div class="center-stage">
<div class="hi-grid">
  <div class="hb">Shift security left</div>
  <div class="equal">≠</div>
  <div class="hb">Become an amateur OS/runtime maintainer</div>
</div>
</div>

<style>
.center-stage{
  display:grid;
  place-items:center;
  min-height: 30vh;
}

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
.equal{
  text-align:center; font-size: 38px; line-height:1; font-weight: 700;
  color:#00e6ff; opacity:0.95;
}
</style>

---
layout: two-cols-header
---

## Hardened images: new container security standard

<br/>

<div class="hi-grid">
  <div class="hb">Low to zero CVEs by design</div>
  <div class="arrow">➡️</div>
  <div class="hb">Safer base from the start</div>

  <div class="hb">No package manager, minimalistic base</div>
  <div class="arrow">➡️</div>
  <div class="hb">Minimized attack surface</div>

  <div class="hb">Focus on continuous CVE patching</div>
  <div class="arrow">➡️</div>
  <div class="hb">The image STAYS safe</div>
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

## Hardened images: Path to Zero Unmanaged Risk
<br/>

Low-to-zero CVEs by design is great, but hardened images are not only about that.
<br/>

<div class="hi-grid">
  <div class="hb">Trusted</div>
  <div class="arrow">➡️</div>
  <div class="hb">Vendor is committed to patching the image. SLAs available</div>

  <div class="hb">Transparent</div>
  <div class="arrow">➡️</div>
  <div class="hb">SBOM provided. You know what's in the image</div>

  <div class="hb">Verifiable</div>
  <div class="arrow">➡️</div>
  <div class="hb">Provenance provided. You know who built the image</div>
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
layout: cover
---

## SBOM? Provenance? ...What?
### Looks like we got cool new tools for our mission :)<br/> No worries, I'll show you how to use them in a minute.

---
layout: cover
---

## But first, let's choose a base.

---

## Hardened images variety: How to choose

- Several vendors, including BellSoft, Chainguard, Docker
- Look for:
    - OS security, packaging, and compliance expertise
    - Signed images, standard attestations, SBOMs
    - SLA for patches
    - OS and runtime built from source in every image


---
layout: image-right
image: /alpaquita.jpg
---

## BellSoft's Hardened Images: Based on Alpaquita Linux

- LTS releases
- Two libc flavors: optimized musl and glibc
- Runtime, OS, and container security from one team
- SLA-backed proactive patching from in-house Linux and Java experts
- Free to use + commercial support

---

## New Dockerfile

```dockerfile {none|1,7}
FROM bellsoft/hardened-liberica-runtime-container:jdk-25-glibc as builder

WORKDIR /app
ADD my-app /app/my-app
RUN cd my-app && ./mvnw -Pproduction package

FROM bellsoft/hardened-liberica-runtime-container:jre-25-glibc

WORKDIR /app
COPY --from=builder /app/my-app/target/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

<v-click>Image size: 219MB</v-click>

---

### Let's scan it to make sure we are on the right way

```bash {all|7}
osv-scanner scan image bellsoft/hardened-liberica-runtime-container:jre-25-glibc

Scanning image "bellsoft/hardened-liberica-runtime-container:jre-25-glibc"
Starting filesystem walk for root: 
End status: 150 dirs visited, 854 inodes visited, 202 Extract calls, 9.993542ms elapsed, 9.993ms wall time

No issues found
```

<v-click at="1">Zero CVEs!</v-click>

---
layout: cover
---

## Let's use the shiny new gear to set up the checks for what gets into the image

---
layout: cover
---

## Step 5: Ensure provenance
### Goal: Establish trust with verifiable image provenance.

---

## Step 5: Ensure provenance

Provenance = verifiable supply chain evidence for:

- Base image authenticity

> Did this really come from the expected publisher?

- App contents

> What dependencies and artifacts are inside?

- Build origin

> Who and what built this image?

---
layout: image
image: /pipeline.svg
---

---

## Step 5.0: Pin the base image by digest

...Because tags can drift

```bash
#base image version: bellsoft/hardened-liberica-runtime-container:jre-25-glibc
FROM bellsoft/liberica-runtime-container:sha256:58f09e3e991a4588b413394cc0400b03118b492927d413c28b872e6f57cbf6fb
```

---

## Step 5.1: Verify the base image
<br/>

Hardened images come with
- Attestation data proving their origin,
- Software Bill of Materials listing the artifacts in the image with versions

---

## Step 5.1: Verify the base image
<br/>

If the foundation is fake, everything built on it is compromised!<br>

Verify signature

```bash
$ cosign verify \
--key cosign.pub \
<image-uri>@sha256:<digest>
```

Verify attestation and get the SBOM:

```bash
$ cosign verify-attestation \
    --key cosign.pub \
    --type cyclonedx \
    $IMG_TTL | jq -r '.payload' | base64 -d | jq -r '.predicate'
```

---

## Step 3.2: Generate an SBOM for your app image

Produce an SBOM for the application<br>
Store it together with the base image SBOM

💡Separation of concerns: OS packages VS application dependencies

> Am I affected?<br>Is this CVE in base image or app deps?<br>What changed between builds?

---

## Step 3.2: Generate an SBOM for your app image

At build time

```bash
mvn -DincludeCompileScope=true \
  -DincludeRuntimeScope=true \
  -DincludeTestScope=false \
  -DincludeProvidedScope=false \
  org.cyclonedx:cyclonedx-maven-plugin:makeBom
```

---

## Step 3.2: Generate an SBOM for your app image

Can add a plugin

```xml {none|1-3|8,11|15-19|22-24}{maxHeight:'250px'}
<plugin>
    <groupId>org.cyclonedx</groupId>
    <artifactId>cyclonedx-maven-plugin</artifactId>
    <version>2.9.1</version>
    <executions>
        <execution>
            <id>make-app-sbom</id>
            <phase>package</phase>
            <goals>
                <!-- Single-module: makeBom ; multi-module app: makeAggregateBom -->
                <goal>makeBom</goal>
            </goals>
            <configuration>
                <!-- App/runtime-focused scopes -->
                <includeCompileScope>true</includeCompileScope>
                <includeRuntimeScope>true</includeRuntimeScope>
                <includeTestScope>false</includeTestScope>
                <includeProvidedScope>false</includeProvidedScope>
                <includeSystemScope>false</includeSystemScope>

                <!-- Output -->
                <outputFormat>json</outputFormat>
                <outputName>app-sbom</outputName>
                <outputDirectory>${project.build.directory}</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

---

## Step 3.3: Store evidence where you can retrieve it fast

Store attestations and SBOMs in:

- OCI registry (attached to image)
- and/or artifact store for indexing/search


---

## Step 3.4: Scan SBOMs
<br/>

We have SBOMs = we scan what we actually built<br>

- SBOM scan is fast and reproducible
- Works well for rescans when new advisories appear
- Keeps security checks tied to a specific image digest/build

---

## Step 3.4: Scan SBOMs

Scan application SBOM:

```bash
osv-scanner -L target/app-sbom.cdx.json --output app-scan-results.json
```

Scan base image SBOM:

```bash
osv-scanner -L target/app-sbom.cdx.json --output base-scan-results.json
```

Classify the risks as discussed in Step 2

---
layout: cover
---

## We are safe!
### ...For now. How do we KEEP the room safe?

---
layout: cover
---

## Step 6: Enable automated updates monitoring
### Goal: Sustain security with automated base update monitoring and rebuild triggers.

---

## Step 6: Enable automated updates monitoring

What to monitor (automatically):

- Base image updates (container/Dockerfile)
- Application dependencies (Maven/Gradle)

---

## Step 6: Enable automated updates monitoring

Use Renovate (or equivalent) to:

- watch dependency and container updates
- raise PRs automatically
- trigger CI rebuild / rescan workflows


---

## Example: setting up base updates monitoring with Renovate

```json {all|7|15,16}
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended"
  ],
  "enabledManagers": ["dockerfile", "docker-compose"],
  "schedule": ["every weekend"],
  "packageRules": [
    {
      "matchDatasources": ["docker"],
      "matchUpdateTypes": ["major"],
      "enabled": false
    },
    {
      "matchDatasources": ["docker"],
      "matchUpdateTypes": ["minor", "patch", "digest"]
    }
  ]
}
```

---

## Use Renovate and Co. Sensibly
<br/>

These tools DO NOT monitor CVEs. They just look for updates
- Do not accept PRs blindly
- Scan and test after PR: got better or worse?
- Accept the PR if necessary

<br/>

<v-click>The robot gathers data - humans make a decision</v-click>
 

---

## Safe rollout strategy

When updating, every dependent service should automatically:

- Rebuild application image on new base
- Regenerate SBOM + provenance
- Sign / attest the new image
- Rescan
- Push by digest
- Roll out through controlled deployment


---
layout: image
image: /good_room.jpeg
---

---
layout: image
image: /good_room.jpeg
---

## Quick summary?

---

## Building the foundation for vulnerability management is not that hard

Five first important steps

- Migrate to a hardened base: start safe
- Shrink privileges, run as non-root: minimize blast radius
- Generate an SBOM: become aware
- Scan SBOMs: stay aware
- Set up updates monitoring: stay safe


---
layout: image-right
image: "/qr-cyber.svg"
---

## Thank you for your attention!

<br/>

- BlueSky: @edelveis.dev
- X: cat_edelveis
- LinkedIn: cat-edelveis
- YouTube: @cbrjar
- bell-sw.com