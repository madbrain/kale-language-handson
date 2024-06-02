
# Analyseur grammatical

## Principe

Maintenant que vous disposez d'un analyseur lexical capable de découper le code source en lexèmes,
il est tant d'en extraire la structure. Comme pour les langages naturels, le concept de grammaire est
utilisé pour décrire la structure du langage.

Voici la définition de la grammaire du langage Kale :
```
KaleFile ::= Assignment* EOF
Assignment ::= IDENT ASSIGN Value
Value ::= MulDivValue ( epsilon | (+ | -) MulDivValue )*
MulDivValue ::= AtomValue ( epsilon | (* | /) AtomValue )*
AtomValue ::= INTEGER | IDENT | STRING
```

Les éléments de la grammaire entièrement en majuscule (ie. `INTEGER`) correspondent aux lexèmes et sont appelés les terminaux.
Les autres éléments sont les non-terminaux et représentent le résultat des règles qui structurent le langage.
La grammaire se lit de cette façon :
* la règle `Assignment ::= IDENT ASSIGN Value` signifie que la séquence formée de `IDENT`, `ASSIGN` et `Value` peut se réduire à `Assignment` ;
* `|` indique une alternative, ie. `INTEGER` ou `IDENT` ou `STRING` peuvent se réduire à `AtomValue`. Ceci est équivalent à avoir écrit séparément
  les règles suivantes :
```
AtomValue ::= INTEGER
AtomValue ::= IDENT
AtomValue ::= STRING
```
* `*` indique que l'élément précédent peut être répété 0 ou plusieurs fois ;
* les parenthèses servent à préciser les priorités des opérations ;
* `epsilon` représente la séquence vide c.-à-d. l'absence d'éléments.

La structure résultant de l'analyse du language est définie dans le fichier `ast.ts` : chaque interface correspond
à une règle de la grammaire .

Il faut maintenant définir la machine capable d'extraire la structure à partir de cette grammaire : le *parser*.
Les parsers sont classés en fonction de la complexité du langage qu'ils sont capable de reconnaître.
Les parsers les plus simples à implémenter sont ceux dit LL(1).

Attention ce type de parser ne permet pas d'être utilisé avec n'importe qu'elle grammaire :
elle ne doit pas contenir de [règles récursives à gauche et être factorisé](https://en.wikipedia.org/wiki/LL_parser#Solutions_to_LL(1)_Conflicts).
Mais pas d'inquiétude, cette grammaire a été écrite pour être de type LL(1)
(d'où les règles `Value` et `MulDivValue` écrites d'une façon pas très naturelle).

## Implémentation

Voici la structure générale du parser :

```typescript
class Parser {
    private token: Token;
    
    constructor(private lexer: Lexer) {
        this.token = this.scanToken()
    }

    parseFile(): KaleFile {
        throw new Error("TODO: not complete");
    }

    private scanToken() {
        this.token = this.lexer.nextToken();
        return this.token;
    }

    private test(kind: TokenKind) {
        if (this.token.kind == kind) {
            this.scanToken();
            return true;
        }
        return false;
    }

    private expect(kind: TokenKind) {
        if (this.token.kind != kind) {
            throw new Error(`Expecting token ${kind}, got ${this.token.kind}`);
        }
        this.scanToken();
    }
}
```

Les parsers de type LL(1) peuvent être écrit sous la forme d'un automate procédurale, c.-à-d. que chaque
non-terminal de la grammaire est décrit par une méthode du parser. Par exemple la règle `Assignment ::= IDENT ASSING Value`
va se traduire en :

```typescript
class Parser {
    private parseAssignment(): Assignment {
        const variable = this.parseIdent();
        this.expect(TokenKind.ASSIGN);
        const value = this.parseValue();
        return { variable, value };
    }
}
```

Si lors de l'analyse d'un non-terminal peut se poursuivre par plusieurs types de lexèmes : ils seront tout à tour tester avec la fonction `test`.
Les répétitions dans la grammaire représentées par `*` deviennent des boucles dans le parser.

## Résultat

Une fois que l'ensemble des tests de l'étape `step-1.2` passent au vert (`npm test`), il est possible de tester
le résultat du parser sur les fichiers d'exemples (après compilation avec `npm run build`) :
```
./bin/kale-interpreter examples/simple_example.kl
```

Le programme doit maintenant afficher l'arbre de syntaxe au format Json.