# Préparation du projet

Pour mener à bien vôtre mission quelques outils vous seront nécessaires:
- [Git](https://git-scm.com/)
- [Node.js](https://nodejs.org/)
- [Visual Studio Code](https://code.visualstudio.com)

Commencez par cloner la souche du projet:
```
git clone https://github.com/madbrain/kale-language-server.git
```

Ce projet contient tout le nécessaire pour construire l'outillage d'analyse de notre language.
Le développement sera guidé par des tests découpés en différentes étapes progressives,
à chaque étape de nouveaux tests nous amènent à écrire une nouvelle fonctionnalité.

Le projet contient:
- `src`, les sources de l'analyseur sur lesquelles nous allons nous concentrer durant le tutoriel ;
- `bin`, les programmes exécutables qui vont nous permettre d'exploiter le langage (un interpréteur en fait).  
- `examples`, quelques exemples de notre DSL.  

Les sources sont écrite principalement en Typescript ce qui permet de les utiliser sans adaptation
à la fois dans une application Web et aussi comme une extension VSCode.
Cependant aucun framework exotique n'est utilisé et il est extrêmement simple d'appliquer les mêmes techniques
avec n'importe quel langage de votre choix.

Pour démarrer le tutoriel il faut commencer par créer une branch à partir du tag `step-1.1` :
```
git checkout -b tutoriel step-1.1
```

Il faut ensuite installer l'ensemble des dépendances nécessaire au tutoriel, pour cela exécutez la commande suivante:
```
npm install
```

À chaque étape il faut compléter le code afin que l'ensemble des tests soit vert :
```
npm test
```

Quand c'est le cas, il faut passer à l'étape suivante en fusionnant le tag suivant :
```
git merge step-1.2
```

Si vous bloquez, vous pouvez consulter la solution proposée à chaque étape sur le tag correspondant,
par exemple la solution de l'étape `step-1.1` se trouve au tag `solution-step-1.1`.

# Présentation du langage Kale[^1]

Le département comptabilité de votre entreprise réalise des calculs extrêmement complexes
pour lesquels les formules Excel ont atteint leur limites (depuis le temps qu'on le dit...)
et vous a missionné pour concevoir un langage simple mais facile à utiliser qui leur permettra
de simplifier leur dur labeur.

Voici le langage que vous avez conçu, il permet d'exprimer des calculs à l'aide
d'expressions arithmétiques et de variables, la variable `message` sera affichée à l'écran :
```c
// Calcul du prix total
prix_unitaire := 10
forfait := 100
total := prix_unitaire * 20 + forfait
message := "Le prix est " + total
```

Ce langage va être analysé en plusieurs étapes :
- découpage du texte en mots : analyse lexicale
- structuration des mots en phrases : analyse grammaticale
- vérification du sens des phrases : analyse sémantique

Chaque étapes repose sur les résultats produits à l'étape précédente.

[^1]: l'origine de ce nom reste un mystère : un chanteur de blues, un légume hype, le Kangooroo Arithmetic Language Evaluator ? qui sait...
