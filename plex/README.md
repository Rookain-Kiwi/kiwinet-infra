# plex — Plex Media Server

Serveur multimédia dockerisé. Accessible via `plex.kiwinet.me`.

> Contexte global : [kiwinet-docs](https://github.com/Rookain-Kiwi/kiwinet-docs)

---

## Stack

| Container | Image                | Port interne |
|-----------|----------------------|--------------|
| `plex`    | `plexinc/pms-docker` | 32400        |

---

## Configuration

| Paramètre      | Valeur                                  |
|----------------|-----------------------------------------|
| Architecture   | ARM AArch64                             |
| Timezone       | Europe/Paris                            |
| PUID / PGID    | 994 / 991 (utilisateur `plex` VM)       |
| Transcodage    | `/tmp/plex-transcode` (RAM)             |
| Données config | `/var/lib/plexmediaserver` (bind mount) |

---

## Structure

```
plex/
├── docker-compose.yml
└── .env                # PLEX_CLAIM (gitignored)
```

---

## Fichier `.env` à créer

```bash
cat > .env << 'EOF'
PLEX_CLAIM=claim-XXXXXXXXXXXXXXXX
EOF
```

Le token se génère sur `https://plex.tv/claim` (valable 4 minutes). Requis uniquement au premier démarrage — le container peut ensuite être relancé sans `PLEX_CLAIM`.

---

## Déploiement

```bash
cd /opt/kiwinet-services/plex

docker compose up -d
docker compose logs -f

# Mise à jour
docker compose pull && docker compose up -d --force-recreate
```

---

## Médias

Bibliothèques montées en lecture seule depuis le NAS Freebox via CIFS (`/etc/fstab`). Les points de montage sont définis dans le `docker-compose.yml`.
