# Notificador de sismos (GitHub Actions → ntfy)

Este repo no hace nada por sí solo — solo escucha cuando el Cloudflare
Worker (`sgc-ntfy-worker`) le avisa que hay un sismo real que notificar,
y en ese momento publica el mensaje a ntfy desde la IP de GitHub
Actions (distinta a la de Cloudflare, para evitar el problema de la IP
compartida saturada).

## Configuración

1. Crea un repo nuevo en GitHub (o usa uno existente) y sube esta
   carpeta tal cual (debe quedar `.github/workflows/notificar-sismo.yml`).

2. En **Settings → Secrets and variables → Actions → New repository
   secret**, agrega:
   - `NTFY_TOPIC`: el nombre de tu topic (ej. `sismoscolombiaalerta15`)
   - `NTFY_TOKEN`: tu token de acceso de ntfy (`tk_...`)
   - `NTFY_URL` (opcional): solo si no usas `https://ntfy.sh`

3. Crea un **Personal Access Token** de GitHub para que Cloudflare
   pueda disparar este workflow:
   - Ve a **github.com → Settings (de tu perfil) → Developer settings
     → Personal access tokens → Fine-grained tokens → Generate new
     token**.
   - Dale acceso de **solo este repositorio**.
   - Permisos: **Contents: Read and write** (o "Actions: Read and
     write", según la versión de GitHub) — necesita poder disparar
     `repository_dispatch`.
   - Copia el token generado (empieza con `github_pat_...`).

4. En el Worker de Cloudflare, configura:
   ```bash
   npx wrangler secret put GITHUB_TOKEN
   # pega el github_pat_... que generaste

   npx wrangler secret put GITHUB_REPO
   # escribe: tu-usuario/tu-repo
   ```

## Cómo probarlo manualmente

Puedes disparar el workflow a mano (sin esperar a un sismo real) para
probar que la configuración de secrets está bien, usando `curl`:

```bash
curl -X POST \
  -H "Authorization: Bearer TU_GITHUB_PAT" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/TU-USUARIO/TU-REPO/dispatches \
  -d '{
    "event_type": "sismo-detectado",
    "client_payload": {
      "id": "PRUEBA123",
      "mag": 4.5,
      "place": "Prueba - Colombia",
      "localTime": "2026-01-01 12:00",
      "depth": 10,
      "status": "manual"
    }
  }'
```

Si todo está bien configurado, en unos segundos debería llegarte la
notificación de prueba a ntfy, y en la pestaña **Actions** del repo
vas a ver la corrida del workflow.
