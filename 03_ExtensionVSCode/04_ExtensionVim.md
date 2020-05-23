
# Extension Vim

VSCode est à l'origine du protocole LSP, mais aujourd'hui un nombre croissant d'autres
[éditeurs](https://microsoft.github.io/language-server-protocol/implementors/tools/) permet
de se connecter à un serveur de langage, c'est le cas du célèbre éditeur **Vim**. Cette fonctionnalité n'est pas native,
mais l'adjonction de plugins et un peu de configuration lui permet de délivrer les
mêmes sensations que votre IDE préférée !

## Installation

Il faut tout d'abord installer le gestionnaire de plugin [vim-plug](https://github.com/junegunn/vim-plug).

Ensuite éditez votre fichier de configuration `~/.vimrc` en y ajoutant :
```bash
call plug#begin('~/.vim/plugged')

Plug 'natebosch/vim-lsc'

call plug#end()
```

Cette configuration va permettre d'installer [vim-lsc](https://github.com/natebosch/vim-lsc), qui va faire
le pont entre Vim et les serveurs de langages. Pour finaliser l'installation, lancez Vim et taper la commande :
```
:PlugInstall
```

Après téléchargement, il ne reste plus qu'à déclarer le lien entre type de fichier et serveur de langage,
dans votre fichier `~/.vimrc`, ajoutez :
```bash
au BufRead,BufNewFile *.kl set filetype=kale
```
Cette configuration (native de Vim) permet d'affecter le type `kale` aux fichiers d'extension `.kl`.
À partir de là, il serait également possible d'écrire un fichier définissant la coloration syntaxique,
libre à vous d'en écrire un à l'aide de ce [tutoriel](https://vim.fandom.com/wiki/Creating_your_own_syntax_files).

Finalement pour ajouter le lien entre le type de fichier `kale` et notre serveur de langage, ajoutez au fichier `~/.vimrc` :
```bash
let g:lsc_server_commands = {
  \ 'kale': '${KALE_LANGUAGE_SERVER}/bin/kale-langserver --stdio'
  \ }
```
en remplaçant `${KALE_LANGUAGE_SERVER}` par le chemin complet vers le script de lancement du serveur (vous pouvez également
mettre le répertoire `bin` dans votre `PATH`).

## Résultat

Vous pouvez éditer vos fichiers Kale avec Vim comme d'habitude mais avec maintenant l'affichage des erreurs à la frappe !
En fonction des capacité du serveur de langage utilisé, il est également possible d'utiliser la complétion, réfactoring, etc.
