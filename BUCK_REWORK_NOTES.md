# Reprise du placement du buck 12 V -> 5 V

Carte modifiee : `Bucket_universel_v2.kicad_pcb`

## Modifications

- U8 n'est plus marque DNP sur le PCB.
- L1 a ete rapprochee des broches LX de U8.
- C33 (bootstrap) a ete place pres de BST/LX et reroute sans via.
- C30 a ete repositionne et relie a L1 par une piste de puissance de 1,2 mm.
- C31 a ete deplace pour liberer la boucle de commutation, puis reconnecte au tronc +12 V.
- Les pistes LX internes a U8 restent fines au niveau du pas des broches, puis s'elargissent vers L1.
- La piste +3V3 qui traversait la nouvelle zone de L1 a ete reportee sur B.Cu.

## Validation

- DRC KiCad : 0 connexion non routee.
- Aucun nouveau court-circuit, conflit de degagement ou chevauchement de courtyards.
- Le projet d'origine possede encore de nombreuses alertes preexistantes (pistes de 0,15 mm,
  grand logo de sérigraphie, perçages et empreintes non synchronisees). Elles ne proviennent pas
  de cette reprise du buck et doivent etre traitees avant fabrication.

Le fichier `Bucket_universel_v2_before_buck_rework.kicad_pcb` conserve la carte avant modification.
