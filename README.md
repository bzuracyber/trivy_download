# Trivy Tool Download to copy over
https://github.com/aquasecurity/trivy/releases/tag/v0.67.2

# Docker Desktop Download
https://docs.docker.com/desktop/setup/install/windows-install/

# Redownload
```bash
docker pull ghcr.io/aquasecurity/trivy-db:2
docker save ghcr.io/aquasecurity/trivy-db:2 -o trivy-db-docker.tar
```

# Pull Trivy Vulnerability DBs
```bash
docker pull ghcr.io/aquasecurity/trivy-db:2
docker pull ghcr.io/aquasecurity/trivy-java-db:1
docker pull ghcr.io/aquasecurity/trivy-checks:1
```

# Save them as Tarballs
```bash
docker save ghcr.io/aquasecurity/trivy-db:2 -o trivy-db.tar
docker save ghcr.io/aquasecurity/trivy-java-db:1 -o trivy-java-db.tar
docker save ghcr.io/aquasecurity/trivy-checks:1 -o trivy-checks.tar
```
# Load them into airgapped system
```bash
docker load -i trivy-db.tar
docker load -i trivy-java-db.tar
docker load -i trivy-checks.tar
```
# Run Trivy Scan
```bash
.\trivy image --db-repository ghcr.io/aquasecurity/trivy-db:2 jenkins/jenkins:lts
```
# Export the Running Container’s Filesystem
```bash
docker export <container_id> -o container.tar
```
# Run Trivy on Exported Container's Filesystem
```bash
trivy fs --skip-update --db-repository ghcr.io/aquasecurity/trivy-db:2 container.tar
```
# Import the exported container tarball:
```bash
docker import container.tar chrome:latest
```
# Scan it as an image:
```bash
trivy image --skip-update --db-repository ghcr.io/aquasecurity/trivy-db:2(*THESE ARE VERSION NUMBERS) chrome:latest
```
download the latest WSL update package (wsl_update_x64.msi) directly from Microsoft’s GitHub releases: 👉 https://github.com/microsoft/WSL/releases
_______________________________________________________________________________________________________
Absolutely — let me streamline and polish those steps into a **ready‑to‑copy implementation guide** for your GitLab + Windows Runner + Trivy offline automation. I’ll keep it concise, structured, and directly actionable so you can drop it into your environment without extra editing.

---

# 🚀 GitLab‑Driven Offline Trivy Scanning (Windows Runner)

## 1. Folder Layout (on Windows air‑gapped machine)

```
C:\trivy\
 ├─ trivy.exe               # Trivy binary
 ├─ gitlab-runner.exe       # GitLab Runner binary
 ├─ db\                     # Offline DBs (db, java-db, checks)
 ├─ images\                 # Place exported .tar images here
 ├─ reports\                # Scan outputs (HTML/JSON)
 └─ html.tpl                # Trivy HTML template (copied from contrib)
```

---

## 2. Prepare on Online Machine

```bash
# Download Trivy DBs
trivy --download-db-only --cache-dir ./db
trivy --download-java-db-only --cache-dir ./db
trivy --download-checks-bundle --cache-dir ./db
```

Copy `db/` folder, `trivy.exe`, `gitlab-runner.exe`, and `html.tpl` to USB → transfer to `C:\trivy\` on the air‑gapped machine.

---

## 3. Install & Register GitLab Runner (Windows)

```powershell
cd C:\trivy
.\gitlab-runner.exe install
.\gitlab-runner.exe start

# Register runner
.\gitlab-runner.exe register --non-interactive `
  --url "http://gitlab.local/" `
  --registration-token "PASTE_TOKEN_HERE" `
  --executor "shell" `
  --description "Airgapped-Windows" `
  --tag-list "windows,offline-scan" `
  --run-untagged="true" `
  --locked="false"
```

---

## 4. Export Images on Linux Host

```bash
docker save -o /tmp/jenkins.tar jenkins:latest
docker save -o /tmp/gitlab.tar gitlab:latest
```

Transfer `.tar` files to `C:\trivy\images\`.

---

## 5. Test Trivy Offline (Windows)

```powershell
C:\trivy\trivy.exe image --input C:\trivy\images\jenkins.tar `
  --skip-update --cache-dir C:\trivy\db `
  --format template --template "C:\trivy\html.tpl" `
  -o C:\trivy\reports\jenkins.html
```

---

## 6. GitLab CI Pipeline (`.gitlab-ci.yml`)

```yaml
stages:
  - scan

variables:
  TRIVY_PATH: "C:\\trivy\\trivy.exe"
  IMAGE_DIR: "C:\\trivy\\images"
  REPORT_DIR: "C:\\trivy\\reports"
  DB_DIR: "C:\\trivy\\db"
  TEMPLATE: "C:\\trivy\\html.tpl"

scan_images:
  stage: scan
  tags:
    - windows
    - offline-scan
  script:
    - powershell -Command "if (-not (Test-Path -Path $env:REPORT_DIR)) { New-Item -ItemType Directory -Path $env:REPORT_DIR | Out-Null }"
    - powershell -Command "Get-ChildItem -Path $env:IMAGE_DIR -Filter *.tar | ForEach-Object {
        $image = $_.FullName;
        $name = [System.IO.Path]::GetFileNameWithoutExtension($_.Name);
        Write-Host 'Scanning' $name;
        & $env:TRIVY_PATH image --input $image --skip-update --cache-dir $env:DB_DIR --format json --output (Join-Path $env:REPORT_DIR \"$name.json\");
        & $env:TRIVY_PATH image --input $image --skip-update --cache-dir $env:DB_DIR --format template --template $env:TEMPLATE --output (Join-Path $env:REPORT_DIR \"$name.html\");
        Write-Host \"Completed scanning $name\";
      }"
  artifacts:
    when: always
    paths:
      - reports/
    expire_in: 14 days
```

---

## 7. Trigger Scans

- **Manual**: Run pipeline in GitLab.  
- **Scheduled**: Add a pipeline schedule (nightly/weekly).  
- **On demand**: Drop new `.tar` images into `C:\trivy\images\` and trigger pipeline.

---

## 8. Optional: Fail on High/Critical

Add to each Trivy command:
```powershell
--exit-code 1 --severity HIGH,CRITICAL
```
This will fail the job if serious issues are found.

---

✅ With this setup:
- You sneaker‑net DBs + images into `C:\trivy\`.  
- GitLab orchestrates scans via the Windows runner.  
- Reports (JSON + HTML) are uploaded as artifacts.  
- You can enforce policy by failing builds on high/critical findings.  

---

Daniel, would you like me to also add a **helper PowerShell script** (`scan-all.ps1`) that you can run locally *outside GitLab* for quick ad‑hoc scans, using the same logic as the pipeline? That way you’d have both CI‑driven and manual workflows unified.
