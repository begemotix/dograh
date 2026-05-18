# Minimal-invasiver Coolify-Spike fuer Dograh

Diese lokale Variante ist als schneller Spike vorbereitet, nicht als produktionsreife DSGVO-Konfiguration. Sie nutzt die veroeffentlichten Dograh-Images fuer `api` und `ui`, damit Coolify nicht erst API und Next.js-UI aus Source bauen muss.

Der Spike veraendert keinen Dograh-Anwendungscode. Insbesondere bleibt MinIO im Upstream-Verhalten unveraendert. Eine echte Storage-Haertung ist bewusst postponed bis zur Entscheidung, Dograh produktiv zu nutzen.

## Dateien

- `docker-compose.coolify.yml`: Coolify-taugliches Compose-Profil ohne Cloudflared/nginx und mit Telemetrie deaktiviert.
- `.env.coolify.example`: Vorlage fuer Coolify-Environment-Variablen.

## Coolify-Einrichtung

1. Repository als privates Git-Repo in Coolify verbinden.
2. Als Compose-Datei `docker-compose.coolify.yml` auswaehlen.
3. Die Variablen aus `.env.coolify.example` in Coolify setzen.
4. Drei HTTPS-Domains routen:
   - UI: `ui` auf Port `3010`
   - API: `api` auf Port `8000`
   - Datei-Endpunkt: `minio` auf Port `9000`
5. MinIO-Konsole auf Port `9001` nicht oeffentlich routen.

Wenn Coolify fuer `ui`, `api` und `minio` automatisch `SERVICE_URL_*` Variablen erzeugt, koennen `PUBLIC_UI_URL`, `PUBLIC_BACKEND_URL` und `PUBLIC_MINIO_URL` leer bleiben. Falls sie manuell gesetzt werden, muessen es echte URLs mit `http://` oder `https://` sein; Platzhaltertexte wie `Set PUBLIC_MINIO_URL` bringen die API beim Start zum Absturz.

## Spike-Abgrenzung

Die Compose-Variante setzt Telemetrie server- und clientseitig auf `false` und leert PostHog-, Sentry- und Langfuse-Defaults. Das ist fuer einen technischen Spike sinnvoll, aber kein Nachweis fuer DSGVO-Produktionsreife.

- Keine echten Anruferdaten oder Gesundheitsdaten im Spike verwenden.
- Keine Dograh-MPS/Demo-Provider fuer echte Anruferdaten verwenden.
- Die UI kommt im Spike aus dem Upstream-Image. Fuer einen spaeteren Produktions-Fork muss erneut entschieden werden, ob die UI selbst gebaut wird, um build-time Defaults vollstaendig zu kontrollieren.
- MinIO bleibt im Upstream-Verhalten. Storage-Haertung wird erst entschieden, wenn Dograh produktiv weiterverfolgt wird.
- LLM/STT/TTS, Webhooks, Tools und Odoo-Zugriffe sind im Spike gesondert zu pruefen.

## Wichtige technische Hinweise

- `FASTAPI_WORKERS` bleibt in `docker-compose.coolify.yml` absichtlich bei `1`. Dograh startet mehrere Uvicorn-Prozesse auf fortlaufenden Ports; ohne vorgeschalteten internen Load Balancer waeren zusaetzliche Worker nicht erreichbar.
- `ui` wartet im Spike nur auf `api` gestartet, nicht auf `api` healthy. Das verhindert, dass der erste Deploy wegen langsamer Migrationen/Initialisierung abbricht. Wenn die UI danach nicht erreichbar ist, zuerst die `api`-Container-Logs pruefen.
- Die Services exposen ihre internen Ports explizit fuer Coolify: `ui:3010`, `api:8000`, `minio:9000`.
- WebRTC/WebSocket-Funktionen brauchen eine oeffentliche API-Domain (`PUBLIC_BACKEND_URL`) und je nach Netzwerkumgebung TURN.
- Diese Datei ist eine Spike-Hilfe. Eine produktive Version braucht danach eine separate Entscheidung zu Storage, Provider-Auswahl, AVVs, Loeschkonzept, Zugriffskontrollen und Betriebsprozessen.
