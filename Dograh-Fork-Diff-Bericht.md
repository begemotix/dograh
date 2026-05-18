# Dograh Spike-Diff-Bericht

Stand: 2026-05-18

## Executive Summary

Die lokale Version ist jetzt wieder minimal-invasiv vorbereitet. Der zuvor eingefuehrte MinIO-Code-Patch wurde zurueckgenommen und ist bewusst postponed bis zur Entscheidung, Dograh produktiv zu nutzen.

Der verbleibende Stand ist ein **technischer Spike fuer Coolify**, kein produktionsreifer DSGVO-Deploy.

## Was bleibt im Fork fuer den Spike?

Es bleiben nur additive Deployment-Dateien:

- `docker-compose.coolify.yml`
- `.env.coolify.example`
- `COOLIFY-DEPLOYMENT.md`
- `ui/Dockerfile.coolify`
- `Dograh-Fork-Diff-Bericht.md` (dieser Bericht)

Es gibt aktuell **keine Aenderungen an bestehendem Dograh-Anwendungscode** mehr.

## Was wurde zurueckgenommen?

Folgende MinIO-Codeaenderungen wurden entfernt:

- `api/constants.py`: keine neue Variable `MINIO_ANONYMOUS_ACCESS_POLICY`
- `api/services/storage.py`: keine Weitergabe einer neuen MinIO-Policy an `MinioFileSystem`
- `api/services/filesystem/minio.py`: kein `public_client`, keine privaten Bucket-Policies, keine Presigned-MinIO-URLs fuer private Buckets

Damit bleibt Dograhs Upstream-MinIO-Verhalten unveraendert.

## Warum ist das wichtig?

Fuer einen Spike wollen wir moeglichst wenig Fork-Delta:

- Dograh-Updates bleiben einfacher.
- Wir testen zuerst, ob Dograh funktional fuer unseren Use Case passt.
- Storage-Haertung wird nicht halb-produktiv eingefuehrt, bevor klar ist, ob Dograh ueberhaupt eingesetzt wird.

Nachteil: Der Spike ist nicht fuer echte Gesundheitsdaten geeignet. MinIO ist im Upstream-Code fuer OSS/Local-Use pragmatisch offen konfiguriert. Das ist fuer einen Funktionstest akzeptabel, aber nicht fuer Produktivbetrieb mit Anruferdaten.

## Verbleibende neue Dateien

### `docker-compose.coolify.yml`

Zweck:

- Separates Compose-Profil fuer Coolify.
- Baut `api` und `ui` aus diesem Clone.
- Deaktiviert Telemetrie-Defaults per Environment.
- Nutzt eigene Public URLs fuer UI, API und MinIO.
- Nutzt Coolifys `SERVICE_URL_*` Werte als Fallback, wenn `PUBLIC_*` nicht manuell gesetzt ist.
- Verzichtet auf Cloudflared/nginx aus der Upstream-Compose.
- Laesst `ui` im Spike bereits starten, sobald `api` gestartet ist; der API-Healthcheck bekommt mehr Anlaufzeit.

Risiko:

- Minimaler Fork-Aufwand, weil die Datei additiv ist.
- Noch nicht mit `docker compose config` verifiziert, da Docker lokal nicht im PATH verfuegbar war.
- Wenn `api` wirklich crasht, kann der Deploy trotzdem weiterlaufen. Dann sind die `api`-Container-Logs die naechste Diagnosequelle.

### `.env.coolify.example`

Zweck:

- Vorlage fuer Coolify-Environment-Variablen.
- Markiert explizit, dass das MinIO-Verhalten fuer den Spike unveraendert bleibt.

Risiko:

- Keine echten Secrets eintragen/committen.
- Nicht fuer echte Anruferdaten verwenden.
- Manuell gesetzte `PUBLIC_*` Werte muessen echte URLs sein. Platzhaltertexte wie `Set PUBLIC_MINIO_URL` sind ungueltig.

### `ui/Dockerfile.coolify`

Zweck:

- Baut die Next.js-UI mit eigener `NEXT_PUBLIC_BACKEND_URL`.
- Verhindert, dass Dograh-Demo-Defaults fuer Chatwoot/PostHog/Sentry in den Build geraten.

Risiko:

- Additive Datei, aber UI-Build muss in Coolify einmal getestet werden.

### `COOLIFY-DEPLOYMENT.md`

Zweck:

- Kurze Anleitung fuer den Coolify-Spike.
- Klare Abgrenzung: kein Produktionsdeploy, keine echten Gesundheitsdaten.

## Aktueller Diff-Charakter

Der relevante Fork-Diff ist jetzt:

- **Deployment-only**
- **additiv**
- **ohne Anwendungscode-Patch**
- **nicht DSGVO-produktionsfertig**

## Postponed Bis Produktiventscheidung

Diese Themen bleiben bewusst offen:

- MinIO/Storage-Haertung
- Alternative: privater EU S3-kompatibler Object Storage
- Loeschkonzept fuer Recordings/Transkripte
- Provider-Auswahl fuer LLM/STT/TTS mit AVV/DPA
- Odoo/Webhook-Datenminimierung
- TURN/WebRTC-Betrieb fuer echte Telefonie

## Empfohlener Spike-Test

1. Coolify-Build mit `docker-compose.coolify.yml`.
2. UI oeffnen und Account/Organisation anlegen.
3. Dummy-Agent ohne echte personenbezogene Daten anlegen.
4. Web Call mit Testdaten ausfuehren.
5. Recording/Transcript nur mit Dummy-Inhalten pruefen.
6. Einen simplen Webhook gegen eine interne Test-URL ausloesen.
7. Danach Entscheidung: Dograh weiterverfolgen oder abbrechen.

Wenn Dograh funktional ueberzeugt, sollte vor Produktivbetrieb ein separater Hardening-Branch geplant werden.
