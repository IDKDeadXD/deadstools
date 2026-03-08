# DeadsTools

> A desktop project manager built for Minecraft Bedrock addon developers.

![Electron](https://img.shields.io/badge/Electron-2B2E3A?style=for-the-badge&logo=electron&logoColor=9FEAF9)
![Node.js](https://img.shields.io/badge/Node.js-1a1a1a?style=for-the-badge&logo=node.js&logoColor=4CAF50)
![JavaScript](https://img.shields.io/badge/JavaScript-1a1a1a?style=for-the-badge&logo=javascript&logoColor=F7DF1E)
![Express](https://img.shields.io/badge/Express-1a1a1a?style=for-the-badge&logo=express&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-1a1a1a?style=for-the-badge&logo=windows&logoColor=0078D6)

---

## Overview

DeadsTools is a native Windows desktop application that streamlines the Minecraft Bedrock addon development workflow. It handles project organisation, file obfuscation, and keeps itself up to date automatically - all without leaving the app.

The project is split into three parts: the **main client app**, a **self-hosted update server**, and a private **admin app** used to publish releases.

---

## Features

- **Project Manager** - Create, organise, and switch between addon projects with a clean card-based UI
- **com.mojang Detection** - Automatically locates your Bedrock development folder on first launch
- **JS & JSON Obfuscation** - Protect addon scripts and data files with one click
- **Auto-Updater** - Silently downloads updates in the background and prompts with a markdown changelog modal on next launch
- **Setup Wizard** - Guided first-run experience for new users

---

## Roadmap

The public roadmap is tracked on Trello: [https://trello.com/b/x5SZjxaG/deads-tools](https://trello.com/b/x5SZjxaG/deads-tools)

---

## Architecture

```
DeadsTools/
├── Client App          Electron - main user-facing application
├── Update Server       Express - self-hosted on a VPS, serves latest.yml + versioned builds
└── Admin App           Electron - private tool for publishing new releases
```

### Client App

Built with **Electron** using `nodeIntegration` and a frameless `BrowserWindow`. Configuration (projects folder, username, setup state) is persisted to `userData/deadstools-config.json` via a thin `config` module wrapping `fs`.

Auto-updates are handled by **electron-updater** pointed at a self-hosted generic provider. When an update is available the app fetches `changelog.md` directly from the update server (bypassing YAML parsing entirely) and renders it in a modal with a custom markdown renderer supporting headings, bullet lists, bold, italic, strikethrough, and inline code.

### Update Server

A lightweight **Express** server running on a VPS. Each release is stored under a versioned directory (`builds/v{version}/`) alongside its `changelog.md`. The server exposes:

| Endpoint | Description |
|---|---|
| `GET /update/latest.yml` | Dynamically generated - consumed by electron-updater |
| `GET /update/v{ver}/changelog.md` | Raw markdown changelog for the client modal |
| `GET /update/version` | Latest version metadata (JSON) |
| `POST /admin/upload` | Authenticated multipart upload (exe + notes) |
| `GET /admin/builds` | List all published builds |
| `DELETE /admin/builds/:version` | Remove a build |

Upload authentication is handled via a static `x-api-key` header checked against an environment variable.

### Admin App

A private **Electron** app used to publish new releases. It scans a selected `dist/` folder, reads `latest.yml` to detect the version and exact installer filename, checks for a `changelog.md` alongside the build, and streams the installer to the update server with real-time upload progress (MB transferred / %). Release notes are written in markdown directly in the app.

---

## Update Flow

```
1. App launches → checkForUpdates()
2. electron-updater fetches /update/latest.yml
3. If server version > installed version → update-available fires
4. App fetches /update/v{version}/changelog.md
5. Changelog rendered in modal - user sees "What's New"
6. electron-updater downloads installer silently in background
7. User clicks "Restart & Update" → quitAndInstall(silent, forceRun)
8. NSIS installer runs silently, app relaunches at new version
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Client UI | Electron + vanilla JS/HTML/CSS |
| Auto-updater | electron-updater (generic provider) |
| Update server | Node.js + Express |
| File uploads | Multipart form-data (no external deps) |
| Build tooling | electron-builder (NSIS installer) |
| Hosting | Self-hosted VPS |

---

## Distribution

Releases are distributed as an NSIS installer built with **electron-builder**. The admin app uploads each build to the VPS update server. End users install once and receive all future updates automatically.

> This repository does not contain source code and will not include it.
