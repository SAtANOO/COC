# CCO MI Solutions — Security Assessment
## Step-by-step README

**Scope:** Static source-code analysis of OWASP WebGoat with SonarQube Community Build, and third-party dependency analysis of OWASP Juice Shop with OWASP Dependency-Check.


---

## 1. Assessment overview

The assessment covers two official repositories:

- WebGoat: https://github.com/WebGoat/WebGoat
- Juice Shop: https://github.com/juice-shop/juice-shop

The required deliverable is one PDF report containing:
1. VM specifications, OS and tool versions.
2. Commands and Docker volume configuration.
3. Repository commit hashes.
4. SonarQube results and screenshots.
5. Dependency-Check results, severity summary and report evidence.

No remediation is required. The purpose is to document the results accurately and reproducibly.

## 2. Environment

The work was performed on an Ubuntu Linux VM accessed from a Mac through SSH.

| Item | Required / recorded value |
|---|---|
| OS release | `Ubuntu server` |
| vCPUs | 4 cpu |
| RAM   | 8 GB; |
| Disk  | 40 GB;|
| Docker Engine | `29.1.3` |
| Docker Compose | `v2.40.3` |
| SonarQube Community Build | `26.9.0.129388` |
| OWASP Dependency-Check | `12.1.0` |

### 2.1 Verify OS and VM specifications

Run on the Ubuntu VM:

```bash
lsb_release -a
nproc
free -h
df -h /
```

If the VM does not meet the assignment's minimums (4 vCPUs, 8 GB RAM and 40 GB disk), adjust its configuration and document the actual specifications.

### 2.2 Verify Docker versions

```bash
docker --version
docker compose version
docker info
```

Record the version output in the report. If `docker compose version` fails, verify Docker Compose is installed and available.

### 2.3 Configure the Linux kernel setting for SonarQube

The assignment specifies `vm.max_map_count` of at least `524288`.

```bash
sudo sysctl -w vm.max_map_count=524288
sysctl vm.max_map_count
```

The second command should show `vm.max_map_count = 524288` or a higher value. For persistence after reboot, configure the setting through the VM's normal system configuration and verify it after restart if required.

## 3. Clone the official repositories

The repository folders used in this assessment are `~/COC/WebGoat` and `~/COC/juice-shop`.


```bash
mkdir -p ~/COC
git clone https://github.com/WebGoat/WebGoat.git ~/COC/WebGoat
git clone https://github.com/juice-shop/juice-shop.git ~/COC/juice-shop
```

After this 

```bash
cd ~/COC/WebGoat
git status
git rev-parse HEAD

cd ~/COC/juice-shop
git status
git rev-parse HEAD
```

Commit hashes recorded from the provided terminal screenshot:

- **WebGoat:** `10cecda8b93f9ea1439188a6257dfb7fc9c954d5`
- **Juice Shop:** `0e6d909b7466e76bc8deabf2fdb62bc5c849961f`

Verify the values again before submission if the repositories have changed since the analysis.

## 4. SonarQube — WebGoat static analysis

### 4.1 Inspect SonarQube

SonarQube Community Build was run in Docker and accessed through an SSH tunnel. If the container is already running, inspect it before starting another instance:

```bash
docker ps
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Ports}}'
```

Record the actual container/image and volume configuration used. Do not claim a sample configuration is the exact setup if it differs from your actual setup.

### 4.2 Open the SonarQube web interface from your Mac

The recorded service was forwarded from VM loopback port `9000` to port `9010` on the Mac.

On the **Mac terminal**, run:

```bash
ssh -N -L 9010:127.0.0.1:9000 aacharya@172.16.165.197
```

Replace the username or IP if they differ. Keep the terminal open, then visit:

http://localhost:9010

### 4.3 Use an analysis token safely

In SonarQube, create a project token as needed. Store it in an environment variable instead of hard-coding it into the report.

```bash
export SONAR_TOKEN='PASTE_YOUR_TOKEN_IN_THIS_TERMINAL_ONLY'
```


### 4.4 Compile and analyse WebGoat

The successful command used for the recorded analysis was:

```bash
cd ~/COC/WebGoat

./mvnw compile \
  org.sonarsource.scanner.maven:sonar-maven-plugin:5.8.0.7211:sonar \
  -Dsonar.host.url=http://127.0.0.1:9000 \
  -Dsonar.projectKey=WebGoat \
  -Dsonar.token="$SONAR_TOKEN"
```

The terminal screenshots showed `BUILD SUCCESS`, `ANALYSIS SUCCESSFUL`, and successful report upload for project key `WebGoat`.

### 4.5 Review and capture the dashboard


Values visible in the supplied screenshots:

| Metric | Observed value |
|---|---:|
| Lines of code | 15,367 |
| Total issues on overview | 746 |
| Security issues | 94 |
| Reliability issues | 67 |
| Maintainability issues | 620 |
| Security hotspots displayed | 0 |
| Quality gate | Passed |
| Code coverage shown | 0.0% |

A separate issue view showed 29 Blocker, 93 High, 349 Medium, 230 Low and 56 Info. These severity values sum to **757**, which does not match the overview's **746** total issues. Recheck the current dashboard and explain the discrepancy; do not treat both totals as consistent.

The dashboard indicated an analysis warning/limited security analysis. The scanner output reported unresolved imports/types and no JaCoCo coverage report was imported. Document these limitations. Zero displayed hotspots does not prove the absence of security risks.

One screenshot showed a finding in `JWTRefreshEndpoint.java` associated with rule `java:S6437`, classified as Blocker and concerning hard-coded credentials. Do not include actual secrets in the report.

## 5. OWASP Dependency-Check — Juice Shop

### 5.1 Report and data directories

- Source: `~/COC/juice-shop`
- Persistent data: `~/OWASP-Dependency-Check/data`
- Reports: `~/COC/juice-shop/odc-reports`

Create the directories if needed:

```bash
mkdir -p ~/COC/juice-shop/odc-reports
mkdir -p ~/OWASP-Dependency-Check/data
```

### 5.2 Obtain and protect the NVD API key

Request a key from the official NVD page: https://nvd.nist.gov/developers/request-an-api-key

Set it in your shell environment; 

```bash
export NVD_API_KEY='PASTE_YOUR_NVD_API_KEY_IN_THIS_TERMINAL_ONLY'
```


### 5.3 Run Dependency-Check in Docker

```bash
docker run --rm \
  -v "$HOME/COC/juice-shop:/src" \
  -v "$HOME/OWASP-Dependency-Check/data:/usr/share/dependency-check/data" \
  -v "$HOME/COC/juice-shop/odc-reports:/report" \
  docker.io/owasp/dependency-check:12.1.0 \
  --scan /src \
  --project "OWASP Juice Shop" \
  --nvdApiKey "$NVD_API_KEY" \
  --format HTML \
  --format JSON \
  --out /report
```

The data volume keeps the vulnerability database between runs. The generated reports are:

```text
~/COC/juice-shop/odc-reports/dependency-check-report.html
~/COC/juice-shop/odc-reports/dependency-check-report.json
```

### 5.4 Check and view the report files

```bash
cd ~/COC/juice-shop/odc-reports
ls -lh
```

To view them on the Mac through SSH, run this on Ubuntu and keep it running:

```bash
cd ~/COC/juice-shop/odc-reports
python3 -m http.server 8000 --bind 127.0.0.1
```

In a **second terminal on your Mac**:

```bash
ssh -N -L 8000:127.0.0.1:8000 aacharya@172.16.165.197
```

Replace the IP if necessary. Then open:

- HTML: http://localhost:8000/dependency-check-report.html
- JSON: http://localhost:8000/dependency-check-report.json

Stop the HTTP server with `Ctrl+C` when finished. Do not expose the report server publicly.

### 5.5 Verified Dependency-Check totals

The JSON report was checked programmatically:

| Metric | Result |
|---|---:|
| Dependency entries | 35 |
| Vulnerable dependencies | 22 |
| Individual vulnerability entries | 47 |
| Suppressed vulnerabilities | 0 |

Each vulnerable dependency was grouped once, by its highest reported severity:

| Highest severity per vulnerable dependency | Count |
|---|---:|
| Critical | 6 |
| High | 6 |
| Medium | 0 |
| Low | 1 |
| Informational | 0 |
| Unknown | 0 |
| **Total classified by script** | **22** |

The severity table counts vulnerable dependencies, not individual CVEs/advisories. The 47 individual vulnerability entries are reported separately.

### 5.6 Reproduce the severity summary

Run from `~/COC/juice-shop/odc-reports`:

```bash
python3 - <<'PY'
import json
from collections import Counter

with open("dependency-check-report.json", encoding="utf-8") as f:
    report = json.load(f)

severity_order = {
    "CRITICAL": 5,
    "HIGH": 4,
    "MEDIUM": 3,
    "LOW": 2,
    "INFO": 1,
    "INFORMATIONAL": 1,
    "UNKNOWN": 0,
}

counts = Counter()

for dep in report.get("dependencies", []):
    vulns = dep.get("vulnerabilities", [])
    if not vulns:
        continue

    severities = [
        str(v.get("severity", "UNKNOWN")).upper().strip() or "UNKNOWN"
        for v in vulns
    ]
    highest = max(severities, key=lambda s: severity_order.get(s, 0))
    counts[highest] += 1

print("Dependency entries:", len(report.get("dependencies", [])))
print("Dependencies with vulnerability entries:",
      sum(bool(d.get("vulnerabilities")) for d in report.get("dependencies", [])))
print("Individual vulnerability entries:",
      sum(len(d.get("vulnerabilities", [])) for d in report.get("dependencies", [])))

print("\nVulnerable dependencies by highest severity:")
for severity in ["CRITICAL", "HIGH", "MEDIUM", "LOW", "INFO", "INFORMATIONAL", "UNKNOWN"]:
    print(f"{severity}: {counts[severity]}")
print("Total classified:", sum(counts.values()))
PY
```


### 5.7 Example findings and scan limitations

Examples shown in the report included:

- `crypto-js@3.3.0` — Critical highest severity; a detail described a PBKDF2 implementation concern involving SHA-1 and a single iteration.
- `lodash@2.4.2` — Critical highest severity; advisories concerned prototype pollution and command injection in affected versions.
- `@cyclonedx/cyclonedx-npm@4.2.1` — High finding in lockfile data. `npm ls @cyclonedx/cyclonedx-npm` did not show it in the installed dependency tree, so do not automatically describe it as an active, reachable runtime vulnerability.

Scan warnings included:
- Database-processing errors for `CVE-2026-6785` and `CVE-2026-6786` because reference URLs exceeded a database column length.
- `node_modules` was absent although `package-lock.json` was present.
- Archive-processing warnings for encrypted/test ZIP files and paths flagged for zip-slip concerns.
- Supplementary Sonatype/component report requests failed and some alias dependencies were skipped.

The report was generated despite these warnings. Disclose them and do not claim the scan was error-free or that all findings are exploitable.

## 6. Recommended evidence order:

1. VM configuration showing vCPU, RAM and disk.
2. Ubuntu release and Docker/Docker Compose versions.
3. Repository commit hashes for both projects.
4. SonarQube project overview.
5. SonarQube issue list and severity counts.
6. SonarQube detailed example finding, with secrets redacted.
7. Terminal showing WebGoat `BUILD SUCCESS`.
8. Terminal showing SonarQube `ANALYSIS SUCCESSFUL`.
9. Dependency-Check HTML summary.
10. Dependency-Check severity script output.
11. Detailed Dependency-Check findings or report screenshots.

Add a caption to each screenshot and make sure it is readable. Never include tokens, API keys, passwords, or actual secret values.
