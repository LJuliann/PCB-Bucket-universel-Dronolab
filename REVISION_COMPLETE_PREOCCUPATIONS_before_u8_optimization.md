# Révision complète du PCB — préoccupations et validations

Date de la révision : 2026-09-03  
Projet : `Bucket_universel_v2`  
Version KiCad utilisée pour les contrôles : 10.0.5

## Portée conservée

- Les 128 empreintes ont conservé exactement leur position, leur rotation et leur face.
- Le contour de carte et tous les trous de fixation sont inchangés.
- Aucun connecteur n'a été déplacé, y compris J13 et J15.
- Le placement général et le grand dessin en sérigraphie ont été conservés.
- Les GPIO potentiellement laissés libres volontairement n'ont pas été réunis automatiquement.
- La reprise précédente du buck autour de U8, L1, C30, C31 et C33 est conservée.

## Corrections appliquées sans ambiguïté

1. U2 et U11 (DRV8874) : la pastille thermique 17 est maintenant reliée à GND et quatre vias thermiques ont été ajoutés sous chaque boîtier.
2. U1 (DRV8825) : deux vias GND supplémentaires ont été ajoutés dans la pastille thermique. Le routage existant sous U1 limite encore la dissipation possible.
3. U12 (LM7805, boîtier DCY/SOT-223) : la languette/pastille 4 est maintenant reliée à GND. Quatre pistes sans net qui la reliaient aux ancrages mécaniques de J9 ont été supprimées.
4. Entrée 12 V : la liaison J9 → F1 a été remplacée par une piste de 1,0 mm et un via de 0,8/0,4 mm.
5. U8 : l'identifiant d'empreinte a été réparé et l'empreinte 22 pads réellement utilisée a été sauvegardée dans la bibliothèque locale.
6. Les entrées de bibliothèque d'empreintes invalides ou en double ont été retirées. La bibliothèque locale du symbole RP2350 a été ajoutée à `sym-lib-table`.
7. R10, R13, R15 et R29 sont maintenant réellement marquées DNP dans le schéma et sur le PCB. Elles sont absentes de la BOM et du fichier de placement « assemblé ».

## Résultats des contrôles

### DRC PCB

- 0 connexion non routée.
- 207 violations restantes :
  - 198 chevauchements sérigraphie/cuivre, surtout causés par le grand dessin blanc;
  - 4 dégagements internes de perçage sur l'empreinte USB-C J15;
  - 4 objets de sérigraphie trop près du bord;
  - 1 différence entre U1 et son empreinte de bibliothèque.
- Les erreurs précédentes de masque autour de U12 et les problèmes associés aux pastilles thermiques corrigées ont disparu.

### ERC schéma

- 483 messages : 59 erreurs et 424 avertissements.
- Principales catégories : 326 extrémités hors grille, 67 étiquettes sur broches isolées, 33 incohérences de labels hiérarchiques, 26 noms utilisés à la fois en local et global, 13 alimentations considérées non pilotées, 7 entrées non pilotées, 6 conflits de types de broches.
- Une partie vient des types de broches incorrects dans les symboles personnalisés et ne représente pas nécessairement une panne réelle. Les incohérences GPIO ci-dessous doivent néanmoins être vérifiées fonctionnellement.

## À confirmer impérativement avant fabrication

### 1. GPIO volontairement libres ou commandes réellement déconnectées

Le PCB contient deux familles de nets différentes : `/Microcontroller/GPIOx` côté RP2350 et `GPIOx` côté périphériques. KiCad les considère comme deux connexions électriques distinctes. Les GPIO d'extension peuvent parfaitement rester séparés si c'est volontaire. En revanche, confirmer le cas de ces commandes embarquées :

| Fonction | Composant | Nets à confirmer |
|---|---|---|
| Driver pas-à-pas | U1 | GPIO2 (STEP), GPIO4 (DIR), GPIO7 (ENABLE) |
| Moteur DC 1 | U2 | GPIO8, GPIO9 |
| Moteur DC 2 | U11 | GPIO10, GPIO11 |
| Servos | U7, U9, U10 | GPIO12, GPIO14, GPIO16 |

Si ces périphériques doivent être commandés directement par le RP2350, leurs labels hiérarchiques doivent être corrigés dans le schéma, puis le PCB mis à jour et rerouté. Les GPIO seulement exposés sur J1/J2/J3 peuvent rester non utilisés, mais devraient recevoir des marqueurs « no connect » explicites au schéma.

Attention : un DRC à 0 connexion non routée ne détecte pas cette erreur d'intention, car chaque famille de nets est entièrement routée séparément.

### 2. Courant du DRV8825 et résistances de mesure

Avec R5 = 23 kΩ, R6 = 10 kΩ et 3,3 V, VREF vaut environ 1,0 V. Avec R3/R4 = 0,1 Ω, la formule du DRV8825 donne environ 2 A pleine échelle :

`ICHOP = VREF / (5 × RSENSE) ≈ 1,0 / (5 × 0,1) ≈ 2 A`

À 2 A, chaque résistance de mesure peut dissiper jusqu'à environ 0,4 W. Une résistance 0603 standard est généralement trop petite pour cette puissance. Il faut confirmer le courant moteur voulu, la puissance nominale et le coefficient thermique exacts de R3/R4, puis adapter VREF, les shunts et le cuivre.

### 3. Courant des DRV8874 et largeur des pistes

Avec VREF proche de 3,3 V, RIPROPI = 3,3 kΩ et le facteur typique de 455 µA/A, le seuil calculé est proche de 2,2 A pour U2 et U11. Cette estimation doit être confirmée avec les tolérances et le courant réel des moteurs.

- Les sorties pas-à-pas U1 → J4 sont principalement en 0,2 mm.
- Les sorties moteurs DC U2/U11 → J5/J12 sont en 0,4 mm.
- Plusieurs chemins +5 V vers les servos sont en 0,4 mm.

Ces largeurs peuvent être insuffisantes selon le courant, l'épaisseur de cuivre, l'échauffement admissible et la durée de charge. Aucun reroutage massif n'a été fait sans ces informations.

Les condensateurs volants C7 et C35 valent 47 nF, tandis que le DRV8874 recommande 22 nF X5R/X7R entre CPH et CPL. Vérifier cette déviation.

### 4. Buck 12 V → 5 V autour de U8

Le placement a été nettement resserré lors de la reprise précédente, mais il reste à valider avec les références exactes :

- présence d'un condensateur céramique d'entrée local entre IN et PGND; C31 est un 22 µF dans une empreinte électrolytique et ne remplace pas forcément ce découplage haute fréquence;
- courant RMS, ESR et tension nominale de C30/C31;
- C30 est indiqué `88u`, valeur possiblement erronée;
- réseau de feedback R23/R24 et RTON R25 encore à quelques millimètres de U8;
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
- U1 diffère de son empreinte de bibliothèque. Ne pas accepter automatiquement une mise à jour d'empreinte sans comparaison visuelle et dimensionnelle.

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
3. la confirmation des neuf GPIO de commande listés plus haut;
4. la confirmation de l'accès mécanique souhaité pour J13 et J15.

Avec ces réponses, la prochaine révision pourra corriger les labels GPIO, dimensionner les pistes et les protections, finir le placement de puissance et réduire les erreurs ERC/DRC sans deviner l'intention du circuit.
