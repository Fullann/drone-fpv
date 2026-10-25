# Nazgul DC5 6S — Build FPV Freestyle & Cinématique HD

Conception, assemblage et configuration logicielle intégrale d'un drone FPV 5 pouces en géométrie Deadcat, optimisé pour le freestyle fluide et la prise de vue cinématique HD via le système DJI O4 Pro.

| Drone assemblé & radiocommande Taranis Q X7 | Vue sur banc d'atelier |
| :---: | :---: |
| ![Nazgul DC5 O4 Pro assemblé](docs/images/nazgul_dc5_main.jpg) | ![Nazgul DC5 sur étagère](docs/images/nazgul_dc5_shelf.jpg) |

---

## Sommaire

- [Pourquoi ce build ?](#pourquoi-ce-build-)
- [Composants utilisés](#composants-utilisés)
- [Câblage & Schéma électronique](#câblage--schéma-électronique)
- [Radiocommande : EdgeTX & ExpressLRS](#radiocommande--edgetx--expresslrs)
- [Configuration sous Betaflight 4.5](#configuration-sous-betaflight-45)
- [Sécurité & GPS Rescue](#sécurité--gps-rescue)
- [Checklist de mise en route](#checklist-de-mise-en-route)
- [Bilan & Prochaines étapes](#bilan--prochaines-étapes)

---

## Pourquoi ce build ?

Plutôt que d'acheter un modèle prêt à voler (BNF), je voulais assembler chaque partie de la machine pour la comprendre à 100% et pouvoir la réparer facilement après un crash.

### Choix de la géométrie et de la motorisation
Pour le châssis, la géométrie **Deadcat** du Nazgul DC5 s'imposait : les deux bras avant sont décalés vers l'arrière, ce qui dégage totalement le champ de vision de la caméra. Aucun bout d'hélice n'apparaît dans les plans vidéo 4K.

Côté propulsion, le combo moteurs **2207 1800KV** et batterie **LiPo 6S 1300mAh** apporte une réponse franche dès le bas des gaz, sans chute brutale de tension (*voltage sag*), avec assez de réserve de puissance pour rattraper un piqué serré en fin de figure.

### Électronique de bord
La stack **SpeedyBee F405 V5** avec son ESC 55A encaisse sans broncher les pics d'intensité. Son gros avantage pratique reste le module Bluetooth intégré : en plein champ, deux minutes sur le smartphone suffisent pour modifier un paramètre, réajuster des rates ou vérifier les capteurs sans avoir à sortir un ordinateur portable.

### Liaison vidéo et contrôle radio
- **Vidéo :** Le système DJI O4 Pro relié aux Goggles 3 offre une dynamique et un piqué d'image qui changent tout par rapport à l'analogique. On repère les branches fines et les obstacles bien plus tôt, ce qui permet d'engager les trajectoires avec beaucoup plus de confiance.
- **Radio :** J'utilise ma FrSky Taranis Q X7 passée sous EdgeTX et équipée d'un module d'émission RadioMaster Ranger Micro en ExpressLRS (2.4 GHz). Le signal reste verrouillé même derrière des arbres ou des structures en béton, avec une latence quasi imperceptible au stick.
- **Sécurité :** Sur une machine de cette valeur, un crash sur perte de signal radio n'est pas envisageable. J'ai donc intégré un GPS HGLRC M100-5883 (puce M10 + boussole) pour gérer un retour automatique fiable (*GPS Rescue*).

---

## Composants utilisés

| Rôle | Composant | Rôle & Justification |
| :--- | :--- | :--- |
| **Châssis** | iFlight Nazgul DC5 V1.1 O4 Pro | Frame carbone 3K Deadcat, bras de 5 mm rigides, champ de caméra dégagé. |
| **Contrôleur (FC)** | SpeedyBee F405 V5 | MCU STM32F405, gyro ICM42688P, baromètre DPS310, Bluetooth/Wi-Fi BLE intégré. |
| **ESC** | SpeedyBee BLS 55A 4-in-1 | 55A continus (pics 65A), compatible DShot300/600 et firmware Bluejay. |
| **Moteurs** | Brushless 2207 1800KV | Bobinage orienté couple pour batterie 6S (25.2V max). |
| **Hélices** | Tripales 5" (5140 / 51433) | Accroche directe en virage et réactivité sans rebond en sortie de plongeon. |
| **Batterie** | LiPo 6S 1300mAh (120C+) | Décharge instantanée élevée pour absorber les gros appels de courant. |
| **Filtrage** | Condensateur Low-ESR 35V 1000µF | Absorbe les pics de tension inductifs générés par le freinage moteur DShot. |
| **Système Vidéo** | DJI O4 Pro Air Unit + Goggles 3 | Flux numérique HD 1080p/4K à faible latence (< 20 ms). |
| **Récepteur (RX)** | RadioMaster Nano ELRS 2.4GHz | Liaison CRSF ultra rapide, antenne T-dipole montée à l'arrière. |
| **Radiocommande** | FrSky Taranis Q X7 + Ranger Micro | Radio sous EdgeTX avec module externe ELRS 2.4 GHz (jusqu'à 1W). |
| **GPS / Boussole** | HGLRC M100-5883 M10 | Fix rapide (15+ satellites) et boussole QMC5883L pour le mode Rescue. |
| **Prints TPU** | Protections vert lime & support cam | Bumpers moteur, patins, support antenne arrière et support incliné caméra d'action. |

---

## Câblage & Schéma électronique

### Architecture des connexions

```
                      +-----------------------------+
                      |   LiPo 6S 1300mAh (XT60)    |
                      +-----------------------------+
                                     |
                         [Condensateur 35V 1000µF]
                                     |
                      +-----------------------------+
                      |   SpeedyBee BLS 55A ESC     |
                      +-----------------------------+
                        |     |        |     |
                       M1    M2       M3    M4
                        \     \        /     /
                      +-----------------------------+
                      |  Nappe 8 pins (VBAT, M1-4)  |
                      +-----------------------------+
                                     |
                                     v
                      +-----------------------------+
                      |  SpeedyBee F405 V5 FC       |
                      +-----------------------------+
                        |            |            |
            [UART2 / 4.5V]    [UART3 / 9V]   [UART4 + I2C / 5V]
                 |                   |                 |
                 v                   v                 v
           RadioMaster Nano      DJI O4 Pro       HGLRC M100-5883
             ELRS (2.4G)          Air Unit          GPS + Compass
```

### Schéma des broches de la SpeedyBee F405 V5

```text
=============================================================================
              AFFECTATION DES PADS : SPEEDYBEE F405 V5
=============================================================================

1. ALIMENTATION ESC & MOTEURS
   - V+ / GND (ESC)  <--- Câble XT60 12AWG + Condensateur 35V 1000µF en parallèle
   - Phases moteurs   <--- 3 fils par bras soudés sur les pads M1, M2, M3, M4
   - Nappe centrale   <--- Connecte l'ESC à la FC (alimentation, signaux, télémétrie)

2. DJI O4 PRO AIR UNIT (sur BEC 9V dédié)
   - 9V (BEC VTX)     <--- Fil Rouge (Alimentation VTX)
   - GND              <--- Fil Noir  (Masse)
   - R3 (RX3)         <--- Fil Blanc (TX du DJI O4)
   - T3 (TX3)         <--- Fil Gris  (RX du DJI O4)

3. RÉCEPTEUR RADIOMASTER NANO ELRS (UART2)
   - 4.5V / 5V        <--- Fil Rouge (alimenté même par câble USB sur établi)
   - GND              <--- Fil Noir  (Masse)
   - R2 (RX2)         <--- Fil Jaune (TX du récepteur)
   - T2 (TX2)         <--- Fil Blanc (RX du récepteur)

4. GPS & BOUSSOLE HGLRC M100-5883 (UART4 + Bus I2C)
   - 5V / 4.5V        <--- Fil Rouge (Alimentation 5V)
   - GND              <--- Fil Noir  (Masse)
   - R4 (RX4)         <--- Fil Vert  (TX GPS)
   - T4 (TX4)         <--- Fil Jaune (RX GPS)
   - SDA              <--- Fil Blanc (SDA Magnétomètre)
   - SCL              <--- Fil Bleu  (SCL Magnétomètre)

5. PORTS SPÉCIFIQUES
   - UART1            <--- Réservé au Bluetooth interne (ne rien souder dessus)
   - UART5            <--- Télémétrie ESC (câblée d'origine via la nappe)
=============================================================================
```

> **Note sur le câblage série :** Penser à croiser TX et RX (`TX périph` vers `RX carte`, et `RX périph` vers `TX carte`). Avant la première mise sous tension avec la LiPo, vérifier systématiquement l'absence de court-circuit au multimètre (test de continuité entre les bornes du XT60) et brancher avec un **SmokeStopper**.

---

## Radiocommande : EdgeTX & ExpressLRS

Pour faire communiquer la FrSky Taranis Q X7 avec le module RadioMaster Ranger Micro, il est nécessaire de mettre à jour le firmware de la radio et d'harmoniser le protocole ExpressLRS.

### 1. Flash de la Taranis Q X7 vers EdgeTX
OpenTX ne recevant plus de mises à jour majeures, le passage à **EdgeTX** permet de gérer nativement les scripts LUA récents et les hauts débits série CRSF :
1. Allumer la radio en mode bootloader (pincer les trims horizontaux vers le centre à l'allumage).
2. Connecter la radio en USB et utiliser l'outil en ligne [EdgeTX Buddy](https://buddy.edgetx.org/).
3. Flasher la version stable pour la cible `FrSky Taranis Q X7`.
4. Remplacer les dossiers de la carte MicroSD par le pack SD EdgeTX correspondant.

> **Astuce technique (Baudrate Q X7) :**  
> Les premières générations de Taranis Q X7 possèdent un inverseur matériel sur la baie JR qui limite parfois la vitesse série. Dans les réglages du modèle sous EdgeTX (`MDL` -> `Configuration RF externe`), régler `Max Bauds` sur **400 000**. Si des alertes de télémétrie intermittentes apparaissent en vol, basculer à **115 200** bauds.

### 2. Flash du module d'émission (Ranger Micro TX)
1. Ouvrir **ExpressLRS Configurator**.
2. Choisir la cible : `RadioMaster 2.4 GHz` -> `RadioMaster Ranger Micro 2400 TX`.
3. Méthode : Via USB-C ou Wi-Fi.
4. Renseigner une **Binding Phrase** personnalisée (ex: `mon-code-drone-2026`).
5. Définir le domaine réglementaire sur `EU_CE_2400` (ou `ISM_2400`) puis lancer `Build & Flash`.

### 3. Flash du récepteur (Nano ELRS RX)
1. Allumer le drone (sous USB ou via batterie avec SmokeStopper).
2. Patienter 30 secondes sans allumer la radio : la LED du récepteur clignote rapidement, signalant l'activation du point d'accès Wi-Fi de secours.
3. Se connecter au réseau Wi-Fi `ExpressLRS RX` (mot de passe : `expresslrs`).
4. Dans le navigateur (`http://10.0.0.1`), téléverser le firmware compilé avec la **même Binding Phrase** que l'émetteur. Dès le redémarrage, la liaison est instantanée et permanente.

### 4. Paramétrage du modèle & script LUA
- Sur la Taranis, désactiver le module interne (`Internal RF = OFF`) et activer le module externe en mode `CRSF`.
- Copier le script `elrsV3.lua` dans `/SCRIPTS/TOOLS/` sur la carte SD.
- Via le menu LUA sur la radio, définir :
  - **Packet Rate :** `250Hz` (très bon compromis fluidité / portée).
  - **Dynamic Power :** Activé (ajuste automatiquement la puissance jusqu'à 250mW ou 1W selon le signal reçu).

---

## Configuration sous Betaflight 4.5

### 1. Mapping des ports UART (Onglet Ports)
- **UART1 :** MSP activé par défaut (Bluetooth SpeedyBee). Ne pas modifier.
- **UART2 :** Cocher `Serial RX` (liaison radio ELRS).
- **UART3 :** Colonne Périphériques -> Sélectionner `VTX (MSP + DisplayPort)` pour le retour OSD HD dans le casque DJI.
- **UART4 :** Colonne Capteurs -> Activer `GPS` (115200 bauds).

### 2. Configuration du récepteur
- Type de récepteur : `Récepteur Série (Serial-based receiver)`.
- Fournisseur : `CRSF`.
- Ordre des voies : `AETR1234`.
- Affectation des switchs :
  - `AUX1` : Armement (interrupteur 2 positions).
  - `AUX2` : Modes de vol (Acro / Angle).
  - `AUX3` : Beeper d'urgence.
  - `AUX4` : Déclenchement manuel du GPS Rescue.
  - `AUX5` : Mode Tortue (*Flip Over After Crash*).

### 3. Moteurs, ESC & Filtrage RPM
- **Sens de rotation :** Configuré en **Props Out** (hélices tournant vers l'extérieur). Sur un châssis Deadcat, cela évite de projeter débris et herbe sur l'optique de la caméra lors des posés en terrain meuble.
- **Protocole :** `DSHOT600`.
- **DShot Bidirectionnel :** Activé avec `14` pôles magnétiques pour les moteurs 2207.
- **ESC Bluejay :** Flasher les 4 ESC en firmware Bluejay (version 0.19+) via [ESC-Configurator](https://esc-configurator.com/) pour activer la remontée télémétrique exacte des régimes moteurs et faire fonctionner les filtres Notch dynamiques de Betaflight.

### 4. OSD HD pour DJI Goggles 3
Dans l'onglet OSD, sélectionner le canvas HD (53x20 caractères).  
Puis dans le CLI :
```text
set osd_displayport_device = MSP
set displayport_msp_serial = 2
save
```
*(L'indice série 2 correspond à l'UART3 sous Betaflight).*

Éléments affichés dans le masque : tension moyenne par cellule (seuil d'alerte à 3.5V), consommation en mAh, nombre de satellites, flèche et distance du point de retour, indicateur Link Quality (LQ) et niveau RSSI en dBm.

---

## Sécurité & GPS Rescue

Le mode GPS Rescue est paramétré pour ramener automatiquement la machine en cas de perte de liaison radio (*failsafe*) ou par activation manuelle sur le switch `AUX4`.

```
                    +------------------------------------+
                    |  Perte radio (Failsafe) ou Switch  |
                    +------------------------------------+
                                      |
                         Le drone est-il à > 20m ?
                                      |
                      +---------------+---------------+
                      | OUI                           | NON
                      v                               v
           Montée à 50 m d'altitude            Désarmement immédiat
                      |
           Alignement vers le point Home
                      |
           Retour en vol rectiligne (54 km/h)
                      |
           Arrivée au-dessus du décollage
                      |
           Descente douce (1.5 m/s)
                      |
           Reprise manuelle au stick dès réception
```

### Configuration CLI Betaflight :
```text
# Protocole GPS & Boussole
set gps_provider = UBLOX
set gps_auto_baud = ON
set gps_ubx_use_galileo = ON
set mag_hardware = QMC5883

# Paramètres GPS Rescue
set gps_rescue_min_sats = 8
set gps_rescue_return_alt = 50
set gps_rescue_ground_speed = 1500
set gps_rescue_descend_rate = 150
set gps_rescue_initial_climb = 15
set gps_rescue_sanity_checks = RESCUE_SANITY_ON
set gps_rescue_allow_arming_without_fix = OFF

# Déclenchement automatique en Failsafe
set failsafe_procedure = GPS-RESCUE
save
```

---

## Checklist de mise en route

1. **Vérification électrique :** Pas de continuité au multimètre entre le `+` et le `-` du XT60. Premier allumage systématique avec un **SmokeStopper**.
2. **Sens des moteurs (SANS HÉLICES) :** Vérifier dans l'onglet Moteurs que les moteurs tournent dans le sens Props Out configuré.
3. **Réponse gyroscopique :** Incliner le drone à la main et vérifier que la représentation 3D dans le configurateur suit fidèlement le mouvement.
4. **Test Failsafe sur table :** Armer à vide (sans hélices), éteindre la radiocommande et observer l'activation immédiate du mode Failsafe / GPS Rescue.
5. **Acquisition satellites :** Premier démarrage GPS en extérieur dégagé. Attendre 8 à 10 satellites minimum pour enregistrer le point Home avant le premier décollage.
6. **Vol de validation :** Décollage en stationnaire à 2 mètres, contrôle de la chauffe moteur après 30 secondes, puis test du switch Rescue à 50 mètres de distance.

---

## Bilan & Prochaines étapes

Ce build répond exactement à mon cahier des charges :
- L'image HD numérique DJI O4 est limpide, sans vibration ni hélice parasite dans le cadre.
- La réactivité des moteurs 1800KV en 6S offre du répondant dès le premier tiers des gaz, parfait pour rattraper des manœuvres engagées.
- Le combo EdgeTX + ExpressLRS assure une liaison radio solide et transparente.
- Le retour automatique GPS Rescue apporte la sécurité nécessaire pour voler sereinement au-dessus de zones accidentées.

**Prochaine étape :** Réutiliser cette chaîne de compétences (électronique de vol, protocoles radio longue portée et paramétrage autonome) pour concevoir et faire voler une **aile volante FPV bimoteur imprimée en 3D** sous iNav.