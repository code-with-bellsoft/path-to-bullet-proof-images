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

## Initial Dockerfile

```dockerfile {none|1|4|6|8,9,10}{maxHeight:'250px'}
FROM eclipse-temurin:25-jdk as builder
WORKDIR /app
COPY . /app/neurowatch
RUN cd neurowatch && ./mvnw -Pproduction package

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

```bash {all|2|all}

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
│ app/app.jar                                 │  jar   │        1        │    -    │
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

## Apart from that...

- package manager,
- runs as root,
- random base,
- tons of packages,
- CVE noise,
- No provenance,
- irregular updates.

---

## Time is running out, what's the plan?

Mission objective:<br><br>
Low-noise, locked-down Java container on a hardened base with zero unmanaged risk

Rules:<br><br>
No chasing after every CVE<br>

---

## Ok, time is running out, what's the plan?

Actions:<br><br>
Harden the base<br>
Shrink privileges<br>
Generate provenance<br>
Scan<br>

---
layout: cover
---

## Step 1: Pick a hardened base

### Goal: Reduce attack surface with a hardened, minimal base image.

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

## Hardened images variety: How to choose

- Several vendors, including BellSoft, Chainguard, Docker
- Look for:
    - OS security, packaging, and compliance expertise
    - Signed images, standard attestations, SBOMs
    - SLA for patches
    - OS and runtime built from source in every image

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

## Great, we cleaned up the mess

### How do we stop intruders from using what's left?

---
layout: cover
---

## Step 2: Shrink privileges

### Goal: Reduce blast radius by dropping privileges and locking runtime behavior.

---

## Step 2: Shrink privileges

🤩 Most hardened images run as non-root by default.<br><br>
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
layout: cover
---

## We reduced what intruders can do.

### But how do we know what got into the room in the first place?

---
layout: cover
---

## Step 3: Ensure provenance

### Goal: Establish trust with verifiable image provenance.

---

## Step 3: Ensure provenance

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

## Step 3.0: Pin the base image by digest

...Because tags can drift

```bash
#base image version: bellsoft/hardened-liberica-runtime-container:jre-25-glibc
FROM bellsoft/liberica-runtime-container:sha256:58f09e3e991a4588b413394cc0400b03118b492927d413c28b872e6f57cbf6fb
```

---

## Step 3.1: Verify the base image

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
layout: cover
---

## We found out what’s in the room and where it came from.

### How do we find a bomb if there is one?

---
layout: cover
---

## Step 4: Scan wisely

### Goal: Prioritize exploitable risk with context-aware scanning, not CVE volume.

---

## Step 4: Scan wisely

We have SBOMs = we scan what we actually built<br>

- SBOM scan is fast and reproducible
- Works well for rescans when new advisories appear
- Keeps security checks tied to a specific image digest/build

---

## Step 4.1: Scan SBOMs

Scan application SBOM:

```bash
osv-scanner -L target/app-sbom.cdx.json --output app-scan-results.json
```

Scan base image SBOM:

```bash
osv-scanner -L target/app-sbom.cdx.json --output base-scan-results.json
```

---

## Scan results for our demo

```bash
Total 1 package affected by 1 known vulnerability (0 Critical, 1 High, 0 Medium, 0 Low, 0 Unknown) from 1 ecosystem.
1 vulnerability can be fixed.


+-------------------------------------+------+-----------+--------------------+---------+---------------+--------------------------+
| OSV URL                             | CVSS | ECOSYSTEM | PACKAGE            | VERSION | FIXED VERSION | SOURCE                   |
+-------------------------------------+------+-----------+--------------------+---------+---------------+--------------------------+
| https://osv.dev/GHSA-mjmj-j48q-9wg2 | 8.3  | Maven     | org.yaml:snakeyaml | 1.33    | 2.0           | target/app-sbom.cdx.json |
+-------------------------------------+------+-----------+--------------------+---------+---------------+--------------------------+

```

<v-click>This is all very well, but what do we do with that?🧐</v-click>

---
layout: image
image: /risk_model.svg
---

---

## Step 4.3: Do not live on CVSS alone

Use CVSS plus context:

- Which component is affected?
- Is it exploitable in this workload?
- Does a patch exist?
- Is it externally exposed?
- What is the blast radius?
- CVSS score / severity

---

## Step 4.4: Example risk model

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

## Example risk classification for our demo

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
        CVE-2022-1471 
      </td>
      <td>
        App dependency (direct)
      </td>
      <td>
        Reachable via admin YAML import flow;<br />
        Potential RCE
      </td>
      <td>
        Yes.<br />
        Upgrade to snakeyaml 2.0+
      </td>
      <td>
        Network
      </td>
      <td>
        Single service compromise;<br />
        Lateral movement potential
        (secrets/DB/internal)
      </td>
      <td>8.3</td>
      <td>Patch now</td>
    </tr>
  </tbody>
</table>

---
layout: cover
---

## We are safe!

### ...For now. How do we KEEP the room safe?

---
layout: cover
---

## Step 5: Enable automated updates monitoring

### Goal: Sustain security with automated base update monitoring and rebuild triggers.

---

## Step 5: Enable automated updates monitoring

What to monitor (automatically):

- Base image updates (container/Dockerfile)
- Application dependencies (Maven/Gradle)
- Security advisories (especially actively exploited / high-impact)

---

## Step 5: Enable automated updates monitoring

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

## Safe rollout strategy

When the base image updates, every dependent service should automatically:

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

Four first important steps

- Migrate to a hardened base: start safe
- Generate an SBOM: become aware
- Scan SBOMs: stay aware
- Set up updates monitoring: stay safe

---

## Thank you for your attention!

<br/>

- BlueSky: @edelveis.dev
- X: cat_edelveis
- LinkedIn: cat-edelveis
- YouTube: @cbrjar
- bell-sw.com