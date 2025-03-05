# Rapport concis

## Quelles améliorations de performances avez-vous observées après l'ajout d'index ?

Les performances des requetes ont été significativement améliorées après l'ajout d'index. Le nombre de documents examinés a été réduit, ce qui a permis d'accélérer l'exécution des requetes.

## Quels types d'index ont été plus efficaces pour vos requetes ?

Les index uniques et les index partiels ont été efficaces

## Avez-vous identifié des compromis entre performance de lecture et d'écriture ?

Oui, les index uniques peuvent ralentir les opérations d'écriture, mais améliorer les performances de lecture.

## Comment choisiriez-vous les index d'une application de bibliothèque en prod ?

Je choisirais des index sur les champs les plus utilisés dans les requetes, comme le titre, l'auteur, le genre, etc et je privilégierais les index uniques pour les champs qui doivent être uniques, comme l'ISBN.
