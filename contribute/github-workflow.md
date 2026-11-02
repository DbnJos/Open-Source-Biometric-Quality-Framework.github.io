---
title: GitHub Workflow
parent: Contribute
nav_order: 3
permalink: /contribute/github-workflow/
layout: default
---

# GitHub Workflow

## 1. Fork and clone

Fork the repository on GitHub, then clone your fork:

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
git remote add upstream https://github.com/Open-Source-Biometric-Quality-Framework/<repo-name>.git
```

## 2. Create a branch

Always branch from the latest `main`:

```bash
git fetch upstream
git checkout -b <branch-name> upstream/main
```

Use a descriptive name, e.g. `fix/fingerprint-threshold` or `docs/install-guide`.

## 3. Make your changes and sign off

All commits must be signed off under the [Developer Certificate of Origin](https://developercertificate.org/):

```bash
git add .
git commit -s -m "Describe your change"
```

## 4. Push to your fork

```bash
git push -u origin <branch-name>
```

## 5. Open a pull request

On GitHub, open a pull request from your branch to `main` in the OpenBQ repository. Describe what changed and why, and link any related issues. A maintainer will review it and may request changes before merging.
