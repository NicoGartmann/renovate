# `renovate` – Zentraler Self-Hosted Renovate Runner & Presets

Zentrales Repository für die automatisierte Dependency-Aktualisierung via [Renovate](https://docs.renovatebot.com/) über GitHub Actions. Beinhaltet sowohl den wöchentlichen GitHub-Actions-Runner als auch wiederverwendbare **Shareable Config Presets** für alle angebundenen Repositories (aktuell [`docker-config`](https://github.com/NicoGartmann/docker-config) sowie dieses Repository selbst).

---

## 1. Verzeichnisstruktur & Architektur

```text
.
├── .github/
│   └── workflows/
│       └── renovate.yml            # Wöchentliche GitHub Action (Mo 06:00 Uhr + manueller Trigger)
├── presets/
│   ├── base.json                   # Globales Basis-Preset (Dashboard, Reifezeit, Labels, Commits)
│   └── docker-compose.json         # Spezialisiertes Preset für Docker-Compose-Repositories
├── config.js                       # Zentrale Runner-Konfiguration (Plattform & explizite Repo-Liste)
└── renovate.json                   # Self-Update-Konfiguration für dieses Repository
```

### Funktionsweise
1. **Zeitgesteuerte Ausführung**: Jeden **Montag um 05:00 Uhr UTC** (06:00 Uhr MEZ / 07:00 Uhr MESZ) startet `.github/workflows/renovate.yml` automatisch. Zusätzlich kann der Lauf jederzeit manuell im Reiter **Actions → Renovate → Run workflow** angestoßen werden (optional nur für ein einzelnes Repository oder im `lookup`-Dry-Run-Modus).
2. **Authentifizierung über eigene GitHub App**: Der Workflow erzeugt zur Laufzeit über `actions/create-github-app-token@v2` aus `RENOVATE_APP_ID` und `RENOVATE_APP_PRIVATE_KEY` ein kurzlebiges Installations-Token. Alle erstellten Pull Requests und Dependency-Dashboard-Issues treten unter dem Namen der GitHub App auf.
3. **Zweistufiges Preset-System**:
   * [`presets/base.json`](./presets/base.json): Definiert globale Standards (`config:recommended`, `:semanticCommits`, `:dependencyDashboard`, `minimumReleaseAge: "3 days"`, kein Automerge, keine künstlichen PR-Limits, Label `major-update` für Major-Sprünge).
   * [`presets/docker-compose.json`](./presets/docker-compose.json): Erweitert `presets/base.json` für Docker-Stacks (`pinDigests: false`, Custom-Regex-Versioning für `ghcr.io/servercontainers/samba` im Format `smbd-only-aX.Y.Z-sX.Y.Z-rN`, sowie das Warn-Label `database-major-upgrade` inkl. Migrationshinweis für Datenbank-Major-Updates).

---

## 2. Einrichtung der GitHub App & Action Secrets (Einmaliges Setup)

Damit der Workflow im `renovate`-Repository auf andere Repositories unter `NicoGartmann/*` zugreifen darf, wird eine eigene GitHub App verwendet:

### Schritt 1: GitHub App erstellen
1. Auf GitHub öffnen: **Settings** (Profilbild oben rechts) → **Developer settings** → **GitHub Apps** → **New GitHub App**.
2. Basisdaten eintragen:
   * **GitHub App name**: z. B. `nico-homelab-renovate` (muss global eindeutig sein)
   * **Homepage URL**: `https://github.com/NicoGartmann/renovate`
   * **Webhook**: Haken bei **Active** **entfernen**
3. Unter **Permissions → Repository permissions** folgende Berechtigungen setzen:
   * **Actions**: `Read and write`
   * **Checks**: `Read and write`
   * **Commit statuses**: `Read and write`
   * **Contents**: `Read and write`
   * **Issues**: `Read and write`
   * **Metadata**: `Read-only`
   * **Pull requests**: `Read and write`
   * **Workflows**: `Read and write`
4. Unter **Where can this GitHub App be installed?** die Option **Only on this account** (`NicoGartmann`) wählen und auf **Create GitHub App** klicken.

### Schritt 2: App-ID notieren & Private Key erzeugen
1. Auf der Übersichtsseite der erstellten App die **App ID** notieren.
2. Weiter unten unter **Private keys** auf **Generate a private key** klicken (lädt eine `.pem`-Datei herunter).

### Schritt 3: GitHub App auf den Repositories installieren
1. Links im Menü der App auf **Install App** → neben `NicoGartmann` auf **Install** klicken.
2. **Only select repositories** wählen und mindestens folgende Repositories freischalten:
   * `NicoGartmann/renovate`
   * `NicoGartmann/docker-config`
3. Auf **Install** klicken.

### Schritt 4: Secrets im `renovate`-Repository hinterlegen
Unter `NicoGartmann/renovate` → **Settings** → **Secrets and variables** → **Actions** → **New repository secret** folgende zwei Secrets anlegen:
* **`RENOVATE_APP_ID`**: Die notierte App ID aus Schritt 2.
* **`RENOVATE_APP_PRIVATE_KEY`**: Der vollständige Inhalt der heruntergeladenen `.pem`-Datei (inkl. Header- und Footer-Zeile).

---

## 3. Neues Repository in Renovate aufnehmen

Um künftig ein weiteres Projekt (z. B. `raspi-ansible` oder `scanner-webui`) einzubinden, sind 3 Schritte nötig:

1. **GitHub App berechtigen**: Unter GitHub *Settings → Applications → Installed GitHub Apps → Configure* das neue Repository zur Auswahl hinzufügen.
2. **In [`config.js`](./config.js) registrieren**: Das Repository in das Array `repositories` eintragen (und bei Bedarf ein weiteres Technologie-Preset unter `presets/` anlegen).
3. **`renovate.json` im Ziel-Repository anlegen**:
   ```json
   {
     "$schema": "https://docs.renovatebot.com/renovate-schema.json",
     "extends": [
       "github>NicoGartmann/renovate//presets/base.json"
     ]
   }
   ```
