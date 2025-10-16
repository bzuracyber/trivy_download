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
