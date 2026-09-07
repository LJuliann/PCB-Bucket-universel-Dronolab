# Rework U1 appliqué — 2026-09-07

**Mise à jour ultérieure :** les signalements DRC et les 7 écarts de parité cités
ci-dessous ont été corrigés. Voir [DRC_CLEANUP_APPLIED.md](DRC_CLEANUP_APPLIED.md)
pour le dernier état : DRC et parité schéma–PCB à zéro.

PCB modifié : `Bucket_universel_v2.kicad_pcb`, dans le dossier du rework v3.
Schéma moteur synchronisé : `untitled.kicad_sch`.
Sauvegarde avant modification : `backups/u1_datasheet_rework_2026-09-07/`.

## Modifications

- U1 utilise la nouvelle empreinte locale
  `DRV8825_local:PWP28_TI_1p5x0p45_EP3p1x5p18` : pastilles de signal
  1,5 × 0,45 mm, pas 0,65 mm, rangées espacées de 5,8 mm,
  pastille thermique rectangulaire 3,1 × 5,18 mm. Dimensions de cuivre
  reprises de l'exemple TI PWP0028C. Ancienne empreinte conservée en bibliothèque.
- C2 et C3, 100 nF, rapprochés respectivement de VMA et VMB :
  distances entre centres de pastilles de 2,265 et 2,198 mm.
  Liaisons locales de 0,3 mm, alimentation commune principalement de 0,6 mm,
  retours GND de 0,6 mm vers vias. Connexion directe de C3.2 au plan GND.
- R3/R4 passent de 0,1 Ω en 0603 à **0,22 Ω, 0,5 W, 1 % en 0805**.
  Chemins ISENA/ISENB élargis à 0,3 puis 0,5 mm.
- R5 passe de 56 kΩ à **20 kΩ** ; R6 reste **10 kΩ**.
  VREF nominal = 3,3 × 10 / (20 + 10) = **1,1 V**.
  IFS nominal = 1,1 / (5 × 0,22) = **1 A**.
  Dissipation maximale nominale d'un shunt à 1 A : 0,22 W.
- R2/C4 et le diviseur déplacés pour permettre ce placement ; routage adapté.
  Parcours moteur conservés et élargis là où les espacements le permettent.
  Anciennes branches devenues inutiles supprimées ; repères locaux repositionnés.

U1 et J4 conservent leurs positions. Les quatre vias thermiques sous U1 restent
présents. Les identifiants des pastilles et leurs nets sont conservés.
Le placement du buck U8 et les deux corrections de courts-circuits précédentes
ne sont pas modifiés. Les règles du projet ne sont pas assouplies.

## Composants à monter

| Référence | Spécification |
|---|---|
| R3, R4 | Panasonic **ERJ6DQFR22V**, 0,22 Ω, 0805, 0,5 W, 1 %, ou équivalent réellement qualifié |
| R5 | 20 kΩ, 0603 ; choisir une tolérance de 1 % |
| R6 | 10 kΩ, 0603 ; choisir une tolérance de 1 % |
| C2, C3 | 100 nF céramique, tension nominale adaptée au +12 V et à ses transitoires |
| C4 | 100 nF céramique, au moins 16 V entre VCP et VM, selon TI |
| C5 | 10 nF, au moins 50 V, selon TI |

La référence Panasonic est donnée par sa [fiche fabricant](https://industrial.panasonic.com/sa/products/pt/current-sensing-chip-resistors/models/ERJ6DQFR22V).
Ne pas monter les anciens shunts de 0,1 Ω avec le nouveau diviseur : le seuil
nominal deviendrait 2,2 A. Les deux changements forment un ensemble.

## Commandes conservées

| Fonction | U1 | Net |
|---|---:|---|
| DIR | 20 | GPIO4 |
| nENBL, actif bas | 21 | GPIO2 |
| STEP | 22 | GPIO7 |

Quart de pas fixe, DECAY flottante, RESET/SLEEP au +3V3 : connexions inchangées.
Bobine A sur J4.1–J4.2 ; bobine B sur J4.3–J4.4.

## Vérification finale

DRC KiCad avec remplissage des zones et contrôle de parité schéma–PCB :

- **0 court-circuit, 0 défaut d'espacement cuivre, 0 connexion non routée.**
- Aucun signalement de piste/via pendante, de thermique incomplet,
  de recouvrement des courtyards ou d'empreinte différente de la bibliothèque.
- 187 signalements restent : 180 de sérigraphie sur vernis,
  3 de sérigraphie près du bord et 4 d'espacement des perçages de J15.
  Avant modification : 203 signalements.
- Les 7 écarts de parité préexistants restent : pads supplémentaires de
  U2/U11/U12 et empreintes H5–H8 manquantes sur le PCB par rapport au schéma.
  Aucun écart ajouté sur U1.
- Nets de toutes les pastilles du circuit revu comparés à la netlist exportée.
  Seuls C2/C3/C4/R2/R3/R4/R5/R6 changent de placement ; les identifiants
  de toutes les pastilles et le fichier de règles du projet sont conservés.

Largeurs et longueurs cumulées des phases après reprise :

| Phase | 0,2 mm | 0,3 mm | 0,4 mm | Vias |
|---|---:|---:|---:|---:|
| OA1 | 6,72 mm | 13,39 mm | 1,43 mm | 1 |
| OA2 | — | — | 21,31 mm | 3 |
| OB1 | 15,69 mm | 1,50 mm | 5,65 mm | 2 |
| OB2 | 13,43 mm | — | 3,57 mm | 0 |

Certaines portions restent à 0,2 mm. Ce rework conserve un seuil nominal de 1 A ;
il ne constitue pas une validation thermique à 2,5 A. Les tensions/transitoires,
le découplage en fonctionnement et les températures doivent être mesurés sur
prototype. Les références exactes de C1 à C6 restent à spécifier dans la BOM.
Le DRC n'est donc pas une validation complète pour fabrication.

Rapports : `u1_rework_drc.json`, `u1_rework_verification.json`.
Aperçu simplifié, sans plans GND : `u1_rework_layout.png`.
Référence : [datasheet TI DRV8825](https://www.ti.com/lit/ds/symlink/drv8825.pdf),
sections 7.3, 8.3.2, 10 et 11, et dessin d'implantation PWP0028C.
