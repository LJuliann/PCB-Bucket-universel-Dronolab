# Correction des signalements PCB — 2026-09-07

Les 187 signalements DRC et les 7 différences schéma–PCB du dernier compte rendu
sont corrigés dans le projet principal. Vérification finale KiCad 10.0.5 avec
remplissage des zones : **0 violation, 0 connexion non routée, 0 écart de parité**.
Aucune règle ni sévérité de contrôle n'a été réduite ; aucune exclusion ajoutée.

Sauvegarde avant cette intervention : `backups/drc_cleanup_2026-09-07/`.
Cette sauvegarde contient le rework U1 terminé et les corrections antérieures du buck.

## J15 — USB-C GCT USB4105

Les trous NPTH de 0,65 mm et leur implantation correspondent au
[dessin GCT USB4105](https://gct.co/files/drawings/usb4105.pdf).
Ils sont conservés, ainsi que les ancrages et la position du connecteur.

Une adaptation locale du cuivre est enregistrée dans
`Rework_local.pretty/USB4105_GND_heel_relief.kicad_mod` et affectée au PCB et au schéma :

- Deux pastilles physiques de masse, portant les numéros superposés
  A1/B12 et A12/B1, raccourcies de 0,12 mm uniquement côté trous.
- Largeur conservée à 0,60 mm ; longueur 1,15 → 1,03 mm ; centre décalé
  de 0,06 mm pour conserver l'extrémité opposée au trou.
- Dégagement cuivre–perçage mesuré sur les polygones : **0,3143 mm**, contre
  0,1944 mm auparavant et un minimum projet de 0,25 mm.

Il s'agit d'une adaptation du dessin de pastilles recommandé, pas d'une nouvelle
dimension mécanique du connecteur. Cette empreinte locale doit accompagner les
fichiers de fabrication et être utilisée pour J15 lors des futures mises à jour.

## Sérigraphie

57 repères de composants repositionnés dans des espaces libres. Les repères
concernés utilisent une hauteur de 0,8 mm et un trait de 0,12 mm.
Les repères J7 et C38 sont maintenant à l'intérieur du PCB.
Le logo est conservé en face avant, réduit à environ 13 × 13,3 mm et replacé
dans l'espace libre au-dessus de l'alimentation. Son orientation est conservée.

## Cohérence schéma–PCB

- U2/U11 : ajout d'une broche explicite EP, numéro 17, reliée à GND dans
  le symbole et la feuille moteur. La feuille étant utilisée deux fois,
  les deux instances sont corrigées. Leur valeur « a » est remplacée par
  `DRV8874PWPR`. La [datasheet TI DRV8874](https://www.ti.com/lit/ds/symlink/drv8874.pdf)
  demande de raccorder la pastille thermique à la masse.
- U12 : symbole adapté au boîtier SOT-223 DCY, avec une broche TAB numéro 4
  explicitement raccordée à GND. Valeur `LM7805MP/NOPB`, description et datasheet
  corrigées pour ce composant TI ; l'ancien symbole générique indiquait un L7805 ST
  dans d'autres boîtiers. Voir la [datasheet TI LM7805](https://www.ti.com/lit/ds/symlink/lm340.pdf).
- H5–H8 : retrait des quatre symboles de trous M2.5 sans connexion, présents
  seulement dans la feuille microcontrôleur. Les quatre fixations M3 H1–H4
  effectivement présentes sur le PCB restent inchangées.
- Les symboles corrigés sont conservés dans `Rework_local.kicad_sym`, déclaré
  dans `sym-lib-table`. `fp-lib-table` déclare l'empreinte locale de J15.

Les nets de toutes les anciennes broches du schéma sont inchangés. Les seules
broches ajoutées à la netlist sont U2.17, U11.17 et U12.4, toutes sur GND.
Le routage complet, les vias, les perçages et les positions des composants sont
identiques à la sauvegarde ; seules les quatre entrées de pastilles J15 indiquées
ci-dessus changent de géométrie. Le rework U1 et le placement U8 sont conservés.

## Contrôles et limites restantes

| Contrôle | Avant | Après |
|---|---:|---:|
| DRC PCB | 187 | **0** |
| Connexions non routées | 0 | **0** |
| Différences schéma–PCB | 7 | **0** |
| ERC schéma : erreurs | 53 | 53 |
| ERC schéma : avertissements | 415 | 415 |

Le contrôle ERC complémentaire retrouve exactement les mêmes signalements qu'avant
cette intervention, sans ajout : 326 extrémités hors grille, 58 labels isolés,
30 différences de labels hiérarchiques, 26 noms de labels locaux/globaux communs,
13 alimentations déclarées non pilotées, 6 conflits de types de broches,
4 entrées non pilotées, 4 problèmes de bibliothèque et 1 broche non connectée.
Ils demandent une revue distincte du schéma ; un DRC PCB propre ne les valide pas.
Les essais électriques et thermiques sur prototype restent également nécessaires.

Rapports : `drc_cleanup_final.json`, `drc_cleanup_verification.json`,
`erc_before_drc_cleanup.json`, `erc_after_drc_cleanup.json`.
Déplacements des repères : `drc_cleanup_changes.json`.
Aperçu sérigraphie et ouvertures du vernis : `drc_cleanup_silkscreen.svg` et `.png`.
