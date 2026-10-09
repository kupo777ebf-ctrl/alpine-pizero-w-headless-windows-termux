<div align="center">

# Alpine Linux sur Raspberry Pi Zero W

### Installation headless avec Windows, Termux, Nmap et SSH

**Raspberry Pi Zero W v1.1 · ARMv6 · Wi-Fi 2,4 GHz · Alpine Linux `armhf`**

[Préparer la microSD](#-1-préparer-la-microsd-sous-windows) ·
[Premier démarrage](#-2-démarrer-le-raspberry-pi) ·
[Détecter le Pi](#-3-trouver-le-raspberry-pi-avec-termux-et-nmap) ·
[Se connecter en SSH](#-4-se-connecter-en-ssh) ·
[Configurer Alpine](#-5-configurer-alpine-à-distance)

</div>

---

## Présentation

Ce dépôt décrit une installation **sans écran, clavier ni câble série** d'Alpine Linux sur un Raspberry Pi Zero W v1.1. La préparation de la carte se fait sur **Windows** ; toutes les opérations après le démarrage se font depuis **Termux sur Android**, au moyen de Nmap et SSH.

Le démarrage initial utilise l'overlay headless du projet [macmpi/alpine-linux-headless-bootstrap](https://github.com/macmpi/alpine-linux-headless-bootstrap). Cet overlay permet de démarrer temporairement un serveur SSH avant la configuration habituelle d'Alpine.

> [!IMPORTANT]
> Cette procédure vise le **Raspberry Pi Zero W de première génération (ARMv6)**, pas le Pi Zero 2 W. Vérifie le modèle de la carte avant de commencer.

## Sommaire

- [Matériel et logiciels](#-matériel-et-logiciels)
- [Pourquoi Alpine et armhf ?](#-pourquoi-alpine-et-armhf-)
- [1. Préparer la microSD sous Windows](#-1-préparer-la-microsd-sous-windows)
- [2. Démarrer le Raspberry Pi](#-2-démarrer-le-raspberry-pi)
- [3. Trouver le Raspberry Pi avec Termux et Nmap](#-3-trouver-le-raspberry-pi-avec-termux-et-nmap)
- [4. Se connecter en SSH](#-4-se-connecter-en-ssh)
- [5. Configurer Alpine à distance](#-5-configurer-alpine-à-distance)
- [6. Sécuriser SSH](#-6-sécuriser-ssh)
- [7. Sauvegarder la configuration](#-7-sauvegarder-la-configuration)
- [Dépannage](#-dépannage)
- [Références](#-références)

## 🧰 Matériel et logiciels

### Matériel

- Raspberry Pi Zero W v1.1.
- Carte microSD ; 512 Mo est la taille minimale mentionnée dans le guide d'origine, mais une carte plus grande est recommandée pour davantage de marge.
- Alimentation micro-USB 5 V stable.
- PC Windows avec lecteur microSD.
- Téléphone Android avec Termux.
- Réseau Wi-Fi **2,4 GHz** auquel le Pi et le téléphone peuvent se connecter.

### Logiciels

| Outil | Utilité | Lien |
|---|---|---|
| 7-Zip | Extraire l'archive Alpine | [7-zip.org](https://www.7-zip.org/) |
| Notepad++ (facultatif) | Éditer les fichiers texte sans changer les fins de ligne | [notepad-plus-plus.org](https://notepad-plus-plus.org/) |
| Termux | Terminal Android | [Page officielle du projet](https://github.com/termux/termux-app) |
| Nmap | Repérer le Pi sur le réseau local | Installé depuis Termux |
| OpenSSH | Se connecter au Pi | Installé depuis Termux |

> [!NOTE]
> Installe Termux depuis une source officielle du projet et conserve la même source pour ses éventuels modules complémentaires. Les versions d'une même application provenant de sources différentes ne sont pas toujours compatibles.

## 🧠 Pourquoi Alpine et `armhf` ?

Le Raspberry Pi Zero W v1.1 utilise un SoC BCM2835 avec un processeur ARMv6 et 512 Mo de RAM. Il faut donc choisir une image Alpine adaptée à cette génération.

- **À utiliser :** l'archive Raspberry Pi `armhf`.
- **À ne pas utiliser :** `aarch64`, qui vise ARM 64 bits, ou une image exigeant ARMv7.
- Le Wi-Fi intégré fonctionne en **2,4 GHz** ; le Pi Zero W n'a pas de port Ethernet intégré.

Cette procédure utilise l'archive Alpine pour Raspberry Pi, qui est une **archive de fichiers** et non une image disque à flasher avec Etcher.

> [!WARNING]
> Les versions et noms de fichiers peuvent évoluer. Cette documentation prend comme exemple Alpine **3.24.2**. Avant de commencer, vérifie la version disponible sur la page officielle de téléchargement et adapte les liens si nécessaire.

## 🪟 1. Préparer la microSD sous Windows

### 1.1 Télécharger Alpine Linux

Télécharge l'archive `armhf` destinée aux Raspberry Pi :

- [Archive Alpine 3.24.2 `armhf`](https://dl-cdn.alpinelinux.org/alpine/v3.24/releases/armhf/alpine-rpi-3.24.2-armhf.tar.gz)
- [Répertoire des versions `armhf`](https://dl-cdn.alpinelinux.org/alpine/v3.24/releases/armhf/)

Enregistre-la dans un dossier facile à retrouver, par exemple `Téléchargements`.

### 1.2 Préparer la carte

1. Insère la microSD dans le PC.
2. Ouvre **Gestion des disques** (`Win + X` → Gestion des disques).
3. Identifie la carte par sa capacité. **Ne sélectionne pas le disque Windows.**
4. Si tu souhaites réutiliser une carte qui contient déjà des données, sauvegarde-les avant toute modification.
5. Crée une partition principale FAT32 occupant l'espace disponible, si Windows ou un outil de partitionnement permet de le faire.

Windows ne propose pas toujours FAT32 pour les supports de grande capacité via son interface intégrée. Si nécessaire, utilise un outil de partitionnement fiable. N'efface jamais une partition tant que tu n'as pas confirmé qu'il s'agit bien de la microSD.

### 1.3 Extraire l'archive à la racine de la carte

1. Installe 7-Zip.
2. Ouvre l'archive `.tar.gz`, puis ouvre l'archive `.tar` qu'elle contient.
3. Extrais **le contenu** directement à la racine de la partition FAT32 de la microSD.

La racine de la carte doit contenir les fichiers de démarrage, par exemple `config.txt`, `cmdline.txt` et le fichier d'arbre de périphériques correspondant au Pi Zero W.

Ne crée pas un dossier supplémentaire tel que `SD:\alpine-rpi\` contenant tous les fichiers : ils doivent être à la racine de la partition de démarrage.

Vérifie la présence de :

```text
bcm2835-rpi-zero-w.dtb
config.txt
cmdline.txt
```

Si `bcm2835-rpi-zero-w.dtb` est absent, arrête-toi et vérifie l'archive extraite.

### 1.4 Ajouter `usercfg.txt`

À la racine de la carte, crée un fichier nommé exactement `usercfg.txt`, contenant :

```ini
gpu_mem=16
```

Dans l'Explorateur, active **Affichage → Afficher → Extensions de noms de fichiers** afin de ne pas créer accidentellement `usercfg.txt.txt`.

N'ajoute pas d'autres options de démarrage au hasard. Commence avec cette configuration minimale.

### 1.5 Ajouter l'overlay headless

Télécharge le fichier `headless.apkovl.tar.gz` depuis le dépôt du projet :

[macmpi/alpine-linux-headless-bootstrap](https://github.com/macmpi/alpine-linux-headless-bootstrap)

Le lien direct du fichier publié est :

[ Télécharger `headless.apkovl.tar.gz`](https://github.com/macmpi/alpine-linux-headless-bootstrap/raw/main/headless.apkovl.tar.gz)

Copie `headless.apkovl.tar.gz` **à la racine** de la partition FAT32, à côté de `config.txt` et `cmdline.txt`.

Cet overlay est fourni par un projet tiers. Consulte son README et ses notes de sécurité avant de l'utiliser.

### 1.6 Configurer le Wi-Fi

Copie le modèle [`configs/wpa_supplicant.conf.example`](configs/wpa_supplicant.conf.example) dans un fichier nommé `wpa_supplicant.conf` à la racine de la microSD.

Exemple de contenu :

```conf
country=FR
network={
    key_mgmt=WPA-PSK
    ssid="NOM_DU_WIFI"
    psk="MOT_DE_PASSE_WIFI"
}
```

Remplace les deux valeurs par celles de ton réseau. Le SSID et le mot de passe sont sensibles à la casse.

**Sécurité :** ce format stocke le mot de passe Wi-Fi en clair sur la carte. Ne partage jamais cette carte, ce fichier ou une capture d'écran qui révèle ces informations. Le fichier d'exemple dans ce dépôt ne contient aucune vraie information de connexion.

Si tu utilises un éditeur de texte, enregistre le fichier en texte brut et, si possible, avec des fins de ligne Unix (LF). Évite le format RTF ou Word.

### 1.7 Vérification finale de la carte

La racine de la partition doit maintenant ressembler à ceci :

```text
SD:\
├── bcm2835-rpi-zero-w.dtb
├── cmdline.txt
├── config.txt
├── usercfg.txt
├── headless.apkovl.tar.gz
└── wpa_supplicant.conf
```

D'autres fichiers Alpine seront également présents : ne les supprime pas.

Avant d'éjecter la carte :

- Vérifie une dernière fois le nom des fichiers.
- Vérifie que le SSID et le mot de passe sont corrects.
- Vérifie que le Pi et le téléphone utiliseront le même réseau local.
- Éjecte la carte proprement depuis Windows.

## 🔌 2. Démarrer le Raspberry Pi

1. Insère la microSD dans le Pi Zero W.
2. Branche une alimentation stable sur le port micro-USB **PWR IN**.
3. Attends environ deux minutes au premier démarrage.
4. Laisse le Pi démarrer et rejoindre le Wi-Fi.

Le premier démarrage peut prendre plus de temps qu'un démarrage ultérieur. Le voyant d'activité n'est pas une preuve suffisante que le Wi-Fi ou SSH fonctionne.

> [!CAUTION] 
> L'overlay de démarrage permet un accès SSH temporaire avec le compte `root` sans mot de passe. Utilise-le uniquement pour l'installation initiale, sur un réseau local de confiance. Ne redirige jamais le port SSH vers Internet. Passe à un compte normal et à une authentification plus sûre dès que possible.

## 📱 3. Trouver le Raspberry Pi avec Termux et Nmap

### 3.1 Installer les outils

Ouvre Termux et exécute :

```sh
pkg update
pkg install nmap openssh
```

Vérifie qu'ils sont disponibles :

```sh
nmap --version
ssh -V
```

### 3.2 Identifier le sous-réseau du téléphone

Exécute :

```sh
ip route
```

Repère une route de réseau local. Par exemple, une route `192.168.1.0/24` indique que le sous-réseau à examiner est `192.168.1.0/24`.

Les adresses changent selon le routeur. **Ne copie pas automatiquement l'adresse d'exemple** : utilise le sous-réseau de ton réseau actuel.

Si `ip route` n'affiche pas suffisamment d'informations, consulte les détails Wi-Fi d'Android ou la liste des appareils connectés dans l'interface du routeur.

### 3.3 Scanner le réseau local

Remplace le sous-réseau d'exemple par le tien :

```sh
nmap -sn 192.168.1.0/24
```

`-sn` demande une découverte des hôtes sans scan de ports. Sur certains réseaux, un appareil peut ne pas répondre aux sondes de découverte ; son absence dans les résultats ne prouve donc pas qu'il est éteint.

Tu peux aussi regarder les appareils connectés depuis l'interface du routeur. Repère une nouvelle adresse IP apparue au moment du démarrage du Pi.

### 3.4 Vérifier le port SSH

Une fois une adresse IP candidate trouvée, teste uniquement le port SSH du Pi :

```sh
nmap -p 22 192.168.1.123
```

Remplace `192.168.1.123` par l'adresse réellement trouvée.

- `22/tcp open` : un service écoute sur le port SSH.
- `22/tcp closed` : l'appareil répond, mais aucun service n'écoute sur ce port.
- `filtered` ou aucune réponse : le réseau ou l'appareil peut bloquer les sondes.

N'effectue ces scans que sur ton propre réseau ou un réseau pour lequel tu as l'autorisation.

## 🔐 4. Se connecter en SSH

### 4.1 Première connexion

L'overlay headless de macmpi indique que l'accès initial se fait avec `root` sans mot de passe. Remplace l'adresse ci-dessous par celle du Pi :

```sh
ssh root@192.168.1.123
```

À la première connexion, SSH peut demander de confirmer l'empreinte de la clé d'hôte. L'overlay de démarrage peut utiliser des clés temporaires connues publiquement : **ne considère pas cette première empreinte comme une identité permanente et vérifiée du Pi**. Continue uniquement sur ton réseau privé de confiance et remplace ces clés durant la configuration finale.

Si SSH demande un mot de passe alors que tu t'attends au comportement de l'overlay, n'essaie pas des mots de passe au hasard. Reviens à la section [Dépannage](#-dépannage).

### 4.2 Vérifier la session

Une fois connecté :

```sh
cat /etc/alpine-release
uname -m
ip addr
```

Ces commandes affichent la version d'Alpine, l'architecture rapportée par le noyau et les interfaces réseau.

## ⚙️ 5. Configurer Alpine à distance

> [!IMPORTANT]
> Les réponses de `setup-alpine` dépendent de l'état réel du système et de la version Alpine. Lis chaque question avant de répondre. Ne choisis pas une option de partitionnement qui effacerait la carte.

### 5.1 Lancer l'assistant

```sh
setup-alpine
```

Pour un Pi Zero W utilisé en Wi-Fi, les choix habituels sont :

| Question | Choix indicatif |
|---|---|
| Hostname | `pizw` |
| Interface réseau | `wlan0` si proposée |
| Adresse IPv4 | DHCP |
| Fuseau horaire | `Europe/Paris` si adapté |
| Serveur SSH | `openssh` |
| Mode disque | `diskless` si tu veux conserver le système en RAM |
| Support de sauvegarde de configuration | La partition de la microSD proposée par Alpine |

Les libellés exacts peuvent varier. Configure le réseau avec les informations de ton Wi-Fi et définis un mot de passe root fort pour l'administration locale, même si tu comptes désactiver ensuite la connexion SSH directe de root.

### 5.2 Corriger l'heure et vérifier les dépôts

Une horloge incorrecte peut provoquer des erreurs TLS lors de l'accès aux dépôts. Vérifie d'abord :

```sh
date
```

Si l'heure est incorrecte, configure la synchronisation NTP selon les services disponibles dans ta version Alpine. Puis vérifie les dépôts configurés :

```sh
cat /etc/apk/repositories
```

Utilise les dépôts officiels correspondant à la version d'Alpine installée. Ne mélange pas des dépôts de versions différentes.

Quand la connexion réseau et l'heure sont correctes, actualise l'index des paquets :

```sh
apk update
```

### 5.3 Installer les outils nécessaires

Pour une installation minimale, commence par le serveur SSH si `setup-alpine` ne l'a pas déjà installé :

```sh
apk add openssh
rc-update add sshd default
rc-service sshd start
```

Si le paquet ou le service est déjà configuré, vérifie son état plutôt que de multiplier les modifications.

## 🛡️ 6. Sécuriser SSH

### 6.1 Créer un utilisateur normal

Sur le Pi, crée un utilisateur personnel. Exemple :

```sh
adduser piuser
addgroup piuser wheel
```

Choisis un nom différent si tu le souhaites. Le groupe `wheel` est utile pour l'administration selon la configuration choisie ; vérifie les outils d'élévation de privilèges installés avant de compter dessus.

Ouvre une deuxième session Termux et teste la connexion de ce nouvel utilisateur avant de désactiver l'accès root.

### 6.2 Générer une clé SSH dans Termux

Sur **Termux**, pas dans la session SSH du Pi :

```sh
ssh-keygen -t ed25519
```

Accepte le chemin proposé et définis une phrase secrète si tu le souhaites. Affiche ensuite la clé publique :

```sh
cat ~/.ssh/id_ed25519.pub
```

La clé publique peut être installée sur le Pi. **Ne partage jamais le fichier privé `~/.ssh/id_ed25519`.**

### 6.3 Installer la clé publique sur le Pi

Connecte-toi au Pi avec le compte d'administration encore accessible, puis remplace `piuser` par ton utilisateur :

```sh
mkdir -p /home/piuser/.ssh
chmod 700 /home/piuser/.ssh
```

Colle le contenu complet de `id_ed25519.pub` dans `authorized_keys` :

```sh
cat > /home/piuser/.ssh/authorized_keys
```

Colle la clé publique, puis termine l'entrée avec `Ctrl+D`.

Applique les permissions :

```sh
chmod 600 /home/piuser/.ssh/authorized_keys
chown -R piuser:piuser /home/piuser/.ssh
```

Teste la connexion depuis une **nouvelle session Termux** :

```sh
ssh -i ~/.ssh/id_ed25519 piuser@192.168.1.123
```

Ne ferme pas la session d'administration initiale tant que cette nouvelle connexion n'a pas fonctionné.

### 6.4 Désactiver la connexion SSH directe de root

Après avoir confirmé que la connexion par clé fonctionne, édite `/etc/ssh/sshd_config` et vérifie ces paramètres :

```text
PermitRootLogin no
PubkeyAuthentication yes
```

Tu peux utiliser `vi` si aucun éditeur plus convivial n'est installé :

```sh
vi /etc/ssh/sshd_config
```

Vérifie la configuration puis redémarre SSH :

```sh
sshd -t
rc-service sshd restart
```

**Ne désactive pas l'authentification par mot de passe avant d'avoir testé la clé dans une deuxième session.** Si tu actives `PasswordAuthentication no`, assure-toi d'abord que l'authentification par clé fonctionne et que tu as une méthode de récupération locale.

## 💾 7. Sauvegarder la configuration

En mode *diskless*, les changements effectués en RAM ne sont pas tous conservés après un redémarrage. Alpine utilise `lbu` pour sauvegarder la configuration.

Vérifie d'abord :

```sh
lbu status
```

Puis, lorsque la configuration est prête :

```sh
lbu commit
```

Le paramètre exact et la destination dépendent de la configuration créée par `setup-alpine`. Si la sauvegarde locale est configurée sur la partition de la microSD et que la documentation de ta version le demande, utilise `lbu commit -d`.

Vérifie que les fichiers importants à conserver — notamment la configuration SSH et les clés publiques autorisées — sont inclus dans la sauvegarde. Ne sauvegarde pas de secrets dans ce dépôt Git.

Avant le redémarrage :

1. Confirme que le nouvel utilisateur se connecte.
2. Confirme que SSH démarre automatiquement.
3. Sauvegarde avec `lbu`.
4. Redémarre et teste à nouveau la connexion.

## 🧯 Dépannage

### Le Pi n'apparaît pas dans Nmap

- Vérifie que le Pi est alimenté par le port `PWR IN`.
- Attends au moins deux minutes au premier démarrage.
- Vérifie que le SSID, le mot de passe et `country=FR` sont corrects.
- Assure-toi que le réseau Wi-Fi est en 2,4 GHz.
- Vérifie que le téléphone et le Pi sont sur le même réseau et que l'isolation des clients Wi-Fi est désactivée.
- Consulte la liste des appareils connectés dans le routeur.
- Vérifie que l'archive Alpine correspond bien au Pi Zero W v1.1.

### Le Pi apparaît, mais le port 22 n'est pas ouvert

- Vérifie que `headless.apkovl.tar.gz` se trouve à la racine de la partition de démarrage.
- Vérifie le nom et le format de `wpa_supplicant.conf`.
- Vérifie l'alimentation et laisse le temps au démarrage de se terminer.
- Si tu as déjà configuré Alpine, vérifie le service `sshd` sur le Pi.

### `Permission denied` lors de la connexion SSH

- Vérifie le nom d'utilisateur.
- Vérifie que la clé publique est dans le fichier `authorized_keys` de cet utilisateur.
- Vérifie les permissions du dossier `.ssh` et du fichier `authorized_keys`.
- Utilise `ssh -v utilisateur@IP` pour obtenir davantage d'informations.

### Avertissement `REMOTE HOST IDENTIFICATION HAS CHANGED`

Cela peut être normal si le Pi a été réinstallé ou si ses clés SSH ont été renouvelées, mais cela peut aussi indiquer que l'appareil à cette adresse n'est pas celui attendu. Vérifie l'identité du Pi et ses nouvelles clés par un canal fiable avant de supprimer une ancienne entrée de `known_hosts`. Ne contourne pas cet avertissement sans vérification.

### La configuration disparaît après redémarrage

En mode diskless, vérifie que `lbu commit` a été exécuté et que les fichiers nécessaires sont inclus dans la sauvegarde locale.

## 📚 Références

- [Alpine Linux — Téléchargements](https://alpinelinux.org/downloads/)
- [Alpine Linux Wiki — Raspberry Pi](https://wiki.alpinelinux.org/wiki/Raspberry_Pi)
- [Alpine Linux Wiki — Installation sur un hôte headless](https://wiki.alpinelinux.org/wiki/Installation_on_a_headless_host)
- [Alpine Linux Wiki — Sauvegarde locale (`lbu`)](https://wiki.alpinelinux.org/wiki/Alpine_local_backup)
- [macmpi — Alpine Linux Headless Bootstrap](https://github.com/macmpi/alpine-linux-headless-bootstrap)
- [Nmap — Référence officielle](https://nmap.org/book/man.html)
- [OpenSSH — Pages de manuel](https://www.openssh.com/manual.html)

## Licence et attribution

Cette documentation est une adaptation destinée à un flux de travail **Windows → Termux**. Elle s'appuie sur les concepts et ressources documentés dans le dépôt [KingsukMajumdar/alpine-pizero-w-headless](https://github.com/KingsukMajumdar/alpine-pizero-w-headless) et sur l'overlay tiers de [macmpi](https://github.com/macmpi/alpine-linux-headless-bootstrap).

Consulte [`LICENSE`](LICENSE) pour les conditions de réutilisation de cette adaptation. L'overlay Alpine headless, Alpine Linux et les outils cités restent soumis à leurs licences respectives.
