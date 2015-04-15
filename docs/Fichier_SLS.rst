************************
Mon premier fichier SLS
************************

.. |minion| replace:: ``Minion``
.. |master| replace:: ``Master``
.. |minions| replace:: ``Minions``
.. |salt| replace:: **Salt**
.. |state| replace:: **state**
.. |sls| replace:: *SLS*
.. |nginx| replace:: ``nginx``

Le système |state| est construit sur les formules |sls|. Ces formules sont construites dans un fichier sur le serveur de fichier |salt|. Pour faire une formule très basique |sls|, il suffit d'ouvrir un fichier sous ``/srv/salt`` nommé ``vim.sls``. Les |state| suivant assure que *vim* est installé sur un système sur lequel un |state| a été appliqué.


``/srv/salt/vim.sls``:


.. code-block:: yaml

	vim:
	   pkg.installed


A partir de là, on installe ``vim`` sur le |minion| en appelant le |sls| directement:

.. code-block:: bash

    salt '*' state.sls vim


Cette commande va invoquer le système |state| et lancer le |sls| ``vim``.

Maintenant, pour renforcer la formule |sls| ``vim``, un ``vimrc`` peut-être ajouté:

``/srv/salt/vim.sls``:

.. code-block:: yaml

	vim:
	  pkg.installed: []

	/etc/vimrc:
	  file.managed:
	    - source: salt://vimrc
		- mode: 644
		- user: root
		- user: root


Maintenant le fichier ``vimrc`` désiré doit être copié dans le serveur de fichier |salt| dans ``/srv/salt/vimrc``. Dans |salt|, tout est fichier, donc auncune redirection de chemin n'a besoin d'être justifié. Le fichier ``vimrc`` est placé juste à coté du fichier ``vim.sls``. La même commande que précédemment peut être éxécuté pour toutes les formules ``vim`` |sls| et vont inclure le fichier de configuration.



Ajout de profondeur
--------------------


Evidemment, maintenir les formules |sls| dans un simple repertoire à la racine du serveur de fichier ne va pas favoriser pour un gros déploiement. C'est pourquoi plus de profondeur. Commençons par faire une formule ``nginx`` d'une meilleur façon, faisons un sous-fichier |nginx| et ajoutons un fichier ``init.sls``:

``/srv/salt/nginx/init.sls``:

.. code-block:: yaml

	nginx:
	  pkg.installed: []
		service.running:
		  -require:
			- pkg: nginx


Quelques concepts sont introduits dans cette formule |sls|.

D'abord, nous avons la déclaration du service qui va s'assurer que le service |nginx| est démarré.

Bien sûr, le service |nginx| ne peut pas être démarré à moins que le paquet soit installé d'où la déclaration ``require`` qui implante une dependance entre les 2.

La déclaration ``require`` s'assure que le composant requit est exécuté avant et que le résultat soit un succés.

Cette nouvelle formule |sls|  a un nom spécial ``init.sls``. Quand une formule |sls| est nommé ``init.sls`` elle hérite du nom du dossier qui la contient. Cette formule peut être référencé via la commande suivante:

.. code-block:: bash

	salt '*' state.sls nginx


Maintenant que les sous-repertoires peuvent être utilisés, la formule ``vim.sls`` peut être effacé. Pour faire les chose d'une manière plus flexible, bougez le ``vim.sls`` et ``vimrc`` dans un sous-repertoire appelé ``edit`` et changez le fichier ``vim.sls`` pour refleter le changement:

``/srv/salt/edit/vim.sls``:

.. code-block:: yaml

	vim:
	  pkg.installed

	/etc/vimrc:
	  file.managed:
	    - source: salt://edit/vimrc
		- mode: 644
		- user: root
		- group: root


Seulement le chemin source du fichier ``vimrc`` a changé. Maintenant la formule est référencé en tant que ``edit.vim`` parcqu'il réside dans le sous-dossier ``edit``. Maintenant le sous-dossier peut contenir des formules pour ``emacs``, ``nano``, ``joe`` ou n'importe quel autre editeur de texte qui devra être déployé.
