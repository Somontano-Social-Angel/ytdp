# MeTube + Basic Auth (para Coolify)

Stack: **Caddy** (puerta con usuario/contraseña) → **MeTube** (interno, sin puerto expuesto).
Coolify (Traefik) enruta y da HTTPS. La auth vive en este stack, así que no depende
de tocar los nombres de router que Coolify autogenera.

## Credenciales por defecto

- Usuario: `admin`
- Contraseña: `cambiame`  ← **CÁMBIALA antes de exponerla a internet**

## Cambiar la contraseña

Genera un hash bcrypt nuevo y pégalo en `Caddyfile`:

```bash
docker run --rm caddy:2-alpine caddy hash-password --plaintext 'TU_PASSWORD_NUEVA'
```

Copia la línea que empieza por `$2a$...` y sustitúyela en `Caddyfile`:

```
basic_auth {
    admin $2a$14$....(hash nuevo)....
}
```

Para cambiar también el usuario, sustituye `admin` por lo que quieras.
Puedes añadir más usuarios con más líneas `usuario  hash` dentro del bloque.

> Nota: el hash va en `Caddyfile` (fichero suelto), NO hace falta escapar `$`.
> Si algún día metes el hash directo en `docker-compose.yml`, ahí sí hay que
> duplicar cada `$` como `$$`.

## Desplegar en Coolify

1. Sube esta carpeta `coolify/` a un repo git (o usa "Docker Compose" pegando el compose).
2. En Coolify: **New Resource → Docker Compose** apuntando al repo/carpeta.
3. Coolify detecta `SERVICE_FQDN_CADDY_80` y te asigna dominio + HTTPS al servicio `caddy`.
   - Si quieres tu propio dominio, ponlo en el campo Domains del servicio `caddy`.
4. Deploy. Entra al dominio → te pedirá usuario/contraseña.

Las descargas quedan en el volumen `./descargas` (persístelo en Coolify si quieres
que sobrevivan a redeploys: configúralo como Persistent Storage).

## Probar en local

```bash
docker run --rm caddy:2-alpine caddy hash-password --plaintext 'test'   # si quieres otra pass
# compose de prueba mapeando caddy a un puerto local:
#   ports: ["8091:80"]  en el servicio caddy
# luego:  curl -u admin:cambiame http://localhost:8091   -> HTTP 200
```
