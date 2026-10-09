# Checklist d'installation

## Windows — carte microSD

- [ ] Vérifier qu'il s'agit du Raspberry Pi Zero W v1.1.
- [ ] Télécharger l'archive Alpine `armhf` depuis le site officiel.
- [ ] Extraire le contenu de l'archive à la racine de la partition FAT32.
- [ ] Vérifier la présence de `bcm2835-rpi-zero-w.dtb`.
- [ ] Créer `usercfg.txt` avec `gpu_mem=16`.
- [ ] Copier `headless.apkovl.tar.gz` à la racine.
- [ ] Créer `wpa_supplicant.conf` avec les paramètres Wi-Fi locaux.
- [ ] Éjecter proprement la carte.

## Termux — découverte et SSH

- [ ] Installer `nmap` et `openssh`.
- [ ] Identifier le sous-réseau Wi-Fi.
- [ ] Découvrir le Pi avec `nmap -sn`.
- [ ] Vérifier le port 22.
- [ ] Se connecter sur le réseau local de confiance.

## Configuration Alpine

- [ ] Lancer `setup-alpine` et vérifier chaque choix.
- [ ] Vérifier l'heure et les dépôts.
- [ ] Créer un utilisateur normal.
- [ ] Créer une clé SSH dans Termux.
- [ ] Installer la clé publique sur le Pi.
- [ ] Tester la connexion dans une deuxième session.
- [ ] Désactiver l'accès SSH direct de root.
- [ ] Sauvegarder avec `lbu`.
- [ ] Redémarrer et vérifier que la connexion persiste.
