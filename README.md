# container-security-foundations

# Container Security Foundations

## Project Overview

This project demonstrates the fundamentals of securing a Docker container.

The goal was to build an intentionally weak container image, identify security issues, harden the image, scan it for vulnerabilities, and apply runtime security controls.

The project focuses on practical container security techniques commonly used in DevSecOps environments.

## Technologies Used

- Docker
- Trivy
- Python
- Flask
- CycloneDX SBOM

## What I Learned

During this project, I practiced:

- Building Docker container images
- Identifying insecure Dockerfile configurations
- Scanning images for vulnerabilities
- Scanning repositories for secrets
- Running containers as a non-root user
- Using minimal base images
- Using `.dockerignore`
- Applying read-only filesystem restrictions
- Removing unnecessary Linux capabilities
- Preventing privilege escalation
- Generating a Software Bill of Materials (SBOM)

## Project Structure

```text
container-security-foundations/
│
├── app.py
├── requirements.txt
├── Dockerfile.insecure
├── Dockerfile.secure
├── .dockerignore
└── sbom.cdx.json
```

## Build the Insecure Image

```bash
docker build \
  -f Dockerfile.insecure \
  -t container-security-lab:insecure \
  .
```

## Scan the Insecure Image

```bash
trivy image container-security-lab:insecure
```

Scan only HIGH and CRITICAL vulnerabilities:

```bash
trivy image \
  --severity HIGH,CRITICAL \
  container-security-lab:insecure
```

## Scan the Project

```bash
trivy fs \
  --scanners vuln,misconfig,secret \
  .
```

This checks the project for:

- Vulnerabilities
- Security misconfigurations
- Exposed secrets

## Build the Hardened Image

```bash
docker build \
  --pull \
  --no-cache \
  -f Dockerfile.secure \
  -t container-security-lab:secure \
  .
```

## Verify the Container Runs as Non-Root

```bash
docker run \
  --rm \
  --entrypoint id \
  container-security-lab:secure
```

The container should not show:

```text
uid=0(root)
```

## Scan the Hardened Image

```bash
trivy image \
  --severity HIGH,CRITICAL \
  container-security-lab:secure
```

## Run the Container Securely

When using Git Bash on Windows:

```bash
MSYS_NO_PATHCONV=1 docker run \
  -d \
  --name secure-lab \
  --read-only \
  --tmpfs /tmp \
  --cap-drop ALL \
  --security-opt no-new-privileges:true \
  -p 8080:5000 \
  container-security-lab:secure
```

Security controls used:

- `--read-only` — prevents changes to the container root filesystem
- `--tmpfs /tmp` — allows temporary files only in `/tmp`
- `--cap-drop ALL` — removes Linux capabilities
- `--security-opt no-new-privileges:true` — prevents privilege escalation

## Test the Application

```bash
curl http://localhost:8080
```

Health endpoint:

```bash
curl http://localhost:8080/health
```

## Verify Security Controls

Check the read-only filesystem:

```bash
docker inspect secure-lab \
  --format 'ReadOnly={{.HostConfig.ReadonlyRootfs}}'
```

Check dropped capabilities:

```bash
docker inspect secure-lab \
  --format '{{json .HostConfig.CapDrop}}'
```

Check the running user:

```bash
docker exec secure-lab id
```

## Generate an SBOM

```bash
trivy image \
  --format cyclonedx \
  --output sbom.cdx.json \
  container-security-lab:secure
```

The SBOM provides an inventory of the software components and dependencies contained inside the image.

## Security Improvements Implemented

The final container uses:

- Minimal base image
- Non-root user
- `.dockerignore`
- Vulnerability scanning
- Secret scanning
- Misconfiguration scanning
- Read-only filesystem
- Dropped Linux capabilities
- No privilege escalation
- SBOM generation

## Troubleshooting

### Git Bash `/tmp` Error

If Docker returns:

```text
invalid mount path: 'C' mount path must be absolute
```

Git Bash is converting `/tmp` into a Windows path.

Use:

```bash
MSYS_NO_PATHCONV=1 docker run ...
```

### Container Name Already Exists

```bash
docker rm -f secure-lab
```

Then run the container again.

### Application Fails as Non-Root

Check file permissions and make sure application files belong to the non-root user.

Example:

```dockerfile
COPY --chown=appuser:appgroup app.py .
```

## Cleanup

Remove the running container:

```bash
docker rm -f secure-lab
```

Remove the images if no longer required:

```bash
docker image rm \
  container-security-lab:insecure \
  container-security-lab:secure
```

## Project Outcome

This project provided hands-on experience with securing containers throughout the container lifecycle.

I learned how to identify vulnerabilities, reduce container attack surface, apply least-privilege principles, generate SBOMs, and enforce runtime security controls before containers are used in production environments.