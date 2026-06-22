# Task Plan Summary

## What we built

A 25-task, 6-phase plan to build a Crossplane + ArgoCD + OpenShift Virtualization VM provisioning stack. Total effort: ~28 days (6–8 weeks for a team of 2–3).

## How to use this plan

1. **Break into issues** — each task is a potential GitHub issue or `.scratch/<task-slug>/` directory
2. **Prioritize** — follow the dependency graph; phases 1–2 must complete before 3
3. **Track progress** — update Status: lines in each issue as tasks move through: needs-triage → needs-info → ready-for-agent/ready-for-human → wontfix
4. **Reference explainers** — each task links back to relevant explainer content for technical details

## Quick reference

- **Explainer 1** (`learning-records/explainer-001-xrd-pipeline.html`) — XRD pipeline, field mapping
- **Explainer 2** (`learning-records/explainer-002-composition-functions.html`) — Composition Functions
- **Explainer 3** (`learning-records/explainer-003-provider-installation.html`) — Provider + RBAC
- **Explainer 4** (`learning-records/explainer-004-argocd-bootstrap.html`) — ArgoCD bootstrap
- **Explainer 5** (`learning-records/explainer-005-end-to-end-runbooks.html`) — End-to-end walkthrough, runbooks
- **Explainer 6** (`learning-records/explainer-006-vm-workload-dev.html`) — OS images, cloud-init, guest agents
- **Explainer 7** (`learning-records/explainer-007-xrd-composition-vm-deep-dive.html`) — XRD & Composition deep dive for VM workloads
