# MeTube + Basic Auth (para Coolify)

Stack: **Caddy** (puerta con usuario/contraseña) → **MeTube** (interno, sin puerto expuesto).
Coolify (Traefik) enruta y da HTTPS. La auth vive en este stack.

Todo en un solo `docker-compose.yml` **autocontenido**: el Caddyfile va embebido en
`configs.content` (sin bind mount), así Coolify no falla al montar un fichero suelto.

## Credenciales por defecto

- Usuario: `admin`
- Contraseña: `cambiame`  ← **CÁMBIALA antes de exponerla a internet**

## Cambiar la contraseña

1. Genera un hash bcrypt:

   ```bash
   docker run --rm caddy:2-alpine caddy hash-password --plaintext 'TU_PASSWORD_NUEVA'
   ```

2. En `docker-compose.yml`, dentro de `configs.caddyfile.content`, sustituye el hash
   de la línea `admin ...`.

   > ⚠️ **Duplica cada `$`**: el hash sale como `$2a$14$...` y aquí debe quedar
   > `$$2a$$14$$...` porque Docker Compose interpola variables. Si no lo duplicas,
   > la contraseña no valida.

   Para cambiar el usuario, sustituye `admin`. Más usuarios = más líneas `usuario hash`.

3. Commit + push. Redeploy en Coolify.

## Desplegar en Coolify

1. **New Resource → Private Git** → conecta este repo.
2. Base Directory `/`, usa `docker-compose.yml`.
3. Coolify detecta `SERVICE_FQDN_CADDY_80` → dominio + HTTPS al servicio `caddy`.
   Para dominio propio, ponlo en el campo Domains del servicio `caddy`.
4. Deploy → pide usuario/contraseña.

Marca `descargas` como **Persistent Storage** en Coolify si quieres que las
descargas sobrevivan a redeploys.

## Probar en local

```bash
# mapea caddy a un puerto y prueba:
#   añade  ports: ["8092:80"]  al servicio caddy (Coolify no lo necesita, usa FQDN)
docker compose up -d
curl -u admin:cambiame http://localhost:8092   # -> HTTP 200
curl http://localhost:8092                       # -> HTTP 401
```
