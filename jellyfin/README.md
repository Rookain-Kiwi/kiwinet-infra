# jellyfin — Media Server (test de cohabitation avec Plex)

Serveur média open source, déployé **en parallèle de Plex, sans coupure**, pour
évaluer une migration potentielle suite à la hausse tarifaire du Plex Pass
Lifetime (juillet 2026). Accès **local uniquement** pour cette phase.

> Contexte global : [kiwinet-docs](https://github.com/Rookain-Kiwi/kiwinet-docs)

---

## Stack

| Container  | Image                          | Port interne |
|------------|--------------------------------|--------------|
| `jellyfin` | `lscr.io/linuxserver/jellyfin` | 8096         |

---

## Configuration

| Paramètre      | Valeur                                                     |
|----------------|-------------------------------------------------------------|
| Architecture   | ARM AArch64                                                  |
| Image          | LinuxServer.io (recommandée par la doc Jellyfin pour ARM, pas d'accélération matérielle disponible sur ce SoC de toute façon) |
| Données config | Volume nommé `jellyfin-config`                              |
| Bibliothèques  | `/mnt/Libraries/{TV Shows,Movies,Music}` → `/data/*` (CIFS, `:ro`) |
| Accès          | LAN direct / VPN WireGuard uniquement — **aucune route Traefik publique pour l'instant** |

---

## Structure

```
jellyfin/
└── docker-compose.yml
```

---

## Déploiement

```bash
cd /opt/kiwinet-services/jellyfin

docker compose up -d
docker compose logs -f

# Mise à jour
docker compose pull && docker compose up -d --force-recreate
```

Premier démarrage : assistant de configuration sur `http://<ip-lan-vm>:8096`.

---

## Ressources (vérifié le 05/09/2026 avant déploiement)

| Métrique       | Disponible sur la VM | Recommandation Jellyfin |
|----------------|-----------------------|--------------------------|
| RAM disponible | 9,1 Gi                | 4 Go mini (serveur headless), 8 Go recommandé |
| Load average   | ~1,1 / 3 vCPU         | —                        |
| Disque `/`     | 65 Go libres           | ~100 Go recommandé (confort, pas bloquant ici) |

Aucune accélération matérielle possible (SoC Marvell Cortex-A72, non supporté
par Jellyfin — seul le Rockchip RK3588/3588S l'est côté ARM). Même contrainte
que Plex : direct play/direct stream doivent rester la norme, un vrai
transcodage sera lent dans les deux cas.

---

## Accès distant — non activé pour cette phase

Pas de route Traefik/DNS publique tant que le test n'est pas concluant.
Options à trancher plus tard, par cohérence avec l'approche déjà retenue pour
`freebox.kiwinet.me` (décoy DNS passif, accès réel via VPN uniquement) :

- **VPN WireGuard existant** (recommandé) : pas d'exposition supplémentaire,
  cohérent avec la posture sécurité actuelle du homelab.
- **Route Traefik publique** (`jellyfin.kiwinet.me`), comme Plex/Komga :
  plus simple pour les enfants en déplacement, mais surface d'attaque
  supplémentaire à assumer — à activer uniquement après validation du test.

---

## Points de vigilance identifiés (test en cours)

| Sujet | Risque | À valider |
|---|---|---|
| Cast Chromecast | Récepteur Cast Jellyfin historiquement instable derrière un reverse proxy HTTPS (rapports Jellyfin 10.8.x) | Tester le cast réel une fois une route Traefik éventuellement activée |
| Appli Samsung Tizen | Déploiement officiel (24/05/2026) inégal selon modèle/firmware/région | Vérifier disponibilité sur le modèle réel avant d'en dépendre |
| Intégration Home Assistant | Native, media_player par session, media source films/séries/musique | Tester le flux réel utilisé par les enfants (sélection appareil → source) |
