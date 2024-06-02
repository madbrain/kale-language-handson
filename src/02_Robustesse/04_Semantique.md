
# Sémantique du langage Kale

`step-2.5`

Comme vu lors des observations, il est possible d'écrire des programmes faux sans que l'interpréteur nous indique la présence d'erreurs
(ie. opérations inconsistantes, variable non définie, etc.). Ce type de comportement non autorisé est généralement donné
par la [sémantique dénotationnelle](https://en.wikipedia.org/wiki/Denotational_semantics). Pour (beaucoup) simplifier,
contrairement à la sémantique opérationnelle qui définit l'évaluation du programme, la sémantique dénotationnelle permet de
transformer le programme sous la forme d'une fonction mathématique (généralement à base de lambda-calcul). Cette transformation
s'apparente plus à la compilation et est souvent utilisée comme spécification d'un compilateur.  

En particulier, une des tâches de la sémantique est de typer précisément les différentes constructions du langage et
d'identifier les incohérences de typage. Ceci se fait par un mécanisme d'inférence de types : le typage des variables se construit
automatiquement à partir du typage de leur valeur.

Chaque construction équivalente au non-terminal `Value` de la grammaire ainsi que la définition des variables se voit affecter une valeur du type :
```typescript
enum Type {
  INTEGER,
  STRING,
  UNKNOWN
}
```
selon les règles suivantes :
* INTEGER pour les valeurs simples de type entier ou le résultat de toutes les opérations exécutées sur des opérandes de type ENTIER
* STRING  pour les valeurs simples de type chaîne ou le résultat de l'opération `+` ayant au moins un opérande de type STRING
* UNKNOWN pour tous les autres cas, c.-à-d. :
  * mauvais typage d'opération ;
  * variable non définie ;
  * construction du langage incomplète due à une erreur d'analyse grammaticale.

Grâce au typage, l'analyse sémantique peut rapporter les utilisations interdites des opérations `-`, `*` et `/`. 

# Résultat

Les programmes invalides ne sont plus exécutés et, par exemple, sur le programme suivant :
```
my_var := 10 / value
message := 20 * "30"
```

L'interpréteur affiche maintenant le diagnostic d'erreur suivant :
```
my_var := 10 / value
               ^^^^^
[0] Unknown variable 'value'

message := 20 * "30"
           ^^^^^^^^^
[1] Cannot use '*' on strings
```