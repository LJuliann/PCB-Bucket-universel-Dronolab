# Optimisation locale du placement U8

Date : 2026-09-03  
Carte : `Bucket_universel_v2.kicad_pcb`

## Modifications

- U8, L1 et C33 sont restés fixes afin de conserver le routage LX/PGND déjà valide.
- C31 a été déplacé de `(199,8; 84,5)` à `(200,25; 88,9)` mm, directement à droite de U8.
- C30 a été déplacé de `(184,0; 77,5)` à `(183,5; 81,3)` mm, dans l'axe de la sortie de L1.
- C34 a été rapproché de VCC.
- R25 a été tourné et rapproché de TON; R21 a été légèrement déplacé pour libérer l'espace.
- Les connexions C31/U8 et L1/C30 ont été refaites en 1,5 mm.
- Quatre vias GND de 0,7/0,35 mm ont été ajoutés près de C30 et C31.
- Le texte de référence C31 a été déplacé hors de U8.

## Amélioration mesurée

| Liaison | Avant | Après |
|---|---:|---:|
| C31 → broche IN U8 la plus proche | 5,82 mm | 3,65 mm |
| C30 → sortie L1 | 6,69 mm | 5,81 mm |
| C34 → VCC U8 | 3,32 mm | 2,67 mm |
| R25 → TON U8 | 3,65 mm | 2,41 mm |

## Validation

- DRC : 206 signalements, 0 connexion non routée.
- Aucun court-circuit, conflit de dégagement, pont de masque, croisement de piste ou chevauchement de courtyard ajouté.
- Signalements restants : 197 sérigraphie/cuivre, 4 perçages internes de J15, 4 sérigraphies près du bord et 1 différence de bibliothèque pour U1.
- Le contour, les trous, tous les connecteurs et tous les GPIO sont inchangés.

## Limite restante

C31 garde une empreinte de condensateur électrolytique. Pour exploiter U8 à fort courant, vérifier la référence, le courant d'ondulation et l'ESR, et envisager un condensateur céramique local approprié entre IN et PGND. Ce choix nécessite une référence de composant et une tension nominale confirmées; il n'a donc pas été modifié automatiquement.
