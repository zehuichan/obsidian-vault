---
url: "https://www.verdaccio.org/"
clipped: 2026-07-05
title: "Verdaccio — A lightweight Node.js private proxy registry"
---

# Verdaccio

A lightweight Node.js private proxy registry

```bash
npm install --global verdaccio
```

## What is Verdaccio?

Verdaccio is a simple, zero-config-required local private NPM registry. No need for an entire database just to get started. Verdaccio comes out of the box with its own tiny database, and the ability to proxy other registries (eg. npmjs.org), also introduces caching the downloaded modules along the way. For those who are looking to extend their storage capabilities, Verdaccio supports various community-made plugins to hook into services such as Amazon's S3, Google Cloud Storage or create your own plugin.

## Features

### Use private packages

If you want to use all benefits of npm package system in your company without sending all the code to the public, and use your private packages just as easy as public ones.

### Link multiple registries

If you use multiple registries in your organization and need to fetch packages from multiple sources in one single project, you can chain multiple registries and fetch from one single endpoint.

### Cache npmjs.org registry

If you have more than one server which you want to install packages from, you might want to use this to decrease latency and provide limited failover.

### Override public packages

If you want to use a modified version of some 3rd-party package, you can publish your version locally under the same name.

## Popular npm clients are supported

Package managers such as **npm**, **yarn**, and **pnpm** are part of any development workflow.

## Deployment

### Docker

```bash
docker pull verdaccio/verdaccio
```

### Kubernetes Helm

```bash
helm install verdaccio/verdaccio
```

## Making the DevOps work easy

We have an official **Docker** image ready to use and **Kubernetes Helm** support for easy deployment.

## Used by

nx, pnpm, vendure, create-react-app, Angular CLI, aurelia, SheetJs, Storybook, and many more.

## Links

- Docs: Getting Started, Docker, Configuration
- Community: Stack Overflow, Discord, Bluesky
- GitHub: https://github.com/verdaccio/verdaccio
