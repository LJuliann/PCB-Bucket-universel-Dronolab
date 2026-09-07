# Révision complète du PCB — préoccupations et validations

Date de la révision : 2026-09-03  
Projet : `Bucket_universel_v2`  
Version KiCad utilisée pour les contrôles : 10.0.5

## Portée conservée

- L'agencement général est conservé. C30, C31, C34, R25 et R21 ont été déplacés localement autour de U8; C6 et R8 ont ensuite été légèrement décalés autour de U1 pour permettre le routage de ses commandes.
- Le contour de carte et tous les trous de fixation sont inchangés.
- Aucun connecteur n'a été déplacé, y compris J13 et J15.
- Le placement général et le grand dessin en sérigraphie ont été conservés.
- Les GPIO potentiellement laissés libres volontairement n'ont pas été réunis automatiquement.
- U8, L1 et C33 restent fixes; la zone du buck a reçu une seconde optimisation locale documentée ci-dessous.

## Corrections appliquées sans ambiguïté

1. U2 et U11 (DRV8874) : la pastille thermique 17 est maintenant reliée à GND et quatre vias thermiques ont été ajoutés sous chaque boîtier.
2. U1 (DRV8825) : la pastille thermique comporte maintenant quatre vias GND de 0,5/0,25 mm, répartis régulièrement. Une copie locale vérifiée de l'empreinte PWP28 est associée au schéma et au PCB.
3. U12 (LM7805, boîtier DCY/SOT-223) : la languette/pastille 4 est maintenant reliée à GND. Quatre pistes sans net qui la reliaient aux ancrages mécaniques de J9 ont été supprimées.
4. Entrée 12 V : la liaison J9 → F1 a été remplacée par une piste de 1,0 mm et un via de 0,8/0,4 mm.
5. U8 : l'identifiant d'empreinte a été réparé et l'empreinte 22 pads réellement utilisée a été sauvegardée dans la bibliothèque locale.
6. Les entrées de bibliothèque d'empreintes invalides ou en double ont été retirées. La bibliothèque locale du symbole RP2350 a été ajoutée à `sym-lib-table`.
7. R10, R13, R15 et R29 sont maintenant réellement marquées DNP dans le schéma et sur le PCB. Elles sont absentes de la BOM et du fichier de placement « assemblé ».
8. Bloc U8 : C31 a été rapproché des broches IN, C30 de la sortie de L1, C34 de VCC et R25 de TON. Quatre vias GND et des pistes de puissance de 1,2–1,5 mm ont été ajoutés aux condensateurs d'entrée et de sortie.
9. U1 : STEP, DIR et nENBL sont reliés respectivement à GPIO7, GPIO4 et GPIO2, du RP2350 jusqu'au driver. Rectificatif du 2026-09-07 : STEP/nENBL étaient inversés dans ce compte rendu ; voir `U1_DATASHEET_REVIEW.md`. OA2 a été reroutée en 0,4 mm pour dégager ces signaux.
10. U1 : la limite nominale a été ramenée d'environ 2 A à environ 1 A par phase avec R5 = 56 kΩ, R6 = 10 kΩ et R3/R4 = 0,1 Ω. R3/R4 sont spécifiées `100m 0.2W 1%` dans le schéma.

## Résultats des contrôles

### DRC PCB

- 0 connexion non routée.
- 205 violations restantes :
  - 197 chevauchements sérigraphie/cuivre, surtout causés par le grand dessin blanc;
  - 4 dégagements internes de perçage sur l'empreinte USB-C J15;
  - 4 objets de sérigraphie trop près du bord.
- Les erreurs précédentes de masque autour de U12 et les problèmes associés aux pastilles thermiques corrigées ont disparu.

### ERC schéma

- 468 messages : 53 erreurs et 415 avertissements, soit 15 messages de moins après la correction des commandes de U1.
- Principales catégories : 326 extrémités hors grille, 58 étiquettes sur broches isolées, 30 incohérences de labels hiérarchiques, 26 noms utilisés à la fois en local et global, 13 alimentations considérées non pilotées, 4 entrées non pilotées, 6 conflits de types de broches.
- Une partie vient des types de broches incorrects dans les symboles personnalisés et ne représente pas nécessairement une panne réelle. Les incohérences GPIO ci-dessous doivent néanmoins être vérifiées fonctionnellement.

## À confirmer impérativement avant fabrication

### 1. GPIO volontairement libres ou commandes réellement déconnectées

Le PCB contenait deux familles de nets différentes : `/Microcontroller/GPIOx` côté RP2350 et `GPIOx` côté périphériques. STEP, DIR et ENABLE de U1 ont été corrigés dans le schéma hiérarchique puis reroutés sur le PCB. Les GPIO d'extension peuvent rester séparés si c'est volontaire. Il reste à confirmer le cas de ces commandes embarquées :

| Fonction | Composant | Nets à confirmer |
|---|---|---|
| Moteur DC 1 | U2 | GPIO8, GPIO9 |
| Moteur DC 2 | U11 | GPIO10, GPIO11 |
| Servos | U7, U9, U10 | GPIO12, GPIO14, GPIO16 |

Si ces périphériques doivent être commandés directement par le RP2350, leurs labels hiérarchiques doivent être corrigés dans le schéma, puis le PCB mis à jour et rerouté. Les GPIO seulement exposés sur J1/J2/J3 peuvent rester non utilisés, mais devraient recevoir des marqueurs « no connect » explicites au schéma.

Attention : un DRC à 0 connexion non routée ne détecte pas cette erreur d'intention, car chaque famille de nets est entièrement routée séparément.

### 2. Courant du DRV8825 et résistances de mesure

Avec R5 = 56 kΩ, R6 = 10 kΩ et 3,3 V, VREF vaut 0,5 V. Avec R3/R4 = 0,1 Ω, la formule du DRV8825 donne environ 1 A pleine échelle :

`ICHOP = VREF / (5 × RSENSE) = 0,5 / (5 × 0,1) = 1 A`

À 1 A, chaque résistance de mesure peut dissiper jusqu'à environ 0,1 W. R3/R4 doivent donc être de véritables résistances de mesure 0603 d'au moins 0,2 W, 1 %, avec un coefficient thermique adapté. La carte ne doit pas être réglée au-dessus de 1 A par phase sans redimensionner les shunts, les pistes de sortie et le refroidissement.

La broche DECAY reste flottante : le mode de decay interne par défaut est conservé. Si le moteur présente du bruit, une mauvaise régulation ou des pas manqués, prévoir un réseau DECAY adapté après essais.

### 3. Courant des DRV8874 et largeur des pistes

Avec VREF proche de 3,3 V, RIPROPI = 3,3 kΩ et le facteur typique de 455 µA/A, le seuil calculé est proche de 2,2 A pour U2 et U11. Cette estimation doit être confirmée avec les tolérances et le courant réel des moteurs.

- Les sorties pas-à-pas U1 → J4 sont principalement en 0,2 mm.
- Les sorties moteurs DC U2/U11 → J5/J12 sont en 0,4 mm.
- Plusieurs chemins +5 V vers les servos sont en 0,4 mm.

Ces largeurs peuvent être insuffisantes selon le courant, l'épaisseur de cuivre, l'échauffement admissible et la durée de charge. Aucun reroutage massif n'a été fait sans ces informations.

Les condensateurs volants C7 et C35 valent 47 nF, tandis que le DRV8874 recommande 22 nF X5R/X7R entre CPH et CPL. Vérifier cette déviation.

### 4. Buck 12 V → 5 V autour de U8

Le placement a été resserré et rerouté une seconde fois :

- C31 est maintenant à environ 3,65 mm de la broche IN la plus proche, contre environ 5,82 mm auparavant;
- C30 est maintenant à environ 5,81 mm de la sortie de L1, contre environ 6,69 mm auparavant;
- C34 est maintenant à environ 2,67 mm de VCC, contre environ 3,32 mm auparavant;
- R25 est maintenant à environ 2,41 mm de TON, contre environ 3,65 mm auparavant;
- quatre vias GND supplémentaires réduisent le retour des condensateurs d'entrée et de sortie;
- U8, L1 et C33 n'ont pas été déplacés, car un rapprochement supplémentaire créait des conflits avec PGND et les courtyards existants.

Le placement est maintenant propre au DRC et nettement meilleur, mais les références exactes doivent encore être validées :

- présence d'un condensateur céramique d'entrée local entre IN et PGND; C31 est un 22 µF dans une empreinte électrolytique et ne remplace pas forcément ce découplage haute fréquence;
- courant RMS, ESR et tension nominale de C30/C31;
- C30 = 88 µF correspond bien à la valeur de l'application typique du fabricant; vérifier cependant la référence réelle, l'ESR et le courant d'ondulation;
- le réseau de feedback R23/R24 reste à quelques millimètres de U8;
- boucle chaude IN–LX–L1–COUT et retour PGND à valider avec le courant maximal;
- PGOOD de U8 est volontairement inutilisé, mais n'a pas encore de marqueur « no connect » explicite.

Le fabricant recommande de rapprocher les condensateurs d'entrée de IN/PGND, C34 de VCC/AGND, le diviseur de FB et RTON de leurs broches respectives, et d'éloigner les signaux sensibles du nœud LX.

### 5. Alimentation et protection

- Le RP2350 semble alimenté par VBUS USB via le NCP1117. Vérifier qu'il est volontairement éteint lorsque seul le 12 V est présent.
- Vérifier le risque de retour de courant entre USB, 3,3 V, 5 V et 12 V; aucune stratégie d'ORing n'est clairement documentée.
- L'entrée XT30 comporte F1, mais aucune protection évidente contre l'inversion de polarité ou les transitoires/TVS.
- F1 est seulement nommé `Fuse`; son courant, sa tension, son pouvoir de coupure et sa référence doivent être définis.
- D4, D6 et D7 sont seulement nommées `D`; leurs références, courants et tensions doivent être définis.

### 6. RP2350 et USB

- Plusieurs condensateurs de découplage du RP2350 sont à environ 6–10 mm du circuit. Le guide Raspberry Pi demande un découplage très proche des broches d'alimentation.
- USB D+ mesure environ 49,20 mm et D− environ 45,97 mm, soit environ 3,24 mm d'écart. Les deux passent par deux vias et aucune règle de paire différentielle/impédance n'est définie.
- Aucune protection ESD/TVS USB dédiée n'est visible.
- Sur une carte 2 couches, confirmer le plan de référence continu sous la paire et l'impédance avec le fabricant.

### 7. Références et boîtiers à valider

- C50/C51/C52 : 47 µF en 0603 — vérifier qu'une référence réelle offre encore la capacité nécessaire sous polarisation DC.
- C16/C17/C19/C20 : 4,7 µF en 0402 — vérifier tension, capacité effective et derating.
- C1/C10/C29/C30/C31/C38/C53 : tension, ESR, courant d'ondulation, polarité et température non documentés dans la valeur du schéma.
- U2/U11 ont pour valeur `a`; remplacer par la référence complète DRV8874 avant achat.
- Le symbole personnalisé DRV8874 ne contient pas la pastille thermique 17, corrigée seulement sur le PCB. Le symbole doit être réparé avant une future synchronisation schéma → PCB, sinon la correction pourrait être perdue.
- U1 utilise maintenant `DRV8825_local:PWP28_5P18X3P1_verified`. Conserver cette bibliothèque locale avec le projet lors de tout déplacement ou archivage.

### 8. Connecteurs et mécanique

- J13 est le connecteur SWD 3 broches, placé en haut à gauche; J15 est l'USB-C, placé en haut à droite. Ils ne sont pas interchangeables électriquement. Leur position n'a pas été modifiée : confirmer seulement que cet accès mécanique est bien celui voulu.
- L'empreinte J15 possède quatre dégagements internes de perçage de 0,1944 mm alors que la règle du projet exige 0,25 mm. Valider l'empreinte avec la fiche mécanique du connecteur et les capacités du fabricant avant de réduire une règle.
- Les 202 avertissements de sérigraphie/bord sont encore présents. Ils semblent principalement esthétiques, mais la lisibilité des références et la sérigraphie sur pastilles doivent être vérifiées.

## Fichiers de fabrication

Le dossier `jlcpcb/` et `jlcpcb.zip` datent d'avant cette révision. **Ne pas les utiliser pour fabriquer la carte.** Régénérer Gerbers, perçages, BOM et fichiers de placement seulement après validation des points ci-dessus et un dernier DRC/ERC.

## Sources techniques utilisées

- [Texas Instruments — DRV8825](https://www.ti.com/lit/ds/symlink/drv8825.pdf)
- [Texas Instruments — DRV8874](https://www.ti.com/lit/ds/symlink/drv8874.pdf)
- [Texas Instruments — LM340/LM7805](https://www.ti.com/lit/ds/symlink/lm340.pdf)
- [Raspberry Pi — Hardware design with RP2350](https://datasheets.raspberrypi.com/rp2350/hardware-design-with-rp2350.pdf)
- `AOZ2261AQI-15.pdf`, copie locale de la fiche technique Alpha & Omega Semiconductor fournie avec le projet.

## Suite recommandée

Avant la prochaine passe, fournir :

1. le courant maximal de chaque moteur et de chaque servo;
2. l'épaisseur de cuivre et le fabricant visé;
3. la confirmation des six GPIO de commande encore listés plus haut;
4. la confirmation de l'accès mécanique souhaité pour J13 et J15.

Avec ces réponses, la prochaine révision pourra corriger les labels GPIO, dimensionner les pistes et les protections, finir le placement de puissance et réduire les erreurs ERC/DRC sans deviner l'intention du circuit.
