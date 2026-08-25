# DevSecOps CI/CD Pipeline Lab

A containerized Flask API secured and validated through a GitHub Actions CI/CD pipeline. The project demonstrates practical DevSecOps controls, including Docker image builds, automated health checks, OWASP ZAP baseline scanning, security-header hardening, and scan-report artifacts.

## Project Goals

- Build and run a Python Flask application in Docker
- Automate CI/CD tasks with GitHub Actions
- Run the application with Gunicorn instead of Flask's development server
- Perform automated dynamic application security testing (DAST) with OWASP ZAP
- Review findings, remediate security issues, and rerun the scan
- Preserve ZAP scan output as a pipeline artifact

## Architecture

```text
Developer Push
      |
      v
GitHub Actions Workflow
      |
      +--> Build Docker image
      |
      +--> Start Flask API with Gunicorn
      |
      +--> Health check: /health
      |
      +--> OWASP ZAP baseline scan
      |
      +--> Upload ZAP report artifact
```

## Technologies Used

- Python 3.11
- Flask
- Gunicorn
- Docker
- GitHub Actions
- OWASP ZAP
- curl

## Application Endpoints

| Endpoint | Method | Description |
|---|---|---|
| `/` | GET | Returns the lab application status |
| `/health` | GET | Health-check endpoint used by the CI/CD pipeline |

Example response:

```json
{
  "message": "DevSecOps Pipeline Lab",
  "status": "running"
}
```

Health-check response:

```json
{
  "status": "healthy"
}
```

## Security Controls

The application applies HTTP response security headers to every response:

| Control | Configuration | Purpose |
|---|---|---|
| Content Security Policy | `default-src 'self'` | Restricts browser resource loading to trusted same-origin sources |
| X-Content-Type-Options | `nosniff` | Prevents MIME-type sniffing |
| Cross-Origin-Resource-Policy | `same-origin` | Restricts cross-origin resource loading |
| Permissions-Policy | Disables camera, microphone, geolocation, payment, and USB | Limits unused browser capabilities |
| Referrer-Policy | `strict-origin-when-cross-origin` | Reduces referrer information shared with other sites |
| X-Frame-Options | `DENY` | Helps prevent clickjacking through framing |
| Cache-Control | `no-store` | Prevents API responses from being stored in browser or proxy caches |
| Pragma | `no-cache` | Provides legacy cache-control support |

The container uses Gunicorn to serve the Flask application rather than Flask's built-in Werkzeug development server.

## Local Setup

### Prerequisites

- Docker Desktop
- Git
- A terminal
- Python 3.11 or later, if running the application outside Docker

### Clone the repository

```bash
git clone https://github.com/nmabeya18/devsecops-pipeline-lab.git
cd devsecops-pipeline-lab
```

### Build the Docker image

```bash
docker build -t secure-flask-app .
```

### Run the container

```bash
docker run -d --name flask-app -p 5001:5001 secure-flask-app
```

### Test the application

```bash
curl http://localhost:5001/
```

Expected output:

```json
{"message":"DevSecOps Pipeline Lab","status":"running"}
```

### Test the health endpoint

```bash
curl http://localhost:5001/health
```

Expected output:

```json
{"status":"healthy"}
```

### Verify security headers

```bash
curl -I http://localhost:5001/
```

Expected headers include:

```text
Content-Security-Policy: default-src 'self'; ...
X-Content-Type-Options: nosniff
Cross-Origin-Resource-Policy: same-origin
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=()
Referrer-Policy: strict-origin-when-cross-origin
X-Frame-Options: DENY
Cache-Control: no-store
Pragma: no-cache
```

### Stop and remove the container

```bash
docker stop flask-app
docker rm flask-app
```

## CI/CD Pipeline

The GitHub Actions workflow runs on repository pushes and automates the following stages:

1. Checks out the repository code
2. Builds the Docker image
3. Starts the containerized Flask API
4. Verifies the `/health` endpoint
5. Runs an OWASP ZAP baseline scan against the running application
6. Generates and uploads the ZAP report as a GitHub Actions artifact

## OWASP ZAP Results

OWASP ZAP baseline scanning was used to identify web-application security misconfigurations.

### Initial findings

The initial scan identified:

- Missing Content Security Policy header
- Missing `X-Content-Type-Options` header
- Missing Cross-Origin-Resource-Policy header
- Missing Permissions-Policy header
- Server version disclosure from the Flask/Werkzeug development server
- Cacheable HTTP responses

### Remediation

The following actions were taken:

- Added a restrictive Content Security Policy without `unsafe-inline`
- Added browser security headers through Flask's `@app.after_request` handler
- Added `Cache-Control: no-store` and `Pragma: no-cache`
- Changed the container startup command from `python app.py` to Gunicorn
- Rebuilt the Docker image and reran OWASP ZAP

### Final results

| Severity | Count | Status |
|---|---:|---|
| High | 0 | No findings |
| Medium | 0 | No findings |
| Low | 0 | No findings |
| Informational | 2 | Reviewed and accepted |

The two remaining informational results are **Non-Storable Content** findings for the API endpoints. They are expected because the application intentionally uses `Cache-Control: no-store` to prevent cached responses.

## Project Structure

```text
devsecops-pipeline-lab/
├── .github/
│   └── workflows/
│       └── ci.yml
├── app.py
├── Dockerfile
├── requirements.txt
└── README.md
```

## Lessons Learned

- Security testing is most useful when it is integrated early into CI/CD rather than performed only after deployment.
- OWASP ZAP can identify missing security headers and insecure caching behavior through automated DAST scanning.
- Security findings should be reviewed, remediated when appropriate, and validated with a follow-up scan.
- Flask's built-in development server is not appropriate for deployed applications; Gunicorn provides a production-oriented WSGI server for the container.
- HTTP security headers and caching rules provide defense-in-depth for even a small API service.

## Future Improvements

- Add automated unit tests with `pytest`
- Add dependency scanning with `pip-audit` or Snyk
- Add secret scanning with GitHub Secret Protection or Gitleaks
- Add image vulnerability scanning with Trivy
- Deploy the container to AWS ECS, AWS App Runner, or another cloud platform
- Store ZAP reports in centralized security monitoring or artifact storage
- Add infrastructure-as-code with Terraform

## Author

Nivea Mabeya

Aspiring Cybersecurity / DevSecOps Professional  
Dallas-Fort Worth, Texas

(https://www.linkedin.com/in/nivea-mabeya-429242251) · (https://github.com/nmabeya18)
