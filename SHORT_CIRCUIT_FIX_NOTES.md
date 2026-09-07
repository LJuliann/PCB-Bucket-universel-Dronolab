# Correction des courts-circuits GPIO6 et USB D−

Date : 2026-09-07.
Carte : `Bucket_universel_v2.kicad_pcb`.
Sauvegarde : `Bucket_universel_v2_before_short_circuit_fix.kicad_pcb`.

- GPIO6 : la piste GND de 0,2 mm sur B.Cu, entre (133,85 ; 114,15) et
  (143,2 ; 114,15) mm, contournait insuffisamment la pastille 5 de J3.
  Elle est remplacée par cinq segments passant sous cette pastille, à Y = 115,5 mm.
  Cela supprime également le pont de masque signalé au même endroit.
- USB D− : le via GND de 0,6/0,3 mm à (169,5 ; 68,5) mm est déplacé
  à (169,5 ; 68,0) mm. Les pistes USB restent identiques.

Les empreintes, leurs positions, les nets et toutes les autres pistes sont conservés.
Le placement du buck repris de la v2 est inchangé. Les zones de cuivre sont remplies.

DRC KiCad 10.0.5 sur le fichier final, avec la configuration du projet :

| Signalement | Avant | Après |
|---|---:|---:|
| Courts-circuits | 2 | 0 |
| Ponts de masque | 1 | 0 |
| Connexions non routées | 0 | 0 |
| Autres signalements | 203 | 203 |

Aucune nouvelle erreur de dégagement ou autre régression DRC. Les 203 signalements
restants sont les 196 sérigraphies sur cuivre, les 3 sérigraphies près du bord et
les 4 conflits de dégagement des perçages de J15 déjà présents avant cette correction.

Rapports : `short_circuit_fix_drc_before.json` et `short_circuit_fix_drc.json`.
