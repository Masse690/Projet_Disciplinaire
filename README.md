# Projet Smart Campus

Projet de terminal connecté intelligent développé dans le cadre du **Projet Disciplinaire M2** à l'Université Côte d'Azur.

Ce dépôt rassemble les documents de conception d'un nœud IoT alimenté par récupération d'énergie solaire. Le démonstrateur associera un module LoRaWAN RAK3172, un gestionnaire d'énergie bq25570, un stockage d'énergie et un ou plusieurs capteurs pour surveiller un environnement du campus.

> **Statut actuel :** sujet, architecture de base et datasheets disponibles. Le firmware, la conception KiCad, le PCB, l'antenne, le boîtier 3D et les résultats de démonstration restent à ajouter.

---

## Vue d'ensemble

L'objectif du projet est de concevoir un objet connecté autonome, depuis le choix des composants jusqu'à sa validation sur le réseau LoRaWAN.

Le projet couvre les étapes suivantes :

- conception du hardware avec KiCad ;
- intégration de capteurs ;
- développement du firmware embarqué ;
- conception et optimisation de l'antenne ;
- création d'un boîtier imprimé en 3D ;
- alimentation par panneau solaire ;
- stockage de l'énergie dans un supercondensateur ou une batterie ;
- fabrication du PCB ;
- assemblage des composants ;
- déploiement d'une démonstration sur une durée de une à deux semaines ;
- validation des communications et de la consommation énergétique.

### Architecture cible

```text
Panneau solaire
      |
      v
   bq25570
   +------+
   |      +--> stockage d'énergie
   |      |    (supercondensateur ou batterie)
   |      |
   |      +--> alimentation 3,3 V
   +------+
      |
      v
+-------------------+
| RAK3172 / STM32WL|<--> antenne LoRa 868 MHz
+-------------------+
      |
      +--> capteur(s) I2C/UART
```

---

## Équipe

| Nom | Rôle |
|---|---|
| Noé Massé | Firmware, électronique, capteurs, réseau |
| Alec Passolunghi | Conception PCB, boîtier 3D |
| Matis Chevreaux | Antenne |

**Encadrement :** 
- Fabien Ferrero — Université Côte d'Azur / CNRS / LEAT
- Dorian Diana — Université Côte d'Azur / LEAT
---

## Cas d'application

Le sujet propose l'utilisation d'au moins un capteur spécifique au projet. Les ressources initiales du dépôt comprennent une datasheet pour le capteur de gaz MQ-2.

Usages possibles :

- détection de fumée dans une salle ;
- surveillance de la qualité de l'air ;
- détection de fuite de gaz ;
- alerte de sécurité dans un bâtiment ;

L'application retenue, .... à definir

---

## Architecture matérielle

### Contrôleur et radio

Le **RAK3172** est un module LoRaWAN compact basé sur le microcontrôleur STM32WLE5CC.

Caractéristiques principales :

- cœur ARM Cortex-M4 jusqu'à 48 MHz ;
- radio LoRa / LoRaWAN ;
- support LoRaWAN 1.0.3 ;
- modes OTAA et ABP ;
- communication P2P ;
- interfaces UART, I2C et GPIO ;
- tension d'alimentation comprise entre 2,0 V et 3,6 V ;
- connexion à une antenne 868 MHz par réseau RF dédié.

Exemples de consommations indiquées dans la datasheet :

| Mode | Valeur indicative |
|---|---:|
| Sleep | 1,69 µA |
| Réception | 5,22 mA |
| Émission | environ 87 mA à 20 dBm |

La valeur finale devra être mesurée sur le prototype assemblé.

### Gestion d'énergie

Le **bq25570** est un chargeur ultra-faible consommation pour applications alimentées par récupération d'énergie.

Fonctions principales :

- démarrage à froid à partir d'environ 600 mV ;
- fonctionnement normal à partir d'environ 100 mV ;
- suivi du point de puissance maximale programmable ;
- protection de la tension de batterie ;
- sortie buck régulée ;
- surcharge, sous-charge et gestion du stockage.

L'étage bq25570 doit permettre d'alimenter la charge électronique pendant les périodes de faible ensoleillement grâce à un stockage tampon.

### Stockage d'énergie

Deux solutions peuvent être utilisées :

- supercondensateur ;
- batterie rechargeable.

La datasheet du bq25570 indique qu'une capacité équivalente d'au moins 100 µF est nécessaire. Le stockage, sa tension maximale, sa capacité utile et ses limites de charge devront être choisis selon le profil d'utilisation.

### Capteur MQ-2

Le MQ-2 est un capteur semi-conducteur sensible aux gaz combustibles et à la fumée.

Gaz détectables :

- GPL ;
- propane ;
- butane ;
- méthane ;
- hydrogène ;
- alcool ;
- fumée ;
- monoxyde de carbone.

Considérations d'intégration :

- alimentation/heater : environ 5 V ;
- consommation du heater : inférieure à 800 mW ;
- résistance de détection typique : 3 à 30 kΩ ;
- préchauffage recommandé : au moins 24 heures ;
- calibration nécessaire avec des concentrations de gaz connues ;
- densité de gaz à confirmer dans l'utilisation finale.

Le MQ-2 est principalement adapté à une détection relative. Les valeurs brutes doivent être calibrées plutôt que directement interprétées comme une concentration précise.

---

## Développement logiciel

La plateforme peut être programmée depuis l'environnement Arduino ou depuis les outils ST, selon le firmware retenu.

Fonctions prévues :

- initialisation du RAK3172 ;
- activation LoRaWAN ;
- acquisition périodique des capteurs ;
- conversion et calibration des mesures ;
- gestion des valeurs limites ;
- transmission via OTAA ou ABP ;
- gestion des accusés de réception ;
- configuration des périodes de sommeil ;
- adaptation du taux de transmission ;
- surveillance de la tension de stockage ;
- journalisation des événements.

Des exemples de scripts, d'envoi de données et de visualisation pourront être ajoutés dans ce dépôt lorsque l'application sera implémentée.

---

## Communications LoRaWAN

Le nœud pourra joindre un réseau LoRaWAN en utilisant OTAA ou ABP.

Les informations suivantes devront être définies pour chaque déploiement :

- région LoRaWAN ;
- identifiants OTAA ou clés ABP ;
- classe de terminal ;
- fréquence d'émission ;
- facteur d'étalement ;
- bande passante ;
- cycle de mesure ;
- payload ;
- serveur réseau ;
- application Node-RED ou serveur de visualisation.

Les clés réseau ne doivent jamais être publiées dans ce dépôt. Elles devront être stockées dans un fichier ignoré par Git ou injectées localement lors de la compilation.

---

## Antenne et conception du boîtier

### Antenne

La carte devra intégrer une solution adaptée à la fréquence choisie, généralement 868 MHz en Europe.

Travaux à réaliser :

- étude de l'impédance ;
- choix du type d'antenne ;
- optimisation dans son environnement réel ;
- mesure du diagramme ou de l'adaptation si un équipement est disponible ;
- distance entre l'antenne, le plan de masse, le panneau solaire et les composants sensibles.

### Boîtier

Le boîtier imprimé en 3D devra :

- protéger l'électronique ;
- permettre le passage ou l'intégration de l'antenne ;
- protéger le capteur tout en laissant l'air circuler ;
- faciliter l'orientation du panneau solaire ;
- permettre l'accès aux connecteurs de programmation ;
- intégrer des éventuelles fixations pour le Smart Campus.

---

## Conception PCB

La carte finale sera conçue avec KiCad et réalisée pour l'assemblage du démonstrateur.

Livrables attendus :

- schéma électrique ;
- annotation des composants ;
- liste BOM ;
- routage du PCB ;
- fichiers Gerber ;
- fichier de perçage ;
- fichiers Pick-and-Place ;
- plan de masse ;
- respect de l'intégrité RF ;
- plan d'essai et de programmation ;
- documentation d'assemblage.

Ces fichiers seront ajoutés au dépôt une fois la première révision validée.

---

## Structure du dépôt

```text
.
├── Data_sheet/                 # Datasheets des composants
│   ├── 605-00008-MQ-2-Datasheet-370464.pdf
│   ├── bq25570.pdf
│   └── RAK3172.pdf
├── Projet SMART CAMPUS_27.pdf  # Sujet du projet
└── README.md
```

Structure cible :

```text
.
├── hardware/
│   ├── schematics/
│   ├── kicad/
│   ├── gerber/
│   ├── bom/
│   └── 3d/
├── datasheets/
├── firmware/
├── application/
├── tests/
├── photos/
└── README.md
```

---

## Tests et validation

Les validations prévues comprennent :

- vérification du schéma et du routage ;
- test de démarrage du bq25570 ;
- mesure des tensions de stockage ;
- test du buck 3,3 V ;
- test de préchauffage du MQ-2 ;
- calibration du capteur ;
- vérification du courant sommeil ;
- mesure du courant de réception ;
- mesure du courant d'émission ;
- test OTAA ou ABP ;
- validation de la portée ;
- démonstration extérieure ;
- analyse de l'autonomie sur plusieurs jours ;
- mesures de consommation avec les temporisations réelles.

Les résultats seront accompagnés de captures, tableaux ou graphiques dès qu'ils seront disponibles.

---

## Installation

### Prérequis prévus

- Arduino IDE avec le cœur RAKwireless, ou environnement STM32 compatible ;
- KiCad pour consulter ou modifier les fichiers hardware ;
- outil de terminal série ;
- compte réseau LoRaWAN ;
- chaîne d'alimentation solaire adaptée ;
- chargeur ou alimentation pour les essais.

### Firmware

Le firmware n'est pas encore ajouté au dépôt. Une fois disponible, cette section devra fournir :

```text
git clone https://github.com/Masse690/Projet_Disciplinaire.git
cd Projet_Disciplinaire
```

Puis les instructions exactes d'installation des dépendances, de compilation et de téléversement.

---

## Critères d'évaluation

| Critère | Pondération |
|---|---:|
| Technique : complexité du système | 30 % |
| Applicabilité : maturité et utilité du démonstrateur | 25 % |
| Présentation : qualité, clarté et réponses aux questions | 25 % |
| Consommation : optimisation énergétique | 20 % |

---

## Planning

| Période | Travaux |
|---|---|
| Phase actuelle | Analyse du sujet, composants et architecture |
| À venir | Sélection finale du cas d'application et du capteur |
| À venir | Modélisation et conception de l'antenne |
| À venir | Développement du firmware |
| À venir | Conception PCB sous KiCad |
| Décembre 2026 | Finalisation de la carte électronique |
| Janvier 2027 | Fabrication de la carte |
| À venir | Assemblage avec pochoir et four à fusion |
| À venir | Tests électriques, LoRaWAN et autonomie |
| À venir | Déploiement sur une à deux semaines |

---

## Ressources

- [Sujet du projet](Projet%20SMART%20CAMPUS_27.pdf)
- [Datasheet RAK3172](Data_sheet/RAK3172.pdf)
- [Datasheet bq25570](Data_sheet/bq25570.pdf)
- [Datasheet MQ-2](Data_sheet/605-00008-MQ-2-Datasheet-370464.pdf)
- [Documentation RAK3172](https://docs.rakwireless.com/product-categories/wisduo/rak3172-module/overview/)
- [Documentation Texas Instruments bq25570](https://www.ti.com/product/BQ25570)
- [Exemple de plateforme EDU RAK 2026](https://github.com/DorianDIANA/EDU_RAK_2026)

---

## Démonstration

Des images, vidéos, résultats de portée, graphiques de consommation et données du réseau seront ajoutés après les premiers essais.

## Licence

Licence à définir avant la diffusion publique du projet.
