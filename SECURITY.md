# Sécurité

## Avant le premier démarrage

- Ne publiez jamais un vrai fichier `wpa_supplicant.conf`.
- Ne publiez jamais de clé SSH privée, de mot de passe, de jeton ou d'adresse IP publique associée à un service privé.
- L'overlay headless tiers permet un accès initial SSH temporaire avec le compte `root` sans mot de passe. N'utilisez cette méthode que sur un réseau local de confiance.
- N'exposez pas le port SSH du Raspberry Pi à Internet pendant le bootstrap.

## Après la configuration

1. Créez un utilisateur normal.
2. Installez et testez une clé SSH personnelle.
3. Renouvelez les clés d'hôte SSH temporaires du bootstrap.
4. Désactivez la connexion SSH directe de root.
5. Désactivez l'authentification par mot de passe seulement après avoir testé l'accès par clé.
6. Sauvegardez les changements avec `lbu` si le système est en mode diskless.

## Fichiers sensibles

Le fichier Wi-Fi d'exemple ne contient que des valeurs fictives. Gardez vos identifiants dans une copie locale non suivie par Git. La clé privée SSH doit rester sur le client Termux.

## Signalement

Ce dépôt est une documentation communautaire. Pour signaler un problème de sécurité lié à cette documentation, ouvrez une issue sans publier de secrets, de clés privées ou d'informations réseau sensibles.
