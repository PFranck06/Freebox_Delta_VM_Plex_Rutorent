# Freebox Delta Debian Media Installer

Script Bash pour préparer une VM Debian 12 sur Freebox Delta avec :

- changement du port SSH ;
- pare-feu UFW ;
- détection automatique des disques Freebox montés dans `/mnt` ;
- installation de Plex Media Server ;
- installation d’Apache ;
- installation de rTorrent ;
- installation de ruTorrent avec plugins ;
- configuration optionnelle d’un certificat SSL Let’s Encrypt ;
- mode première installation ou mise à jour ;
- utilisateur unique `freebox`.

Le script est prévu pour une VM Debian 12 créée depuis Freebox OS, avec les disques Freebox déjà montés dans `/mnt`, par exemple :

```bash
/mnt/Freebox
/mnt/Freebox2
```

---

## Fonctionnalités

### Système

- Utilise l’utilisateur VM par défaut : `freebox`.
- Ajoute `freebox` au groupe `sudo`.
- Configure SSH sur un port choisi au lancement.
- Désactive la connexion SSH root.
- Active l’authentification SSH par mot de passe.
- Configure UFW.

### Disques Freebox

Le script détecte automatiquement tous les disques montés sous :

```bash
/mnt/Freebox*
```

Exemples détectés :

```bash
/mnt/Freebox
/mnt/Freebox2
```

Le disque principal choisi sert aux téléchargements rTorrent :

```bash
/mnt/Freebox/Torrents
```

Les autres disques restent accessibles depuis ruTorrent via :

```bash
/mnt/
```

### rTorrent

- rTorrent tourne avec l’utilisateur `freebox`.
- Le service systemd est créé automatiquement.
- rTorrent démarre au boot.
- Port peer par défaut : `50000`.
- SCGI local : `127.0.0.1:5000`.
- Dossiers créés :

```bash
/mnt/Freebox/Torrents/watch
/mnt/Freebox/Torrents/downloads
/mnt/Freebox/Torrents/session
```

### ruTorrent

- Installation propre depuis GitHub.
- Plugins inclus via clone récursif.
- Interface protégée par HTTP Basic Auth.
- Utilisateur HTTP fixe : `freebox`.
- Mot de passe généré automatiquement.
- Mot de passe affiché dans le résumé final.
- Identifiants sauvegardés dans :

```bash
/root/freebox-media-credentials.txt
```

### Plex

- Installation via dépôt Plex officiel.
- Ouverture du port `32400`.
- Ajout de l’utilisateur `plex` au groupe `freebox`.
- Vérification de l’accès aux disques Freebox.
- Claim Plex demandé à la fin, car le token expire rapidement.
- Détection si Plex est déjà revendiqué.

### SSL optionnel

Si un nom de domaine est fourni, le script peut configurer un certificat Let’s Encrypt avec Certbot et Apache.

---

## Prérequis

- Freebox Delta avec VM Debian 12.
- Utilisateur VM nommé `freebox`.
- Accès root ou sudo.
- Disques Freebox déjà montés dans `/mnt`.
- Accès Internet depuis la VM.
- Redirections de ports configurées dans Freebox OS si accès externe souhaité.

Ports utiles :

```text
SSH personnalisé : choisi au lancement
HTTP             : 80
HTTPS            : 443
Plex             : 32400
rTorrent peer    : 50000
```

---

## Installation

Télécharger le script sur la VM :

```bash
wget -O Freebox_Delta_VM_Plex_RuTorrent.sh https://github.com/PFranck06/Freebox_Delta_VM_Plex_RuTorrent.sh
```

Rendre le script exécutable :

```bash
chmod +x install_freebox_media_vm.sh
```

Lancer l’installation :

```bash
sudo bash install_freebox_media_vm.sh
```

---

## Première installation

Lors d’une première installation, le script lance une configuration complète :

- paquets système ;
- SSH ;
- firewall ;
- Apache ;
- Plex ;
- rTorrent ;
- ruTorrent ;
- droits sur les disques ;
- SSL optionnel ;
- Plex claim en fin d’installation.

Questions posées au début :

```text
Port SSH souhaité
Disque principal pour les téléchargements rTorrent
Nom de domaine Apache/SSL, vide = pas de SSL
Email Let’s Encrypt, si domaine fourni
```

Le Plex Claim Token est demandé à la fin.

---

## Mode update

Si une installation existante est détectée, le script passe en mode update.

Détection via :

- fichier d’état ;
- dossier ruTorrent ;
- service `rtorrent.service` ;
- installation Plex existante.

Fichier d’état :

```bash
/etc/freebox-media-installer/state.conf
```

Menu update :

```text
1) Tout mettre à jour / reconfigurer
2) Modifier uniquement le mot de passe ruTorrent
3) Mettre à jour uniquement les droits/détection des disques Freebox
4) Reconfigurer rTorrent + ruTorrent + Apache
5) Mettre à jour Plex + vérifier/claim Plex
6) Configurer/renouveler SSL domaine uniquement
7) Choix personnalisé
```

---

## Plex Claim

Le token Plex Claim expire rapidement.

Le script demande donc le token uniquement à la fin, après vérification qu’Apache et Plex répondent localement.

Obtenir un token :

```text
https://plex.tv/claim
```

Si le claim automatique échoue, utiliser la commande suivante immédiatement avec un nouveau token :

```bash
curl -X POST 'http://127.0.0.1:32400/myplex/claim?token=TON_TOKEN'
```

Si l’interface Plex ne voit toujours pas le serveur, utiliser un tunnel SSH :

```bash
ssh -L 32400:127.0.0.1:32400 -p PORT_SSH freebox@IP_VM
```

Puis ouvrir localement :

```text
http://127.0.0.1:32400/web
```

---

## Accès après installation

### ruTorrent

```text
http://IP_VM/rutorrent
```

Utilisateur :

```text
freebox
```

Mot de passe :

```bash
sudo cat /root/freebox-media-credentials.txt
```

### Plex

```text
http://IP_VM:32400/web
```

### Logs

Log installation :

```bash
sudo cat /root/freebox-media-install.log
```

Log ruTorrent :

```bash
cat /tmp/rutorrent-errors.log
```

État rTorrent :

```bash
sudo systemctl status rtorrent.service
```

État Plex :

```bash
sudo systemctl status plexmediaserver.service
```

État Apache :

```bash
sudo systemctl status apache2.service
```

---

## Disques et permissions

Le script ne modifie pas récursivement tous les disques Freebox.

Il applique les droits nécessaires :

- sur les points de montage détectés ;
- sur le dossier applicatif rTorrent ;
- sur l’installation ruTorrent.

Cela évite les erreurs liées à certains dossiers spéciaux comme :

```bash
lost+found
```

Utilisateur principal :

```text
freebox
```

Groupes utilisés :

```text
freebox
www-data
plex
```

---

## Redirections Freebox OS

Pour un accès depuis Internet, configurer les redirections dans Freebox OS.

Exemple :

```text
WAN 80    -> VM Debian 80
WAN 443   -> VM Debian 443
WAN 32400 -> VM Debian 32400
WAN SSH   -> VM Debian port SSH choisi
WAN 50000 -> VM Debian 50000
```

Le port rTorrent `50000` permet aux pairs BitTorrent de se connecter au client.

---

## Sécurité

- `/RPC2` est limité en local dans Apache.
- ruTorrent est protégé par mot de passe.
- Le mot de passe ruTorrent est généré automatiquement.
- Les identifiants sont sauvegardés dans un fichier root-only.
- Le compte root SSH est désactivé.
- UFW bloque les connexions entrantes non autorisées.

Fichier sensible :

```bash
/root/freebox-media-credentials.txt
```

Ne pas publier ce fichier.

---

## Dépannage

### dpkg interrompu

Le script tente automatiquement :

```bash
sudo dpkg --configure -a
sudo apt-get -f install -y
```

Si le problème persiste, lancer manuellement :

```bash
sudo dpkg --configure -a
sudo apt-get -f install
```

### ruTorrent refuse le mot de passe

Réinitialiser uniquement le mot de passe :

```bash
sudo bash install_freebox_media_vm.sh
```

Choisir en mode update :

```text
2) Modifier uniquement le mot de passe ruTorrent
```

### rTorrent ne démarre pas

Vérifier :

```bash
sudo systemctl status rtorrent.service
sudo journalctl -xeu rtorrent.service
```

Vérifier aussi que les disques Freebox sont montés :

```bash
ls /mnt
```

### Plex ne voit pas les fichiers

Vérifier que Plex peut lire les disques :

```bash
sudo -u plex ls /mnt/Freebox
sudo -u plex ls /mnt/Freebox240
```

Si nécessaire, relancer :

```bash
sudo bash install_freebox_media_vm.sh
```

Puis choisir :

```text
3) Mettre à jour uniquement les droits/détection des disques Freebox
```

---

## Mise à jour du script

Télécharger la dernière version :

```bash
wget -O install_freebox_media_vm.sh https://raw.githubusercontent.com/USER/REPO/main/install_freebox_media_vm_final_v10.sh
```

Relancer :

```bash
sudo bash install_freebox_media_vm.sh
```

Le mode update sera détecté automatiquement.

---

## Avertissement légal

Ce script installe rTorrent et ruTorrent pour un usage légal.

L’utilisateur est responsable des fichiers téléchargés, partagés ou hébergés via sa VM.

---

## Licence

MIT License.

Vous pouvez utiliser, modifier et partager ce script librement.
