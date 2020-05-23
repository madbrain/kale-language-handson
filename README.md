# Introduction

Les DSL (*Domain Specific Language*) sont partout, nous les utilisons tous les jours sans vraiment s'en rendre compte
(SQL, Maven, Ansible, etc. [encore plus d'exemples](https://tomassetti.me/domain-specific-languages/)).
Les méthodologies de conception telles que le DDD (*Domain Driven Design*)
invitent a représenter le domaine métier au travers d'un langage clairement défini : un DSL.
Les DSL ont donc pour but de nous rendre plus productif en se concentrant sur l'essentiel.
Mais l'accroissement du nombre de langage à apprendre et à maîtriser peut rendre la tâche ardue.

Le DSL doit être accompagné de l'outillage permettant un apprentissage rapide.
Dans l'ère post-IntelliJ[^1], cet outillage doit au minimum :
* afficher des erreurs à la frappe,
* fournir une complétion de code intelligente,
* permettre des refactorings.

## Types de DSL

Pour une analyse plus en profondeur de l'utilisation des langages spécifiques et de l'outillage associé,
je vous invite à lire l'[article de Martin Fowler](https://www.martinfowler.com/articles/languageWorkbench.html).

Voici pour résumer les différentes implémentations possibles d'un DSL.

### DSL Interne

Un DSL interne utilise la syntaxe de votre langage hôte (Java, Groovy, Python, etc.),
l'outillage est donc le même. Par contre en fonction des capacités du langage hôte, l'expressivité peut être limitée
et il est souvent difficiles d'exprimer des constructions complexes
(vérification de contraintes liées aux données, etc.).

### DSL externe :

#### Langage valise

La grande tendance a été d'utiliser des langages comme XML, JSON et maintenant YAML comme DSL, certainement
pour leur outillage standard (grand nombre de bibliothèque de lecture/écriture). Ces outils sont aussi
leur faiblesse, même s'il possède parfois la notion de schéma, ils n'ont pas été conçus pour aider les utilisateurs
avec la sémantique spécifique des langages métiers.

Finalement ces syntaxes clés en main contiennent :
* plus de bruit que de contenu (XML)
* des constructions alternatives ambiguës (YAML).

#### Langage propre

L'écriture d'analyseurs spécifiques à un langage (lexer/parser) est perçue par la plupart des développeurs
comme une tâche compliquée voire intimidante, certainement une réminiscence des cours de compilation qui
avait pour objectifs d'expliquer les techniques de compilation pour un language généraliste.
La portée d'un DSL (comme son nom l'indique) est bien plus restreinte et les techniques misent en œuvre
pour l'analyser suivent la même tendance.

Nombre d'outils sont disponibles pour aider à la réalisation de lexer/parser (flex, bison, ANTLR, etc.).
Ces outils permettent d'être rapidement productif, mais deviendront à un moment ou un autre également le facteur limitant :
* ils n'aident pas vraiment à comprendre la théorie (malheureusement indispensable) de l'analyse de langage (LL/LR, récursivité à gauche, etc.)
* il est souvent difficile de comprendre le fonctionnement du code qu'ils produisent et offrent un débogage limité
* et il est difficile, voire impossible, de personnaliser leur fonctionnement pour les faire correspondre à vos besoins précis.
  En particulier la gestion robuste des erreurs qui se produisent systématiquement dans le cas d'une analyse interactive où
  l'utilisateur écrit son code progressivement.

Il est en fait souvent plus simple d'écrire son propre lexer/parser pour des petits langages comme les DSL.
Cela permet de réaliser l'ensemble des modifications nécessaires aux spécificités du traitement interactif
grace à un code 100% maîtrisé. Et nous allons voir qu'en quelques étapes il est possible d'obtenir un outillage
robuste et puissant pour un petit DSL.

## Objectifs

Les objectifs ambitieux mais réalisables sont :

* Écrire l'outillage pour aider à l'écriture d'un petit DSL, avec entre autre :
  * affichage des erreurs à la frappe
  * complétion de code
  * refactoring
* Intégration de l'outillage dans VSCode ou une application Web.

[^1]: «post-IntelliJ» est le terme utilisé par Martin Fowler, je n'ai personnellement aucun a priori, remplacez par votre IDE à tout faire préféré!
