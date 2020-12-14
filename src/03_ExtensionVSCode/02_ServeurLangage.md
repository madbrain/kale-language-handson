
# Serveur de langage Kale

`step 3.1`

Le serveur de langage repose sur la bibliothèque `vscode-languageserver`. Afin de disposer de ces nouvelles dépendances, lancez la commande :
```
npm install
```

L'objet principal qui centralise toute l'infrastructure du serveur est l'objet obtenu avec la fonction `createConnection` :

```typescript
import { createConnection, ProposedFeatures } from "vscode-languageserver";

const connection = createConnection(ProposedFeatures.all);

/*
 * ... more initialisations
 */

connection.listen();
```

Pour chaque requête ou notification à traiter l'objet `connection` propose un *hook* pour intercepter le message et dans le cas
d'une requête pouvoir envoyer la réponse. Par exemple :

```typescript
import { InitializeParams, InitializeResult, TextDocumentSyncKind } from "vscode-languageserver";

connection.onInitialize((params: InitializeParams): InitializeResult => {
	return {
		capabilities: {
			textDocumentSync: TextDocumentSyncKind.Incremental,
		} as any
	}
});
```

Le serveur doit garder une version des documents ouverts afin de suivre leur modifications. La bibliothèque `vscode-languageserver`
met à disposition la fonction `TextDocument.create()` pour créer des documents sur lesquels la fonction `TextDocument.applyEdits()`
permet d'appliquer les modifications. A chaque action sur un document (ouverture, changement et fermeture) le document doit être
validé avec notre lexer/parser/checker maison et les erreurs publiées sous forme de diagnostiques avec la méthode `connection.sendDiagnostics()`.

# Résultat

Ça y est votre serveur de langage est prêt ! Mais sans un vrai client graphique l'expérience est plutôt limitée.
Dans le chapitre suivant une extension VSCode va être développée pour communiquer avec notre serveur.