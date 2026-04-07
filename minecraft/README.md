# minecraft — Serveur Minecraft Java Edition

Serveur privé dockerisé. Accessible via `minecraft.kiwinet.me:25565`.

> Contexte global : [kiwinet-docs](https://github.com/Rookain-Kiwi/kiwinet-docs)

---

## Stack

| Container   | Image                           |
|-------------|---------------------------------|
| `minecraft` | `itzg/minecraft-server` (ARM64) |

---

## Configuration

| Paramètre   | Valeur                             |
|-------------|------------------------------------|
| Type        | VANILLA (PaperMC dès support 26.1) |
| Version     | 26.1                               |
| RAM allouée | 4 Go                               |
| Joueurs max | 6                                  |
| Difficulté  | Normal / Survie                    |
| Whitelist   | Activée                            |
| RCON        | Activé (port 25575, interne)       |

---

## Structure

```
minecraft/
├── docker-compose.yml
└── .env                # RCON_PASSWORD (gitignored)
```

Le volume `minecraft-data` est déclaré `external: true` — il persiste entre les recreations.

---

## Fichier `.env` à créer

```bash
cat > .env << 'EOF'
RCON_PASSWORD=<mot_de_passe>
EOF
```

---

## Déploiement

```bash
cd /opt/kiwinet-services/minecraft

docker compose up -d
docker compose logs -f

# Mise à jour
docker compose pull && docker compose up -d --force-recreate
```

---

## RCON — console à distance

```bash
docker exec -i minecraft rcon-cli --password <RCON_PASSWORD>

# Commandes utiles
> whitelist add <pseudo>
> whitelist remove <pseudo>
> whitelist list
> op <pseudo>
> say Message serveur
```

---

## Routing réseau

```
Client → minecraft.kiwinet.me:25565 → Traefik TCP passthrough → container:25565
```

Le port 25565 n'est pas exposé directement — tout passe par le TCP passthrough Traefik.
