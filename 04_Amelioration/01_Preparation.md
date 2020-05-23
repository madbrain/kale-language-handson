# Langage amélioré

Maintenant que vous disposer d'un éditeur confortable, vous avez pu tester l'édition de fichier Kale
plus intensément et vous avez pu remarquer que dans certaines situations il rapporte bien plus d'erreur
que ce qu'on l'on pourrait imager. Dans l'exemple suivant :
```bash
my_var := 10 +
other_var := 100 * my_var
message := "result " + other_var
```
en l'absence de délimiteur de fin d'affectation tel que `;` typique des langages inspirés du C,
l'analyseur n'a aucun moyen de savoir qu'un identifiant démarre ou pas une affectation et, au lieu de déclarer
une expression invalide, l'erreur déborde sur l'affectation suivante qui se retrouve à son tour en erreur et ainsi de proche en proche.

Inconsciemment en tant que développeur, nous organisons notre code afin de le rendre plus lisible et
nous voyons rapidement que seule la première affectation est invalide, les deux autres prises indépendamment sont parfaitement valides.

## Solution

`step-4.1`

Le langage peut être amélioré en utilisant les informations de layout et distinguer les identifiants qui démarrent une nouvelle ligne des autres.

## Résultat

L'exemple précédent ne doit maintenant comporter qu'une seule erreur. Cet exemple montre que, bien que les deux versions du langage
permettent d'exprimer la même chose, la nouvelle version est beaucoup plus robuste à l'édition et augmente drastiquement
la productivité de ses utilisateurs.
