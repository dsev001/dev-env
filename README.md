# dev-env

Images für die Arbeitsplätze auf dem Coder-Server. Je Ordner ein Image; der Workflow baut sie bei jedem Push auf `main` und legt sie auf `ghcr.io` ab.

| Ordner | Image | Inhalt |
|---|---|---|
| `base/` | `ghcr.io/dsev001/dev-env-base` | Ubuntu 26.04; `git`, `gh`, Claude Code (apt, Kanal stable), Codex (Release-Binary); `tmux`, `make`, `vim`, `nano`; `ripgrep`, `fd`, `jq`, `yq`, `tree`, `htop`, `ncdu`; `curl`, `wget`, `rsync`, `openssh-client`, `dnsutils`, `netcat`; `python3` mit `venv` und `pip`, `sqlite3`, `shellcheck`, `actionlint`, `git-lfs`; Docker-Client mit `buildx` und `compose` (apt von Docker, ohne Daemon; `docker-compose` als Link); Benutzer `coder` mit UID 1000 |
| `cls/` | `ghcr.io/dsev001/dev-env-cls` | Basis plus JDK 25 (Temurin), Node 26.13.0, Pillow, `psql`, Skript `cls-setup` |

Ein Projekt-Image setzt `FROM ghcr.io/dsev001/dev-env-base` davor und ergänzt seine Werkzeuge, zum Beispiel JDK und Node. Die Coder-Vorlage des Projekts nennt dann dieses Image.

## Regeln

- Kein `sudo` im Image. Ein Werkzeug kommt über das Dockerfile, damit jeder Nutzer dasselbe Image hat.
- Versionen und Digests sind gepinnt. Beim Wechsel Tag und Digest zusammen ändern.
- Gebaut wird in GitHub Actions, nicht lokal. Das Paket auf `ghcr.io` muss öffentlich sein, damit der Cluster es ohne Anmeldung zieht: nach dem ersten Lauf unter „Packages“ prüfen.
- Nichts Persönliches im Image: keine Anmeldungen, keine Schlüssel. Anmeldungen (`gh auth login`, `claude`) liegen im Home des Workspace, das ein Volume ist.

## Werkzeug hinzufügen

1. Paket in `base/Dockerfile` ergänzen, bei Fremdquellen mit Schlüsselprüfung wie bei `gh`, Claude Code und Docker; Binaries wie Codex mit Version und Prüfsumme.
2. Push auf `main`; der Workflow baut das Image neu.
3. Workspace in Coder neu starten; er zieht `latest`.

## Projekt-Image hinzufügen

1. Ordner `<projekt>/` mit `Dockerfile`, erste Zeile `FROM ghcr.io/dsev001/dev-env-base:latest`.
2. Im Workflow einen Job wie `cls` anlegen: `needs: build`, Ordner als `context`, Image `dev-env-<projekt>`.
3. Zeile in der Tabelle oben.
