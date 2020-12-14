# Conclusion

Voilà ! c'est la fin de ce tutoriel, mais c'est seulement le début de vos explorations dans le monde
des serveurs de langage, rien que sur notre exemple de langage, des dizaines de fonctionnalités peuvent être ajoutées :
* ajouter une action de réfactoring sur les variables indéfinies pour leur ajouter une définition
* afficher les définitions et références correspondant à une variable en répondant à la requête [`textDocument/documentHighlight`](https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_documentHighlight)
* ajouter le réfactoring de renommage en répondant à la requête [`textDocument/rename`](https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_rename)
* formater le code en répondant à la requête [`textDocument/formatting`](https://microsoft.github.io/language-server-protocol/specifications/specification-current/#textDocument_formatting)
* etc.

Vous pouvez également tester des évolutions du langage lui même en ajoutant :
* l'ajout de conditions sur les affectations ;
* l'interpolation de variables dans les chaînes de caractères ;
* l'appel de fonctions ;
* etc.