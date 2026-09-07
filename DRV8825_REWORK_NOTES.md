# Révision du DRV8825 (U1)

**Historique du 2026-09-03.** Les valeurs et l'implantation ont depuis été
modifiées : consulter [U1_REWORK_APPLIED.md](U1_REWORK_APPLIED.md) pour l'état
actuel et les composants à monter.

Date : 2026-09-03  
Projet : `Bucket_universel_v2`

Rectificatif du 2026-09-07 : voir `U1_DATASHEET_REVIEW.md`. Les fonctions STEP et nENBL
étaient inversées dans ce compte rendu ; les lignes ci-dessous reflètent les connexions réelles.

## Modifications appliquées

- Limite de courant ramenée à environ **1 A par phase** : R5 = 56 kΩ, R6 = 10 kΩ, R3/R4 = 0,1 Ω.
- R3 et R4 sont identifiées `100m 0.2W 1%`. Utiliser de vraies résistances de mesure 0603 d'au moins 0,2 W.
- STEP (broche 22) est reliée à GPIO7.
- DIR (broche 20) est reliée à GPIO4.
- nENBL (broche 21) est reliée à GPIO2.
- OA2 a été reroutée en 0,4 mm pour libérer le passage des trois commandes.
- C6 et R8 ont été légèrement déplacés, sans modifier l'agencement général de la carte.
- La pastille thermique de U1 est reliée à GND par quatre vias de 0,5/0,25 mm.
- U1 utilise l'empreinte locale vérifiée `DRV8825_local:PWP28_5P18X3P1_verified`.

## Calcul de courant

Avec 3,3 V, R5 = 56 kΩ et R6 = 10 kΩ :

`VREF = 3,3 × 10 / (56 + 10) = 0,5 V`

Avec RSENSE = 0,1 Ω :

`ICHOP = VREF / (5 × RSENSE) = 0,5 / (5 × 0,1) = 1 A`

Ne pas augmenter le courant en changeant seulement R5. Au-delà de 1 A, il faut aussi revoir R3/R4, la largeur des sorties moteur et la dissipation thermique.

## Vérifications finales

- DRC : 0 connexion non routée, aucun court-circuit et aucune nouvelle erreur de dégagement.
- 205 alertes restantes, toutes héritées : 197 sérigraphie/cuivre, 4 dégagements de perçage sur J15 et 4 sérigraphies près du bord.
- Les 7 alertes de parité schéma–PCB sont identiques à la version précédente et ne concernent pas U1.
- ERC : 468 messages contre 483 avant cette correction; les 15 suppressions correspondent aux commandes de U1 maintenant connectées.

## Points à surveiller aux essais

- DECAY est volontairement laissée flottante pour conserver le mode interne par défaut.
- Les sorties de phase existantes ne sont pas toutes aussi larges que OA2; la limite de 1 A est donc conservatrice et doit rester la valeur de départ.
- Vérifier le courant réel du moteur, la température de U1 et des shunts, puis les pertes de pas sous charge avant toute fabrication en série.
