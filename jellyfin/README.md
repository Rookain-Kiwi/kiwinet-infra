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
| Accès          | `jellyfin.kiwinet.me` (HTTPS/Traefik, public) + LAN direct sur `:8096` |

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

## Accès distant — activé (décision du 06/09/2026)

Route Traefik publique `jellyfin.kiwinet.me` activée, même posture que
Plex/Komga. Choix assumé après validation de la phase de test local :
priorité donnée à la simplicité d'accès pour les enfants en déplacement,
au prix d'une surface d'attaque équivalente à celle déjà acceptée pour Plex
(pas de saut qualitatif de risque, juste un renoncement à l'avantage
"VPN uniquement" qui avait été envisagé initialement).

Pré-requis pour que le certificat Let's Encrypt (challenge HTTP) fonctionne :
- Enregistrement DNS `jellyfin.kiwinet.me` pointant vers la même cible que
  `plex.kiwinet.me` (à créer chez Bluehost si pas déjà fait)
- Port 80/443 déjà ouvert côté ufw et déjà forwardé côté Freebox pour les
  autres services Traefik — aucun changement réseau supplémentaire requis

---

## Points de vigilance identifiés (test en cours)

| Sujet | Risque | À valider |
|---|---|---|
| Cast Chromecast | Récepteur Cast Jellyfin historiquement instable derrière un reverse proxy HTTPS (rapports Jellyfin 10.8.x) | Tester le cast réel une fois une route Traefik éventuellement activée |
| Appli Samsung Tizen | Déploiement officiel (24/05/2026) inégal selon modèle/firmware/région | Vérifier disponibilité sur le modèle réel avant d'en dépendre |
| Intégration Home Assistant | Native, media_player par session, media source films/séries/musique | Tester le flux réel utilisé par les enfants (sélection appareil → source) |
