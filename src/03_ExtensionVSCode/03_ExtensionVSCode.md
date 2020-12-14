# Initialisation du projet d'extension VSCode

La première étape consiste à créer le projet correspondant à l'extension VSCode.
La procédure est tirée de [la documentation officielle](https://code.visualstudio.com/docs/extensions/example-hello-world)
de VSCode et utilise le générateur de code [Yeoman](http://yeoman.io/).

```
npm install -g yo generator-code
yo code
```
Répondez aux questions dans l'ordre :
- `New Extension (Typescript)` pour le type d'extension
- `Kale Extension` comme nom d'extension
- `kale-extension` comme identifiant de l'extension
- entrez une description
- et choisissez ou pas d'initialiser un dépôt Git
- enfin choisissez le gestionnaire de package npm

Le projet généré est un projet VSCode, ouvrez l'IDE avec la commande :
```
code kale-extension
```

Le fichier `package.json` est le point d'entrée de l'extension (l'extension elle même est un package Node).
Il déclare une nouvelle commande `extension.sayHello` et indique que l'extension est activée lorsque cette
commande est invoquée.

Tout le code de l'extension est dans le fichier `src/extension.ts`.
Il comporte deux fonctions obligatoires `activate` et `deactivate`.
La première, appelée au démarrage de l'extension, donne la définition de la commande qui permet d'afficher
le fameux message d'information `Hello world!`.

Démarrez l'extension comme indiqué dans la documentation en appuyant sur la touche `F5`.
Une nouvelle instance de l'IDE démarre avec la nouvelle extension disponible.
Lancer la commande Hello en appuyant sur `F1` et en sélectionnant la commande : le message s'affiche dans un toaster.

Remarquer également l'affichage du log dans la console lors de l'activation de l'extension.

# Définition des contributions de l'extension

Notre extension va permettre de gérer un nouveau type de fichier, il faut pour
cela remplacer la section `contributes` dans le fichier `package.json` par :

```json
"contributes": {
  "languages": [{
    "id": "kale",
    "extensions": [ ".kl" ],
    "aliases": [ "Kale" ]
  }]
}
```

Démarrez de nouveau l'instance de test de l'IDE (`F5`) et ouvrez le fichier `examples/simple_example.kl` (`CTRL+O`),
vous pouvez constater que le type de fichier est bien reconnu avec l'apparition du type de fichier `Kale` (au lieu de `Plain Text`)
dans la barre de statut en bas à droite.

A part reconnaître un type de fichier, l'extension ne fait, pour l'instant, pas grand chose.
Pour aller un peu plus loin il est possible de définir une autre contribution :

```json
"grammars": [{
  "language": "kale",
  "scopeName": "source.kale",
  "path": "syntax-kale.json"
}]
```

Cette contribution permet de définir la coloration syntaxique pour le langage `kale`.
La définition du langage utilise le formalisme de l'éditeur de texte [TextMate](https://manual.macromates.com/en/language_grammars).
Créer le fichier `syntax-kale.json` à la racine de l'extension avec le contenu suivant :

```json
{
  "scopeName": "source.kale",
  "patterns": [
    {
      "name": "comment.double-slash.kale",
      "match": "//.*\\n"
    },
    {
      "match": "\\b(\\w+)\\s*(:=)",
      "captures": {
        "1": {
          "name": "entity.name.other.kale"
        },
        "2": {
          "name": "punctuation.definition.entity.kale"
        }
      }
    },
    {
      "match": "\\d+",
      "name": "constant.numeric.integer.kale"
    },
    {
      "name": "string.quoted.double.kale",
      "begin": "\"",
      "end": "\""
    }
  ]
}
```

L'éditeur dispose maintenant de la coloration syntaxique (ou plutôt lexicale pour être exact),
cette coloration dépend des catégories prédéfinies préfixées par `comment`, `keyword`, etc. et s'adapte
donc automatiquement au thème de couleurs actif.

Il est donc relativement facile de personnaliser l'IDE pour un nouveau langage sans écrire de code.
Ces personnalisations restent cependant très superficielles et les fonctionnalités telles que l'affichage
d'erreurs en ligne ou la complétion de code ne sont, quant-à-elles, possibles que par l'ajout de code.

L'activation de l'extension ne peut plus se faire sur la commande `sayHello` que nous
avons supprimée, mais sur l'ouverture d'un fichier de type Kale. Modifier le fichier `package.json`
pour inclure le code suivant :

```json
"activationEvents": [
  "onLanguage:kale"
]
```

Le message dans la console de débuggage doit de nouveau apparaître à l'ouverture d'un fichier Kale.

# Communication avec le serveur de langage

Pour communiquer avec le serveur de langage et profiter de ses fonctionnalités, il est nécessaire d'ajouter une librairie cliente :
```
npm install vscode-languageclient
```

Et vu que notre serveur de langage est également un package Node, pour simplifier la distribution,
on l'installe également comme une dépendance, locale cette fois pour simplifier le développement
(cela va nous permettre de modifier le serveur sans avoir à réinstaller la dépendance) :
```
npm install ${CHEMIN_VERS}/kale-language-server
```

Remplacer le fichier `src/extension.ts` par le code TypeScript suivant qui se limite simplement à démarrer le serveur
en indiquant le chemin vers son point d'entrée et quel type de documents il gère :

```typescript
import * as path from 'path';
import * as vscode from 'vscode';
import { LanguageClient, ServerOptions, LanguageClientOptions, TransportKind } from 'vscode-languageclient';

export function activate(context: vscode.ExtensionContext) {

	  const serverModule = context.asAbsolutePath(
        path.join(
            "node_modules",
            "kale-language-server",
            "lib",
            "server.js"
        )
    );

    const serverOptions: ServerOptions = {
		    module: serverModule,
        transport: TransportKind.ipc,
        args: ["--node-ipc"]
    };

    const clientOptions: LanguageClientOptions = {
        documentSelector: [ "kale" ],
    };

    const client = new LanguageClient(
        "kale-langserver",
        "Kale Language Server",
        serverOptions,
        clientOptions
    );
       
    context.subscriptions.push(client.start());
}

export function deactivate() {
}
```

Ici nous profitons de facilités offertes part le fait que notre serveur s'exécute avec NodeJS, mais il est également possible
de lancer n'importe quel exécutable de son choix et de choisir le mode de communication avec le client (stdio, socket, etc.).

L'édition d'un fichier Kale dans l'éditeur de test permet maintenant de constater que les erreurs s'affichent automatiquement
à la frappe au clavier.

# Packaging de l'extension

Une fois satisfait des fonctionnalité de votre extension, il est temps de la packager pour pouvoir la distribuer à vos collègues,
pour cela installez l'outil `vsce` :
```
npm install -g vsce
```

Malheureusement un [bug](https://github.com/microsoft/vscode-vsce/issues/203) de l'outil nous force
à d'abord packager localement `kale-language-server`, pour cela créer dans le projet le fichier `.npmignore`
contenant :
```
.vscode/
src/
examples/
tsconfig.json
.mocharc.json
.gitignore
```
afin d'exclure les fichiers servant uniquement au développement. Puis lancez la commande :
```
npm pack
```
afin de packager le projet et d'obtenir le `.tar.gz` correspondant.

Enfin dans le projet d'extension, ajouter dans le fichier `package.json` le champ `publisher` avec par exemple votre nom.
Si vous voulez publier l'extension sur le MarketPlace VSCode, ce nom devra d'abord avoir été déclaré en vous enregistrant.

Ensuite il faut remplacer la dépendance directe sur le répertoire du projet du serveur de langage par une référence sur le
`.tar.gz` et utiliser `vsce` afin de packager l'extension :
```bash
npm install ${CHEMIN_VERS}/kale-language-server/kale-language-server-1.0.0.tgz
vsce package
```

ce qui permet enfin d'obtenir le package d'extension `.vsix`. Depuis VSCode, cette extension peut être installée avec
la commande `Extensions: install from VSIX...` (`CTRL-SHIFT-P` pour ouvrir la palette).