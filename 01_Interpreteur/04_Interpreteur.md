
# Interpréteur

## Principe

Un fois l'arbre de syntaxe obtenu avec le parser, il est possible de l'interpréter afin d'obtenir le résultat
réel du calcul. Pour les puristes, l'interpréteur peut se définir à partir de la
[sémantique opérationnelle](https://en.wikipedia.org/wiki/Operational_semantics) suivante:

$$
\frac{\langle E,s\rangle \Rightarrow V}{\langle L:=E\,,\,s\rangle\longrightarrow (s\uplus (L\mapsto V))}
$$

$$
\frac{\langle E_1,s\rangle \Rightarrow V_1,\langle E_2,s\rangle \Rightarrow V_2}{\langle E_1 op E_2,s\rangle\longrightarrow (V_1 op V_2)}
$$

$$
\frac{}{\langle I,s\rangle\longrightarrow (I)}
$$

$$
\frac{}{\langle S,s\rangle\longrightarrow (S)}
$$

$$
\frac{(L\mapsto V) \in s}{\langle L,s\rangle\longrightarrow (V)}
$$

$$
\frac{\langle A_1,s\rangle \longrightarrow s'}{\langle A_1 A_2,s\rangle \longrightarrow \langle A_2,s'\rangle}
$$

$$
\frac{(message\mapsto V) \in s}{\langle ,s\rangle \longrightarrow print(V)}
$$

Si ce charabia mathématique n'a aucun sens pour vous : c'est pas grave !
Les définitions ci-dessus comportent de plus certainement quelques erreurs, mais il est important d'avoir une idée des outils
standards utilisés dans la conception de langages. En effet un bon langage doit reposer sur des bases solides
et la définition formelle de sa sémantique en est une.

Globalement les règles ci-dessus indiquent que tout au long de l'évaluation du programme un état $$s$$ va être lu et mis à jour.
De façon concrète cet état est du type `Map<string, string | number>` et permet de stocker le résultat de l'évaluation des affectations.
La sémantique indique également que :
* les affections sont évaluées dans l'ordre dans lequel elles apparaissent dans le fichier
* la valeur de l'affectation est calculée par évaluation des opérations
* la valeur d'une variable est lue depuis l'état
* et à la fin la valeur de la variable `message` est affichée.

## Résultat

Une fois que l'ensemble des tests de l'étape `step-1.3` passent au vert, il est possible de tester
le résultat de l'interpréteur sur les fichiers d'exemples :
```
./bin/kale-interpreter example/simple_example.kl
```

Le programme affiche maintenant le résultat de l'évaluation tant attendu, hourra !.

## Observations

Mais pas si vite ! que se passe t-il si l'on essaye d'évaluer les programmes suivants :
```
message := 30 @ "tutu"
```

```
message := 30 + / "tutu"
```

```
message := 30 * "tutu"
```

Dans les deux premiers cas il plante lamentablement avec une indication d'erreur pas très explicite : est ce une erreur dans le programme évalué
ou un bug dans l'interpréteur ? Il est clair qu'il y a respectivement une erreur lexicale et grammaticale dans les programmes d'exemples.
Mais à part le message, l'interpréteur ne nous donne aucune information de position (ligne, colonne) : imaginer trouver l'erreur dans un programme
de taille plus conséquente.

Les experts Javascript peuvent trouver le résultat de la dernière évaluation correct, par contre pour vos clients ça n'a aucun sens de multiplier un
nombre et une chaîne de caractères et l'interpréteur devrait clairement l'indiquer, tout comme utiliser une variable non définie, etc..

Il serait également intéressant que l'évaluateur soit robuste et soit capable d'indiquer l'ensemble des erreurs d'un programme et pas juste la première[^1]. Par exemple dans le programme suivant :
```
my_var := 30 + / 50
message := "resultat : " + my_other_var
```

Il doit être capable d'indiquer une erreur au niveau du symbole `/` et que dans la deuxième expression il est fait référence
de la variable `my_other_var` non définie.

Dans la seconde partie du tutoriel nous allons rendre l'interpréteur robuste à tous types d'erreur et lui donner un rapport d'erreur précis.

[^1]: notre objectif est l'aide à l'écriture de code et contrairement à l'évaluation un programme, cela peut passer par des phases où il contient beaucoup d'erreurs (ie. pendant les phases de réfactoring).