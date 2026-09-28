# Historique des versions

Ce fichier suit le format [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/), et le projet
respecte le [versionnage sémantique](https://semver.org/lang/fr/).

## [Non publié]

### Ajouté

- Scénarios de diagnostic pour les builds debug (`SIGNAL_SCOUT_DIAGNOSTIC`) : chaque verdict et
  chaque conseil de canal se voient sans dérégler sa box.
- `make page` publie la documentation sur la page de téléchargement entre deux versions.

### Modifié

- L'image disque s'appelle désormais `Signal-Scout.dmg`, sans numéro de version : le bouton
  **Télécharger** de la page mène directement à la dernière version.
- Page de téléchargement refaite : c'est désormais le README complet, avec les captures du menu en
  direct, des emplacements et du diagnostic, les questions fréquentes et le détail des mesures.

## [1.2.0] — 2026-09-28

Signal Scout ne se contente plus de mesurer le signal : elle dit d'où vient un problème de
connexion, et quoi faire.

### Ajouté

- **Diagnostic de la connexion** (⌘D) : délai et pertes vers la box et vers Internet, pour savoir
  si le problème vient du Wi-Fi, de la box ou de la connexion Internet, avec un verdict et un
  conseil.
- **Conseil de canal** : le diagnostic écoute les réseaux voisins, mesure l'encombrement de chaque
  canal (chevauchement en 2,4 GHz, largeur de canal en 5 GHz) et propose un canal plus calme,
  avec la marche à suivre dans les réglages de la box.
- Test de débit à la demande (débits descendant et montant, délai sous charge), via l'outil
  `networkQuality` de macOS.
- **Copier le rapport** : tout le diagnostic en texte, avec les versions de l'app et de macOS.
- **Ouvrir les réglages de la box** depuis la fenêtre du diagnostic.
- Chaque emplacement marqué retient le réseau et la borne captés ; avec plusieurs bornes (répéteur,
  Wi-Fi maillé), le titre de l'emplacement indique laquelle.
- Recherche des mises à jour : une fois par jour si l'option est cochée, ou à la demande avec
  **Rechercher les mises à jour…**.
- Page de téléchargement publique : plus besoin d'accès au dépôt pour installer l'app.
- Publication prête pour la notarisation par Apple (`NOTARY_PROFILE`) : avec un compte Apple
  Developer, les versions suivantes pourront s'ouvrir sans passer par Réglages Système.

### Modifié

- Le SNR et la qualité en pourcentage n'apparaissent qu'une fois vérifié que la carte Wi-Fi mesure
  vraiment le bruit ; s'il reste figé, ils sont masqués et l'option du pourcentage est grisée.
- Le texte de la demande de localisation mentionne aussi les réseaux voisins lus pendant un
  diagnostic.
- La fenêtre de bienvenue propose un troisième réglage, la recherche des mises à jour.

## [1.1.0] — 2026-09-28

*Wi-Fi RSSI* devient **Signal Scout**. Les options et les emplacements enregistrés par la 1.0 sont
conservés tels quels.

### Ajouté

- Installation par image disque (`.dmg`) : Apple silicon et Intel, sans Xcode ni Terminal.
- Icône de l'app.
- Écran de bienvenue au premier lancement : où trouver l'app, lancement à l'ouverture de session
  et affichage du nom du réseau en un clic.
- Menu **À propos de Signal Scout**.
- Rouvrir l'app depuis le Finder, Launchpad ou Spotlight déroule son menu.
- Norme Wi-Fi négociée dans le menu (Wi-Fi 4, 5, 6, 6E, 7).
- Sous-menu par emplacement marqué : SNR, bande, canal, date du marquage, **Renommer…** et
  **Supprimer**.
- Confirmation avant **Supprimer tous les emplacements…**.
- Interface entièrement traduite en français et en anglais, selon la langue du système.
- Libellé VoiceOver sur l'élément de la barre des menus.
- Raccourcis ⌘M (marquer un emplacement) et ⌘R (réinitialiser) dans le menu ouvert.
- Tests unitaires (swift-testing), vérification automatique des traductions (`make l10n`) et du
  formatage (`make lint`).
- Mode simulation pour les builds debug (`SIGNAL_SCOUT_SIMULATE`) : chaque état se teste sans se
  déplacer ni couper le Wi-Fi.
- Build universel Apple silicon + Intel (`UNIVERSAL=1`), signature avec une identité Developer ID
  (`CODESIGN_IDENTITY`), et publication d'une version en une commande (`make release`).

### Modifié

- Projet restructuré en package Swift : un module `SignalScoutCore` sans AppKit et testé, et l'app
  `SignalScout` ; build, installation et désinstallation via `make`. `make install` remplace
  l'ancienne `WiFiRSSI.app`.
- Documentation réorganisée : un README pour les utilisateurs, et pour aller plus loin
  `docs/mesures.md`, `docs/architecture.md` et `CONTRIBUTING.md`.
- Swift 6 avec vérification stricte de la concurrence ; avertissements traités comme des erreurs.
- La mesure d'un emplacement moyenne désormais aussi le SNR sur 10 secondes, comme le RSSI.
- **Afficher le nom du réseau…** indique quand l'accès est déjà accordé, ou mène à Réglages
  Système s'il a été refusé.
- Lancement à l'ouverture de session : si macOS demande une approbation, Réglages Système s'ouvre
  directement au bon endroit.
- **Marquer cet emplacement…** est grisé tant qu'aucun réseau n'est connecté.

### Corrigé

- Supprimer un emplacement supprimait aussi tous ceux portant le même nom.
- Le nom proposé pour un nouvel emplacement pouvait reprendre celui d'un emplacement existant.
- Un bruit non communiqué par le pilote (0 dBm) produisait un SNR et une qualité faux.
- Interface mi-anglaise, mi-française.

## [1.0.0] — 2026-08-10

Première version, sous le nom *Wi-Fi RSSI* : RSSI en direct dans la barre des menus, paliers
colorés, mini-courbe, pic et creux de session, emplacements marqués, lancement à l'ouverture de
session.
