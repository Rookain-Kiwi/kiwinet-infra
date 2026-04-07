# kiwinet-services

Services applicatifs auto-hébergés de Kiwinet — reverse proxy, médias, domotique, jeux.
Un `docker-compose.yml` par service, Traefik comme point d'entrée unique.

> Contexte global du projet : [kiwinet-docs](https://github.com/Rookain-Kiwi/kiwinet-docs)

---

## Prérequis

- Docker + Docker Compose installés sur la VM
- Réseau Docker `proxy` créé : `docker network create proxy`
- Fichiers secrets créés localement (voir section de chaque service)

---

## Structure

```
kiwinet-services/
├── traefik/
│   ├── docker-compose.yml
│   ├── traefik.yml             # Config statique (restart requis si modifié)
│   ├── dynamic.yml             # Config dynamique (routers, middlewares)
│   ├── acme.json               # Certificats Let's Encrypt (chmod 600, gitignored)
│   └── .htpasswd               # Auth basic dashboard (gitignored)
├── plex/
│   ├── docker-compose.yml
│   └── .env                    # PLEX_CLAIM (gitignored)
├── minecraft/
│   ├── docker-compose.yml
│   └── .env                    # RCON_PASSWORD (gitignored)
└── ha/
    ├── docker-compose.yml
    ├── mosquitto/config/mosquitto.conf
    └── config/                 # Données HA (gitignored)
```

---

## Ordre de démarrage

Traefik crée le réseau `proxy` — il doit démarrer en premier.

```bash
cd traefik && docker compose up -d

# Puis dans n'importe quel ordre
cd plex      && docker compose up -d
cd minecraft && docker compose up -d
cd ha        && docker compose up -d
```

---

## Mise à jour

```bash
# Machine locale
git push origin main

# VM
cd /opt/kiwinet-services
git pull
cd <service> && docker compose up -d --force-recreate
```

---

## Services

| Service                    | Sous-dossier | URL                          | README                                       |
|----------------------------|--------------|------------------------------|----------------------------------------------|
| Traefik                    | `traefik/`   | `traefik.kiwinet.me`         | [traefik/README.md](./traefik/README.md)     |
| Plex                       | `plex/`      | `plex.kiwinet.me`            | [plex/README.md](./plex/README.md)           |
| Minecraft                  | `minecraft/` | `minecraft.kiwinet.me:25565` | [minecraft/README.md](./minecraft/README.md) |
| Home Assistant + Mosquitto | `ha/`        | `hub.kiwinet.me`             | [ha/README.md](./ha/README.md)               |
