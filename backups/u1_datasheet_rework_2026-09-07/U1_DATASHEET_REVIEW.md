# Vérification de U1 — DRV8825PWPR

Date : 2026-09-07. Carte actuelle, après correction des courts-circuits.
Référence : [datasheet TI DRV8825, SLVSA73F](https://www.ti.com/lit/ds/symlink/drv8825.pdf),
également présente sous `drv8825.pdf`.

**Connexions cohérentes, mais découplage à rapprocher et réglage du courant à nuancer.
Les anciennes notes inversaient STEP et nENBL.** PCB et schémas inchangés pendant cet audit.

## Commandes

Correspondance confirmée dans le symbole, la netlist hiérarchique et le PCB.
Fonctions des broches : datasheet, page 3.

| Fonction | Broche U1 | Net actuel | Broche U5 |
|---|---:|---|---:|
| DIR | 20 | GPIO4 | 7 |
| nENBL, actif bas | 21 | GPIO2 | 4 |
| STEP | 22 | GPIO7 | 10 |

Le firmware doit émettre STEP sur GPIO7 et abaisser GPIO2 pour activer le driver.
Un firmware basé sur l'ancienne affectation documentée GPIO2 = STEP / GPIO7 = ENABLE
ne piloterait pas correctement ce PCB. Aucun firmware n'est présent pour vérifier ce point.
Conserver cette ancienne affectation nécessiterait une permutation dans le schéma ET le routage.

## Découplage

C2/C3 : 100 nF entre +12V et GND. C1 : 100 µF. VMA et VMB sont bien au +12V.
Distances entre centres des pastilles, à vol d'oiseau :

| Liaison | Distance |
|---|---:|
| U1.4 VMA → C2.1 | 7,87 mm |
| U1.4 VMA → C3.1 | 10,29 mm |
| U1.11 VMB → C2.1 | 5,75 mm |
| U1.11 VMB → C3.1 | 5,96 mm |

Ce sont des bornes inférieures aux longueurs routées. TI demande un 100 nF près de chaque
broche VM et un retour GND large (sections 10 et 11.1). Je recommande de déplacer un
condensateur près de la broche 4 et l'autre près de la broche 11, puis de raccourcir leurs boucles.

## Courant et puissance

R5 = 56 kΩ, R6 = 10 kΩ ; R3/R4 = 0,1 Ω, marquées « 0.2W 1% ».
AVREF et BVREF partagent le même diviseur.

- VREF nominal = 3,3 × 10 / (56 + 10) = 0,5 V.
- IFS nominal = VREF / (5 × RSENSE) = 1 A, seuil pleine échelle.
- Entre 0 et 1 V de VREF, TI indique une précision dégradée (section 7.3).
- À 1 A, I²R = 0,1 W par shunt. La référence fabricant et son déclassement thermique restent inconnus.

Le réglage actuel est donc fonctionnel en principe, sans garantir un courant précis de 1 A.

| Net | Largeur | Longueur cumulée | Vias |
|---|---:|---:|---:|
| OA1 | 0,2 mm | 21,46 mm | 1 |
| OA2 | 0,4 mm | 21,24 mm | 3 |
| OB1 | 0,2 mm | 22,77 mm | 2 |
| OB2 | 0,2 mm | 17,08 mm | 0 |
| ISENA → R3 | 0,2 mm | 2,82 mm | 0 |
| ISENB → R4 | 0,2 mm | 2,33 mm | 0 |

Je recommande d'élargir les phases et les chemins de courant des shunts dès la sortie des
pastilles. Le courant admissible dépend aussi du cuivre, du refroidissement et du moteur.
Ces seules mesures ne valident pas un fonctionnement à 2,5 A.

## Autres contrôles

| Élément observé | Résultat |
|---|---|
| C5 = 10 nF entre CP1 et CP2 | Valeur/connexion cohérentes ; vérifier la spécification 50 V |
| C4 = 100 nF et R2 = 1 MΩ entre VCP et +12V | Valeurs/connexions cohérentes |
| C6 = 470 nF entre V3P3OUT et GND | Cohérent ; V3P3OUT n'alimente aucun autre circuit |
| nRESET et nSLEEP | Reliés au +3V3 |
| MODE2/MODE1/MODE0 = 0/1/0 | Quart de pas fixe |
| DECAY flottante ; NC flottante | Mixed decay ; NC correctement libre |
| nFAULT et nHOME | Rappels de 10 kΩ présents ; pas de liaison au microcontrôleur |
| J4 | Bobine A sur 1–2 ; bobine B sur 3–4 |
| Masse/thermique | Broches 14, 28 et pastille 29 à GND ; quatre vias 0,5/0,25 mm sous U1 |

Les tensions nominales, diélectriques et références d'achat de C1 à C6 ne sont pas renseignés.
Leur aptitude réelle et la température en charge restent à confirmer.

## Empreinte

Pas mesuré : environ 0,65 mm ; pastille centrale : 3,10 × 5,18 mm.
Les autres pastilles mesurent 1,27 × 0,3048 mm ; les centres des rangées sont espacés de 5,9436 mm.
L'exemple TI PWP0028C propose 1,5 × 0,45 mm et un espacement de 5,8 mm.
L'empreinte locale n'en est donc pas une reproduction, malgré son nom « verified ».
Cela ne prouve pas qu'elle est inutilisable ; je recommande une validation d'assemblage
ou une reprise de l'empreinte TI appropriée avant fabrication.

## Vérifications KiCad

DRC avec remplissage des zones : aucun court-circuit, aucune connexion non routée.
203 signalements existants de sérigraphie/perçages. Parité schéma–PCB : 7 signalements
existants, aucun sur U1. Nets de toutes les pastilles U1 comparés explicitement à la netlist.

Fichiers : `u1_datasheet_review_drc.json` et `u1_datasheet_review_layout.png`
(aperçu simplifié sans plans GND). Le DRC ne valide pas le comportement transitoire ou thermique.
