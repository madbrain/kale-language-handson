
# Robustesse du lexer

## Etape 1 : Ajout du positionnement des lexèmes

`step-2.1`

Chaque lexème dispose maintenant d'un champ span qui reflète sa position exacte dans le fichier :
```typescript
export Token {
  span: Span;
  kind: TokenKind;
  value?: string;
}
```

Les méthodes `getChar` et `putBackChar` doivent comptabiliser les positions (ligne et colonne)
à chaque fois qu'elles renvoient ou reprennent un caractère. Attention lorsque `putBackChar` reprend un retour chariot :
il faut retourner à la fin de la ligne précédente !

## Etape 2 : Robustesse

`step-2.2`

Afin d'être robuste, le lexer ne peut plus s'arrêter par une exception. Cela se produit sur deux types d'erreurs possibles :
- soit il a commencé à lire un lexème mais il ne se termine pas correctement
  (ie. une chaîne de caractères sans délimiteur de fin) alors une erreur est rapportée mais le lexème est retourné avec la portion déjà lu.
  S'il y a plusieurs types de lexème possibles, le type est choisi arbitrairement.
- soit le caractère lu ne correspond à rien alors une erreur est rapportée et l'analyse reprend à l'état initial,
  ce qui a pour effet d'ignorer le(s) caractère(s) en erreur.

## Résultat

Le lexer ne s'arrête plus à la première erreur et rapporte précisément l'erreur. Par exemple sur le programme suivant :
```
my_var := 10 @ + 30
message := "result :
```

L'interpréteur affiche maintenant le diagnostic d'erreur suivant :
```
my_var := 10 @ + 30
             ^
[0] Unknown character(s)

message := "result :
           ^^^^^^^^^
[1] Unterminated string
```
