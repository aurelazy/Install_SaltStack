*****************************
Salt State
*****************************

.. |minion| replace:: ``Minion``
.. |master| replace:: ``Master``
.. |minions| replace:: ``Minions``
.. |salt| replace:: **Salt**
.. |state| replace:: **state**
.. |sls| replace:: *SLS*
.. |id| replace:: ``ID``


Comme vu précédement, voici un fichier |sls| simple, il est écrit en ``YAML``:

.. code-block:: yaml

	apache:
	  pkg.installed: []
	  service.running
	    - require:
		  - pkg: apache


Ces données |sls| vous s'assurer que le paquet nommé apache est installé, et que le service apache est démarré.Les composants peuvent être expliqué d'une façon simple.

La première ligne est l'|id| pour un ensemble de données, et est appelé ``ID Declaration``. Cet |id| determine le nom de l'objet qui doit être manipulé.

La seconde et troisième ligne contiennent le ``state module function`` à lancer, au format ``<state_module>.<function>``. Le ``state module function`` ``service.running`` s'assure qu'un daemon est démarré.

Finallement, sur la ligne 5, nous avons le mot ``require``. On l'appel un ``Requisite Statement``, et il vérifie que le service ``Apache`` sera lancé seulement si l'installation du paquet c'est fait correctement.


Ajout de configurations et d'utilisateurs
--------------------------------------------

Lorsque l'on installe un service comme *Apache*, beaucoup plus de composants doivent être ajoutés. La configuration d'*apache* doit être géré, et un utilisateur et un group doit être installé.

.. code-block:: yaml

	apache:
	  pkg.installed: []
	  service.running:
	    - watch:
		  - pkg: apache
		  - file: /etc/httpd/conf/httpd.conf
		  - user: apache
	  user.present:
	    - uid: 87
		- gid: 87
		- home: /var/www/html
		- shell: /bin/nologin
		- require:
		  - group: apache
	  group.present:
	    - gid: 87
		- require:
		  - pkg: apache
		  
	/etc/httpd/conf/httpd.conf:
	  file.managed:
	    - source: salt://apache/httpd.conf
		- user: root
		- group: root
		- mode: 644

Ces données |sls| étendent considerablement le premier exemple, et inclus un fichier de configuration, un utilisateur, un groupe et un nouveau ``requisite statement``: ``watch``.

Ajouter plus de |state| est facile, dès lors que le nouvel |state| utilisateur et groupe sont en dessous de l'|id| ``apache``, l'utilisateur et le groupe seront l'utilisateur et le groupe d'``Apache``. 
Les ``require`` déclarations* vont s'assurer que l'utilisateur va être créé après le groupe, et que le groupe ne sera créé qu'après l'installation de paquet ``Apache``.

Ensuite, la déclaration ``require`` sous *service* a été changé par ``watch``, et il vérifie maintenant 3 |state| au lieu d'un seul. 
La déclaration* ``watch`` fait la même chose que ``require``, il s'assure que les autres |state| soient démarrés avant le |state| ``watch``, mais il ajoute un composant supplémentaire.
La déclaration* ``watch`` va lancer le *state watcher function* (observateur d'état) pour tout changement effectué sur le *watched state*.
Donc si le paquet a été updaté, le fichier de configuration changé, ou l'UID modifié, alors le *service state watcher* sera lancé.
Le *service state watcher* va relancer le service, donc dans ce cas, un changement dans le fichier de configuration va également amorcer un redemarrage du service lié.


Aller au dela du fichier SLS unique
------------------------------------

Lorsque l'on veut éditer Salt d'une manière évolutive, nous aurons besoin d'utiliser plus d'un fichier |sls|. L'exemple précédent était dans un fichier |sls| unique, mais 2 ou plusieurs fichiers peuvent être combinés pour construire notre "architecture".

L'exemple précédent nous présentait un fichier d'une manière bizarre - ``salt://apache/httpd.conf``. 
Ce fichier a également besoin d'être présent.

Les fichiers |sls| sont énoncés dans une structure de dossier sur le "*Salt Master*"; un |sls| est simplement un fichier et les fichiers à télécharger sont seulement des fichiers.

L'exemple pour apache serait énoncé à la racine du serveur de fichier de |salt| de la manière suivante:

.. code-block:: bash

	apache/init.sls
	apache/httpd.conf


Donc le fichier ``httpd.conf`` est simplement un fichier dans le dossier apache, et est référencé directement.

Mais lorsque nous utilisons plus d'un fichier |sls|, plusieurs composants peuvent être ajoutés à la "boite à outil". Considérons cet exemple pour SSH:

``ssh/init.sls``

.. code-block:: bash

	openssh-client:
	  pkg.installed

	/etc/ssh/ssh_config:
	  file.managed:
	    - user: root
		- group: root
		- mode: 644
		- source: salt://ssh/ssh_config
		- require:
		  - pkg: openssh-client



``ssh/server.sls``

.. code-block:: bash

	include:
	  - ssh

	openssh-server:
	  pkg.installed

	sshd:
	  service.running:
	    - require:
		- pkg: openssh-client
		- pkg: openssh-server
		- file: /etc/ssh/banner
		- file: /etc/ssh/sshd_config

	/etc/ssh/sshd_config:
	  file.managed:
	    - user: root
		- group: root
		- mode: 644
		- source: salt://ssh/sshd_config
		- require:
		  - pkg: openssh-server

	/etc/ssh/banner:
	  file:
	    - managed
		- user: root
		- group: root
		- mode: 644
		- source: salt://ssh/banner
		- require:
		  - pkg: openssh-server

.. note::

	On peut noter les 2 façons différente pour gérer les fichiers dans Salt. 
	Dans le section ``/etc/ssh/sshd_config`` ci-dessus, on utilise la déclaration ``file.managed``, alors que dans la section ``/etc/ssh/banner``, on utilise la déclaration ``file``, et on ajoute l'attribut ``managed`` dans la déclaration d'état. 
	Ces 2 manières différentes amènent la même chose; la première solution - "file.managed" - sera simplement plus rapide.


Maintenant, notre arborescence sera:

.. code-block:: bash

	apache/init.sls
	apache/httpd.conf
	ssh/init.sls
	ssh/server.sls
	ssh/banner
	ssh/ssh_config
	ssh/sshd_config


Cet exemple introduit la déclaration ``include``.
Cette déclaration ``include`` inclue un autre fichier |sls| de sorte que les composants trouvés à l’intérieur peuvent être requis (``require``), surveillés (``watch``)ou comme nous le verrons plus tard, enrichis (``extended``).

La déclaration ``include`` permet aux états d'être croisés.
Lorsqu'un fichier |sls| à une déclaration ``include``, elle sera littéralement étendue pour inclure le contenu des fichiers |sls| inclus

A noter que certains fichiers SLS sont nommés "init.sls", tandis que d'autres ne le sont pas. Plus d'info sur le pourquoi du comment peut être trouvé dans le paragraphe :ref:`Mon premier fichier SLS <ajout_profondeur>`.


Extension et inclusion des données SLS
---------------------------------------

Parfois les données |sls| ont besoins d'être étendues.
Peut-être que le service apache de vérifier des ressources additionnelles, ou sous certaines circonstances un fichier différent aurait besoin d'être inséré.

Dans ces exemples, le premier va ajouter une bannière customisé à SSH et le second va ajouter plusieurs "watcher" (observateurs) à apache pour inclure le ``mod_python``.

``ssh/custom-server.sls``

.. code-block:: yaml

	include:
	  - ssh.server

	extend:
	  /etc/ssh/banner:
	    file:
	      - source: salt://ssh/custom-banner


``python/mod_python.sls``

.. code-block:: yaml

	include:
	  - apache

	extend:
	  apache:
	    service:
		  - watch:
		  - pkg: mod_python

	mod_python:
	  pkg.installed


Le fichier ``custom-server.sls`` utilise la déclaration pour réécrire l'emplacement du fichier de la bannière à télécharger, et donc changer le fichier qui sera utiliser pour configurer la bannière.

Dans le nouveau ``mod_python`` |sls| le paquet "mod_python" est ajouté, mais plus important, le service apache est étendu pour, aussi, vérifier le paquet "mod_python".

.. note::

	La déclaration ``extend`` fonctionne différemment pour ``require`` ou ``watch``. 
	Il ajoute, à la place de remplacer le composant requis.

