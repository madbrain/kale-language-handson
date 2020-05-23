
# Meilleur reporting

Afin de faciliter l'apprentissage du langage, il est important de donner un diagnostiques précis des erreurs
et en particulier indiquer la position de l'erreur : la ligne voire mieux l'intervalle dans la ligne en erreur.

Les utilisateurs peuvent à tout moment éditer un texte en erreur. Si l'erreur est située avant
leur position d'édition l'analyse s'arrête et ils n'auront alors aucune information de potentielles erreurs
à la position d'édition. Il est donc important d'avoir une analyse suffisamment robuste pour pouvoir indiquer
le maximum d'erreurs possibles à l'utilisateur et en particulier à la position d'édition.

## Positionnement

```typescript
interface Position {
    offset: number;
    line: number;
    character: number;
}

interface Span {
    from: Position;
    to: Position
}
```

Les informations ligne/character et offset paraissent redondantes mais chaque éditeur de texte
gère le positionnement dans le texte de façon différente :
- le language Server Protocol utilisé par VSCode n'accepte que ligne/colonne ;
- Codemirror accepte plutôt ligne/colonne, mais des fonctions de conversion existent ;
- le positionnement brut dans un fichier est plus aisé avec l'offset.

C'est donc une bonne pratique d'utiliser le triplet ligne/colonne/offset pour le positionnement dans le texte.

## Rapport d'erreurs multiples

Pour rapporter les erreurs, les différentes phases de l'analyse disposent d'un objet du type suivant :
```typescript
interface ErrorReporter {
    reportError(span: Span, message: string);
}
```

## Description des outils de test

Afin de simplifier l'écriture des tests la fonction utilitaire `code` est utilisée pour définir à la fois le code à analyser
ainsi que la position de marqueurs (`@{1}`, `@{2}`, etc.). Ces marqueurs sont ensuite utilisés pour comparer avec les positions
fournis par le résultat de l'analyse.