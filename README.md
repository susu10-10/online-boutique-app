# Online Boutique: Application Repository

[![PR Scan Checks](https://github.com/susu10-10/online-boutique-app/actions/workflows/pr-checks.yml/badge.svg)](https://github.com/susu10-10/online-boutique-app/actions/workflows/pr-checks.yml)
[![Build and Push Boutique Containers](https://github.com/susu10-10/online-boutique-app/actions/workflows/ci-main.yml/badge.svg)](https://github.com/susu10-10/online-boutique-app/actions/workflows/ci-main.yml)
[![Build and Push Boutique Images to ECR](https://github.com/susu10-10/online-boutique-app/actions/workflows/ci-ecr.yml/badge.svg)](https://github.com/susu10-10/online-boutique-app/actions/workflows/ci-ecr.yml)


An **11-microservice e-commerce platform** with a fully automated, security-hardened CI/CD pipeline. This repository holds the **application source and the pipeline** that produces signed, immutable container images, then hands them to a GitOps platform (ArgoCD) for deployment by updating the kustomization file.

```
Developer push ──► Unit tests ──► Build ──► Trivy scan ──► Cosign sign ──► Push to DOCR ──► GitOps deploy
```

> **This is the app half of a two-repo architecture.** The platform half lives in [`online-boutique-doks-pf`](https://github.com/susu10-10/online-boutique-doks-pf): Terraform-provisioned DOKS, ArgoCD GitOps, and Kyverno policy enforcement, including verification of the signatures this pipeline creates.

---

## Contents
- [Design Decisions](#-design-decisions)
- [Highlights](#highlights)
- [Repository Layout](#repository-layout)
- [Documentation](#documentation)
- [Technologies](#technologies)
- [Local Development](#local-development)

---

## Design Decisions

**Immutable SHA tags, `latest` banned.** Every deployed image traces back to an exact commit, there is no ambiguity about what's actually running in production, and no risk of a `latest` tag silently changing underneath a running deployment.

**Scan → sign → push.** Nothing gets a Cosign signature until Trivy has cleared it.

**CI does not touch the cluster.** This repo's pipeline ends the moment it writes a signed tag. Deployment is entirely on the [`online-boutique-doks-pf`](https://github.com/susu10-10/online-boutique-doks-pf)'s job via GitOps, and keeping that boundary strict is what makes the two-repo split actually mean something, instead of just being organizational.

---

## Highlights

- **11 microservices, 5 languages** (Go, C#, Node.js, Python, Java) communicating over gRPC
- **Immutable SHA tags.** Every image is traceable to the exact commit. `latest` is banned in production.
- **Scan → Sign → Push ordering.** Nothing is signed until Trivy says it's clean. Nothing is pushed unsigned.
- **Cosign keypair signing.** Private key in CI secrets; public key embedded in the cluster's Kyverno policy.
- **PR security gate.** Semgrep (OWASP), TruffleHog, Trivy FS, Hadolint, and unit tests must all pass before merge.
- **Hardened containers.** Non-root, read-only rootfs, all capabilities dropped, resource limits on every service.
- **Pure GitOps handoff.** CI never touches the cluster. It writes one tag file; ArgoCD does the rest.

---

## Repository Layout

```
online-boutique-app/
├── src/                          # 11 microservice source trees
├── container-images/
│   └── docker-compose.yml        # Single hardened build definition for all services
├── .github/workflows/
│   ├── pr-checks.yml             # PR gate: unit tests + security scans
│   ├── security-essential.yml    # Reusable security workflow
│   └── ci-main.yml               # Main pipeline: build → scan → sign → push → GitOps handoff
└── docs/                         # Full documentation
```

## Documentation

| Document | What you'll learn |
|----------|-------------------|
| [Architecture](docs/architecture.md) | Two-repo GitOps design, service graph, tech stack |
| [CI/CD Pipeline](docs/ci-cd-pipeline.md) | Every stage of the pipeline, command by command |
| [Security](docs/security.md) | PR gate → scan → sign → verify |
| [Services](docs/services.md) | All 11 microservices: language, port, dependencies |

## Technologies

| Area | Stack |
|------|-------|
| Languages | Go · C# · Node.js · Python · Java |
| Communication | gRPC / protobuf |
| CI/CD | GitHub Actions |
| Registry | DigitalOcean Container Registry |
| Scanning | Trivy · Semgrep · TruffleHog · Hadolint |
| Signing | Cosign (keypair) |
| Deploy target | DOKS + ArgoCD + Kyverno (infra repo) |

## Local Development

```bash
# Build all 11 service images
docker compose -f container-images/docker-compose.yml build

# Run the full stack (Caddy on :80/:443)
docker compose -f container-images/docker-compose.yml up -d

# Simulate traffic (10 users, 1 req/s)
docker compose -f container-images/docker-compose.yml --profile load-test run loadgenerator
```

> **Part of a series exploring DevSecOps patterns across platforms:**
> [`online-boutique-pf`](https://github.com/susu10-10/online-boutique-pf) · [`online-boutique-doks-pf`](https://github.com/susu10-10/online-boutique-doks-pf) · [`k8s-3tier-automation`](https://github.com/susu10-10/k8s-3tier-automation) · [`3tier-k8s-Hardening`](https://github.com/susu10-10/3tier-k8s-Hardening)
