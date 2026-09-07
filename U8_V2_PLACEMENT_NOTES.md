# Placement du buck repris de la v2

Date : 2026-09-07.

Carte modifiée : `Bucket_universel_v2.kicad_pcb` dans le dossier du rework v3.
Source : `/home/juliann/Desktop/Bucket_universel_v2/Bucket_universel_v2.kicad_pcb` (inchangée).
Sauvegarde avant cette intervention : `Bucket_universel_v2_before_v2_buck_placement.kicad_pcb`.

Cette disposition remplace celle décrite dans `U8_PLACEMENT_NOTES.md` du 3 septembre.

## Placement

Les douze composants du buck conservent exactement leurs orientations et leurs positions
relatives de la v2. Une translation de (+83,2 ; +38,2) mm adapte le groupe au rework.

| Référence | X (mm) | Y (mm) | Orientation KiCad |
|---|---:|---:|---:|
| U8 | 197,9 | 84,5 | 90° |
| L1 | 192,0 | 80,0 | 180° |
| C30 | 183,6 | 83,1 | 180° |
| C31 | 207,8 | 84,7 | 0° |
| C32 | 194,8 | 89,1 | -90° |
| C33 | 193,1 | 84,3 | 180° |
| C34 | 193,3 | 87,5 | -90° |
| R21 | 199,925 | 91,9 | 180° |
| R22 | 196,825 | 91,9 | 180° |
| R23 | 197,4 | 88,4 | 90° |
| R24 | 199,2 | 88,4 | -90° |
| R25 | 201,5 | 87,6 | 180° |

Pour libérer l'espace, C10 est descendu de 1,3 mm à (207,95 ; 92,5) mm et C7
est tourné de 180° à -90° autour de son centre inchangé (212,95 ; 84,7125) mm.
Leurs pistes sont adaptées. Tous les autres composants restent en place.

## Raccordements

- Routage des signaux du buck repris de la v2 et raccordements LX, +12 V, +5 V,
  VCC et GND adaptés aux empreintes corrigées du rework.
- Portions de pistes +3V3 déportées sur B.Cu pour contourner le groupe.
- Repères C31, C7 et C10 repositionnés.
- Empreintes, valeurs, numéros de pastilles, nets, attributs et contour conservés.
- Schémas et configuration du projet inchangés. Zones de cuivre remplies.

## Vérification

Contrôle des positions et orientations des douze composants par comparaison avec la source.
Contrôle des identifiants, nets et attributs des empreintes et pastilles, du contour et des
composants extérieurs au groupe. Aucun chevauchement de courtyards ni nouvelle erreur
électrique ou mécanique au DRC.

DRC KiCad 10.0.5, avec remplissage des zones et configuration du projet :

| Signalement | Avant | Après |
|---|---:|---:|
| Connexions non routées | 0 | 0 |
| Courts-circuits | 2 | 2 |
| Pont de masque | 1 | 1 |
| Dégagement des perçages | 4 | 4 |
| Sérigraphie sur cuivre | 197 | 196 |
| Sérigraphie près du bord | 3 | 3 |
| Total des signalements | 207 | 206 |

Les deux courts-circuits existent dans la sauvegarde avant intervention et restent hors
de la zone modifiée :

- GND / GPIO6 : piste B.Cu passant sous la pastille 5 de J3, vers (138,94 ; 114,27) mm.
- GND / USB_D− : via GND à (169,5 ; 68,5) mm en conflit avec une piste B.Cu.

Ces deux courts-circuits ont ensuite été corrigés le 2026-09-07, à la demande de
l'utilisateur : voir `SHORT_CIRCUIT_FIX_NOTES.md` et `short_circuit_fix_drc.json`.
Les chiffres ci-dessus décrivent l'état avant cette correction. Les autres signalements
concernent notamment le grand logo de sérigraphie et les perçages de J15.

Rapports : `u8_v2_placement_drc_before.json` et `u8_v2_placement_drc.json`.
Aperçu simplifié : `u8_v2_placement_preview.png` (pistes et pastilles ; plans GND et
sérigraphie masqués, références superposées pour repérage).
