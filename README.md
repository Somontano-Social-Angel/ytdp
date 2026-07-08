# MeTube + Basic Auth (para Coolify)

Stack: **Caddy** (puerta con usuario/contraseña) → **MeTube** (interno, sin puerto expuesto).
Coolify (Traefik) enruta y da HTTPS. La auth vive en este stack.

Todo en un solo `docker-compose.yml` **autocontenido**: el Caddyfile va embebido en
`configs.content` (sin bind mount, así Coolify no falla al montar un fichero suelto).
El usuario y el hash se leen de **variables de entorno**.

## Credenciales por defecto

- Usuario: `admin`
- Contraseña: `cambiame`  ← **CÁMBIALA antes de exponerla a internet**

## Cambiar la contraseña (recomendado: env en Coolify, SIN tocar git)

1. Genera un hash bcrypt:

   ```bash
   docker run --rm caddy:2-alpine caddy hash-password --plaintext 'TU_PASS_NUEVA'
   ```

   Sale algo como `$2a$14$....`

2. En Coolify → tu recurso → **Environment Variables**, añade:

   | Variable | Valor |
   |----------|-------|
   | `METUBE_USER` | el usuario que quieras (ej. `admin`) |
   | `METUBE_HASH` | el hash **tal cual**, con sus `$` (ej. `$2a$14$....`) |

   > Aquí el hash va **crudo**, NO se duplican los `$`. Solo se duplican si lo
   > escribieras dentro del compose; como va por env, no hace falta.

3. **Redeploy**. Listo — sin commits ni push.

## Desplegar en Coolify

1. **New Resource → Private Git** → conecta este repo.
2. Base Directory `/`, usa `docker-compose.yml`.
3. Coolify detecta `SERVICE_FQDN_CADDY_80` → dominio + HTTPS al servicio `caddy`.
4. (Opcional pero recomendado) pon `METUBE_USER` / `METUBE_HASH` en Environment Variables.
5. Deploy → pide usuario/contraseña.

Marca `descargas` como **Persistent Storage** en Coolify para que las descargas
sobrevivan a redeploys.

## Probar en local

```bash
# defaults (admin/cambiame):
docker compose up -d          # + añade  ports: ["8093:80"]  al servicio caddy para probar
curl -u admin:cambiame http://localhost:8093     # 200
curl http://localhost:8093                         # 401

# con otra pass via env (hash crudo, sin duplicar $):
export METUBE_USER=jefe
export METUBE_HASH="$(docker run --rm caddy:2-alpine caddy hash-password --plaintext 'test1234')"
docker compose up -d
curl -u jefe:test1234 http://localhost:8093        # 200
```
