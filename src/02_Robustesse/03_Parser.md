
# Robustesse du parser

## Etape 1 : Ajout du positionnement de l'arbre de syntaxe

`step-2.3`

Chaque nœud de l'arbre de syntaxe dispose maintenant d'un champ span qui reflète sa position exacte dans le fichier :
```typescript
export AstNode {
  span: Span;
}
```

La fonction `mergeSpan` est bien utile pour obtenir l'intervalle complet d'un nœud à partir de ses fils ou des lexèmes.

## Etape 2 : Robustesse

`step-2.4`

Actuellement lors d'une erreur de syntaxe le parser s'arrête avec une exception, il nous faut être capable de rapporter l'erreur
mais d'être capable de continuer l'analyse sur le reste du code.
Pour cela, la technique la plus simple est celle du *panic mode*.
Cette technique consiste, en cas d'erreur, à ignorer les lexèmes du flux
à la recherche d'un lexème de synchronisation. Les lexèmes de synchronisation sont choisis de telle sorte que
si l'analyse reprend à cet endroit elle a de grandes chances de se dérouler correctement.
L'analyse reprends avec la règle adéquate en fonction du lexème de synchronisation :
par exemple dans le langage C, le lexème qui suit `;` permet de redémarrer l'analyse au début d'une instruction.

Il est également possible d'insérer/modifier le(s) lexème(s) attendu(s) en espérant que l'analyse reprenne correctement,
par exemple en ajoutant une accolade de fin de bloc. Faire le bon choix entre les différentes modifications requiert généralement
de tester les différentes solutions et de choisir celle qui mène l'analyse la plus au delà de l'erreur précédente.
Ce mode de reprise sur erreur est plus performant en termes de capacité de récupération, mais est bien plus compliqué que le simple
*panic mode*.

Deux nouvelles méthodes utilitaires du parser sont nécessaires :

```typescript
private recoverWith<T>(syncTokens: TokenKind[], start: Span,
                       makeError: (span: Span) => T, parseFunc: () => T): T {
    try {
        return parseFunc();
    } catch(e) {
        const tokens = this.skipTo(syncTokens);
        const range = tokens.length > 0 ? mergeSpan(start, tokens[tokens.length-1].span) : start;
        return makeError(range);
    }
}

private skipTo(syncTokens: TokenKind[]): Token[] {
    const tokens: Token[] = []
    while (! (this.token.kind == TokenKind.EOF || syncTokens.indexOf(this.token.kind) >= 0)) {
        tokens.push(this.token);
        this.scanToken();
    }
    return tokens;
}
```

La méthode `recoverWith` permet de protéger l'exécution du bloc de code d'analyse `parseFunc` et en cas d'erreur
pendant l'analyse `skipTo` avance la lecture des lexèmes jusqu'à rencontrer un token de synchronisation ou EOF.
La fonction `makeError` permet de construire un nœud de syntaxe représentant le fait qu'il y a eu une erreur
sur l'intervalle en paramètre.

Par exemple lors de l'analyse d'une condition `Condition ::= Expr EQUALS Expr`, si une erreur survient pendant l'analyse
de la deuxième expression, on construit un nœud partiel contenant le maximum d'information accumulée jusque là :

```typescript
private parseCondition(): Condition {
  const startSpan = this.token.span;
  const left = parseExpr();
  expect(TokenKind.EQUALS);
  return this.recoverWith(CONDITION_SYNC_TOKENS, startSpan, (endSpan) => {
      return { span: mergeSpan(startSpan, endSpan), isOk: false, op: Relation.Equals, left };
  }, () => {
      const right = parseExpr();
      return { span: mergeSpan(expr.span, right.span), isOk: true, op: Relation.Equals, left, right };
  });
}
```

L'analyse reprendra sur une des lexèmes de `CONDITION_SYNC_TOKENS`. cet ensemble de lexèmes doit permettre de redémarrer l'analyse comme
si le non-terminal `Condition` avait été émit et contient donc l'ensemble des lexèmes valides après ce non-terminal.

Chaque non-terminal a un ensemble de lexèmes de synchronisation différent et correspond aux fameux [Follow Set](https://www.cs.uaf.edu/~cs331/notes/FirstFollow.pdf). Si un non-terminal est utilisé dans des contextes très différents son ensemble peut être spécialisé et réduit
pour chaque règle où il est utilisé.

Voici pour notre grammaire le résultat su calcul des ensembles First et Follow : 

### First Set

```
First(KaleFile) = { IDENT }
First(Assignment) = { IDENT }
First(Value) = { INTEGER, IDENT, STRING }
First(MulDivValue) = { INTEGER, IDENT, STRING }
First(AtomValue) = { INTEGER, IDENT, STRING }
```

### Follow Set

```
Follow(Assignment) = { IDENT, EOF }
Follow(Value) = { IDENT, EOF }
Follow(MulDivValue) = { ADD, SUBSTRACT, IDENT, EOF }
Follow(AtomValue) = { ADD, SUBSTRACT, MULTIPLY, DIVIDE, IDENT, EOF }
```

## Résultat

Le parser ne s'arrête plus à la première erreur et rapporte précisément l'erreur. Par exemple sur le programme suivant :
```
my_var := 10 / + 30
my_other_var + 20
```

L'interpréteur affiche maintenant le diagnostic d'erreur suivant :
```
my_var := 10 / + 30
               ^
[0] Expecting STRING, got ADD

my_other_var + 20
             ^
[1] Expecting ASSIGN, got ADD
```