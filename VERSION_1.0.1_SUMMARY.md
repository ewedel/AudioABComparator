# Résumé de version 1.0.1

**Date :** 23 août 2026  
**Version technique, affichée et publiée :** 1.0.1  
**Statut :** version corrective stable, Windows et Linux

## Résultat

Audio A/B Comparator affiche désormais les deux formes d’onde sur la même durée commune de lecture lorsque les pistes chargées n’ont pas exactement la même longueur. Cette correction supprime le décalage horizontal artificiel produit auparavant par l’étirement indépendant de chaque forme d’onde.

Les différences temporelles réellement présentes dans les fichiers audio restent visibles et audibles : l’application ne recale, ne compresse et n’étire aucun signal.

## Validation

- compilation Release Linux réussie ;
- suite CTest complète réussie ;
- test de régression avec deux pistes synthétiques de durées différentes ;
- chargement vérifié avec deux stems vocaux réels en PCM 24 bits / 44,1 kHz et PCM 16 bits / 48 kHz.

## Publication

La version 1.0.1 est distribuée sous forme de ZIP portable non signé pour Windows 10/11 x64 et d’AppImage pour Linux x86_64. Les deux artefacts sont accompagnés de leurs empreintes dans `SHA256SUMS`.
