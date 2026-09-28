# Comprendre les mesures

Ce que Signal Scout affiche, d'où viennent les chiffres et comment ils sont calculés. Les valeurs
brutes proviennent de CoreWLAN, le framework Wi-Fi de macOS, interrogé une fois par seconde sur la
connexion en cours, sans recherche de réseaux. Seul le [diagnostic](#diagnostic-de-la-connexion),
lancé à la demande, écoute les réseaux voisins et envoie des pings.

## RSSI

Le *Received Signal Strength Indicator* est la puissance du signal reçu de la borne, en dBm
(décibels-milliwatts). L'échelle est logarithmique et négative : plus la valeur est proche de 0,
plus le signal est fort.

- Perdre 3 dB revient à diviser la puissance reçue par deux.
- Perdre 10 dB revient à la diviser par dix.

C'est pourquoi quelques dBm comptent : passer de -60 à -70 dBm, c'est recevoir dix fois moins de
puissance.

## Verdicts

| Verdict | RSSI | Couleur | En pratique |
| --- | --- | --- | --- |
| Excellent | -50 dBm et plus | Vert | Tout près du point d'accès |
| Très bon | -51 à -60 dBm | Vert | Tout passe sans effort |
| Bon | -61 à -67 dBm | Jaune | -67 dBm est le seuil usuel pour la voix et la visio |
| Faible | -68 à -78 dBm | Orange | Navigation correcte, débit et latence dégradés |
| Mort | -79 dBm et moins | Rouge | Décrochages et pertes de connexion |

Les seuils sont définis à un seul endroit,
`SignalTier.swift`. Un emplacement marqué ne
stocke que son RSSI : son verdict est toujours recalculé avec les seuils en vigueur.

## Bruit, SNR et qualité

- **Bruit de fond** : l'énergie radio ambiante sur le canal, en dBm, typiquement entre -90 et
  -100 dBm. Micro-ondes, réseaux voisins et appareils Bluetooth le font monter.
- **SNR** (*signal-to-noise ratio*, rapport signal/bruit) = RSSI − bruit, en dB. C'est lui qui
  détermine le débit réellement atteignable : un RSSI correct dans un environnement bruyant peut
  donner un lien médiocre.
- **Qualité** : le SNR ramené sur 0–100 %, de façon linéaire entre 5 dB, où plus rien ne passe, et
  45 dB, où le lien est parfait :

  ```text
  qualité = (SNR − 5) / 40 × 100, bornée entre 0 et 100 %
  ```

Certains pilotes ne communiquent pas le bruit (ils renvoient 0 dBm). Le menu affiche alors *Bruit
de fond non communiqué*, sans SNR ni qualité, plutôt que des valeurs fausses. Une valeur hors de
-120 à -50 dBm est écartée de la même façon : des versions de macOS ont déjà renvoyé +24 dBm
([Intuitibits, 2018](https://www.intuitibits.com/2018/08/23/whats-going-on-apple/)).

### Le bruit est-il vraiment mesuré ?

Un pilote peut aussi renvoyer un bruit plausible mais figé, qui ne correspond à aucune mesure ; le
SNR et la qualité seraient alors inventés. Signal Scout le vérifie sur chaque Mac, sans rien
demander :

- tant que rien n'est prouvé, le menu affiche *SNR : vérification de la mesure du bruit par ce
  Mac…*, et ni le pourcentage ni le SNR des emplacements ne sont enregistrés ;
- dès que le bruit change sur un même canal, il est **mesuré** : SNR et qualité s'affichent, et ce
  verdict est définitif ;
- s'il ne bouge pas pendant 15 minutes de connexion sur le même canal, il est **figé** : le menu
  affiche *SNR indisponible*, et l'option du pourcentage est grisée. Un changement ultérieur lève ce
  verdict.

Le verdict est mémorisé : la vérification ne recommence pas à chaque lancement. Sur un MacBook
Pro M1 Max sous macOS 26, le bruit est bien mesuré, mais rafraîchi lentement : il est resté à
-90 dBm près de 30 secondes avant de passer à -91, -93 puis -94 dBm. D'où ce délai de 15 minutes
avant de conclure qu'un bruit est figé.

## Pic, creux et tendance

- **Pic** et **creux** : meilleur et pire RSSI depuis le lancement ou la dernière réinitialisation
  (⌘R). À côté, l'historique détaillé garde les 120 dernières mesures, soit deux minutes, pour la
  courbe, la tendance et la moyenne des emplacements.
- **Écart au pic** : mesure actuelle moins pic. `0 dB` signifie que vous êtes au meilleur niveau
  mesuré ; `-12 dB`, que vous recevez environ seize fois moins de puissance qu'au meilleur endroit.
- **Tendance** : la moyenne des 3 dernières mesures comparée à celle des 3 précédentes. La flèche
  (`↑`, `↓`) ne bouge qu'au-delà de 1,5 dB d'écart, pour ignorer le scintillement de ±1 dB de
  toute radio au repos ; sinon elle reste à `→`.

## Emplacements marqués

**Marquer cet emplacement…** enregistre la moyenne du RSSI et du SNR sur les 10 dernières
secondes, avec la bande et le canal du moment. Si le nom du réseau est accessible (autorisation de
localisation), l'emplacement retient aussi le réseau et l'adresse de la borne (BSSID). La moyenne lisse les variations passagères : restez
immobile une dizaine de secondes avant de marquer.

Les emplacements sont classés du meilleur au pire RSSI, puis par nom.

Avec plusieurs bornes (réseau maillé, répéteur), chaque emplacement porte la lettre de la borne qui
le desservait : *borne A*, *borne B*… Une box émet sous une adresse par bande et par réseau ; ces
adresses se suivent presque toujours, et ne diffèrent que par le dernier chiffre hexadécimal, voire
par le premier octet, que certains fabricants marquent comme « administré localement ». Deux
adresses identiques une fois ces chiffres mis de côté sont donc comptées comme une seule borne, et
un Mac qui passe du 2,4 au 5 GHz de la même box ne fait pas apparaître de seconde lettre.
## Courbe

La courbe de la barre des menus trace les 10 dernières mesures sur 8 niveaux (`▁▂▃▄▅▆▇█`), de
-90 dBm à -30 dBm, soit environ 8,6 dB par niveau. L'échelle est fixe et non adaptée aux données :
une échelle automatique aplatirait justement les différences que l'on cherche en se déplaçant, et
une même hauteur de barre correspond ainsi au même niveau partout.

## Débit, norme et canal

- **Débit** : le débit physique négocié avec la borne. C'est un plafond théorique, pas le débit
  réel d'un téléchargement, qui est souvent deux fois moindre.
- **Norme** :

  | Norme | Nom commercial |
  | --- | --- |
  | 802.11n | Wi-Fi 4 |
  | 802.11ac | Wi-Fi 5 |
  | 802.11ax | Wi-Fi 6, ou Wi-Fi 6E sur la bande 6 GHz |
  | 802.11be | Wi-Fi 7 |

  Les normes plus anciennes (802.11a, b, g) gardent leur nom technique.
- **Bande** : 2,4 GHz porte plus loin et traverse mieux les murs ; 5 et 6 GHz vont plus vite mais
  portent moins loin.
- **Canal et largeur** : où et sur quelle largeur (20 à 160 MHz) le lien émet. Un canal plus large
  va plus vite, mais souffre davantage des interférences.

## Diagnostic de la connexion

**Diagnostiquer la connexion…** (⌘D) répond à la question « c'est mon Wi-Fi ou ma connexion
Internet ? » en une quinzaine de secondes. Tout se passe à la demande : rien ne tourne en
arrière-plan.

### Deux pings, une soustraction

L'app envoie en même temps 30 pings (5 par seconde) :

- **vers la box**, dont l'adresse est celle du routeur que le réseau a donnée à l'interface
  Wi-Fi ;
- **vers Internet**, à `1.1.1.1` (Cloudflare), ou à `8.8.8.8` (Google) si le premier ne répond pas.

Les deux passent par le Wi-Fi, même si un câble Ethernet est branché (`ping -b`), mais seul le
second va au-delà de la box. Le premier mesure donc le Wi-Fi seul, et ce que le second ajoute au
premier revient à la connexion Internet.

Chaque tronçon est jugé sur la **médiane** des allers-retours, et non sur la moyenne : le Wi-Fi d'un
Mac présente des pics réguliers de 20 à 100 ms, quand la radio quitte brièvement son canal pour
AirDrop, Handoff ou une recherche de réseaux, alors que la plupart des réponses arrivent en 3 à
5 ms. Ces pics sont signalés à part (*Réponses retardées*) sans assombrir le verdict.

| Tronçon | Bon | Moyen | Mauvais |
| --- | --- | --- | --- |
| Wi-Fi (Mac → box) | médiane < 8 ms, aucune perte | médiane de 8 à 20 ms, quelques pertes, ou 10 % des réponses au-delà de 100 ms | médiane ≥ 20 ms ou pertes ≥ 5 % |
| Internet (au-delà de la box) | ajout < 40 ms, aucune perte en plus | ajout de 40 à 100 ms, ou quelques pertes en plus | ajout ≥ 100 ms ou pertes en plus ≥ 5 % |

Une box ne perd normalement aucun ping : la radio Wi-Fi renvoie d'elle-même les trames perdues, et
un ping perdu signifie qu'elle y a renoncé.

### Le verdict

| Verdict | Quand |
| --- | --- |
| Tout va bien | Les deux tronçons sont bons |
| Le Wi-Fi est le maillon faible | Le tronçon Wi-Fi est au moins aussi mauvais que le tronçon Internet |
| La connexion Internet est le maillon faible | Le tronçon Internet est le plus mauvais |
| La box n'a pas accès à Internet | La box répond, Internet non |
| Rien ne répond | Ni la box ni Internet |
| Internet fonctionne | Internet répond, mais la box ignore les pings, comme certaines y sont réglées |
| Pas d'adresse de routeur | Le réseau n'a pas donné l'adresse de sa box |

À égalité, le Wi-Fi passe en premier : c'est ce que l'on peut corriger soi-même, et un Wi-Fi
instable ralentit aussi tout ce qui passe derrière lui.

### Le conseil de canal

Après les pings (une recherche de réseaux sort la radio de son canal quelques instants, ce qui
fausserait la mesure), l'app écoute les réseaux voisins et compare le canal de la box à ceux qu'elle
pourrait utiliser :

- **Chaque voisin pèse selon sa force** : rien en dessous de -90 dBm, 1 à partir de -70 dBm,
  proportionnellement entre les deux. Un voisin fort occupe l'antenne à chaque fois qu'il émet ; un
  voisin à peine audible ne gêne presque pas.
- **Et selon son recouvrement.** Sur 2,4 GHz, les canaux sont espacés de 5 MHz mais un signal en
  occupe environ 22 : un voisin à *d* canaux compte pour (22 − 5 *d*) / 22, jusqu'à quatre canaux
  d'écart. Les canaux proposés sont 1, 6 et 11, les seuls qui ne se chevauchent pas. Sur 5 GHz, un
  lien de 40, 80 ou 160 MHz occupe un bloc de canaux (36–48, 52–64, 100–112…) : un voisin compte en
  entier dès que son bloc touche celui de la box.
- **Charge** : la somme de ces poids. En dessous de 1, le canal est *dégagé* ; de 1 à 3,
  *partagé* ; au-delà, *encombré*.
- **Un autre canal n'est conseillé que s'il retire au moins un voisin fort et un tiers de la
  charge.** Changer de canal coupe le Wi-Fi quelques secondes : un gain marginal n'en vaut pas la
  peine.
- Les canaux 52 à 144 partagent la bande avec des radars météo (DFS) : la box écoute une minute
  avant d'en utiliser un, et en change seule si elle détecte un radar. Ils ne sont conseillés
  qu'avec une demi-unité de charge d'avance, et les canaux à partir de 149, que toutes les box et
  tous les pays n'autorisent pas, avec une unité d'avance. Seuls les canaux que la
  réglementation locale permet à ce Mac sont proposés.
- Sur 6 GHz, la place ne manque pas : aucun changement n'est conseillé.

Le réseau de la box elle-même, ses autres bandes et les bornes d'un même réseau maillé sont retirés
des voisins d'après leur nom. Sans autorisation de localisation, macOS masque les noms : l'app
écarte alors le réseau entendu sur le même canal à la force la plus proche de la connexion.

La mesure se fait depuis le Mac, pas depuis la box. Dans un logement, les deux entendent en général
les mêmes voisins ; pour un conseil plus sûr, lancez le diagnostic près de la box.

### Le débit

**Mesurer** lance `networkQuality`, l'outil de test de débit intégré à macOS, sur l'interface
Wi-Fi, pendant 20 secondes au plus, vers les serveurs d'Apple. Il donne :

- le débit descendant et montant, en Mbit/s ;
- la **latence sous charge** : le délai d'un aller-retour pendant que la connexion est saturée.
  macOS l'exprime en allers-retours par minute (*RPM*) ; l'app le convertit en millisecondes
  (60 000 / RPM). Une valeur élevée (plusieurs centaines de ms) explique qu'un appel vidéo hache dès
  que quelqu'un télécharge.

Le test sollicite la connexion à fond : évitez-le sur un partage de connexion limité.
