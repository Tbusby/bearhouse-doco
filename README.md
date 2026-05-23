# bearhouse-docs

Documentation repository for a production-shaped homelab platform focused on learning and demonstrating modern DevOps, SRE, and platform engineering practices.

## Purpose

This repository contains the documentation for a homelab environment built to use industry-standard tooling and workflows where practical.

The goal is to treat the lab as a production-like platform:
- designed intentionally
- automated where possible
- documented thoroughly
- backed up and recoverable
- useful for hands-on learning and portfolio development

## Repo Contents

This repository includes:

- architecture documentation
- infrastructure design
- Kubernetes platform design
- operations documentation
- backup and recovery notes
- runbooks
- standards
- architecture decision records (ADRs)

## Local Development

### Requirements

- Python 3
- `pip`
- MkDocs Material

### Setup

Create a virtual environment and install dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install mkdocs-material pymdown-extensions