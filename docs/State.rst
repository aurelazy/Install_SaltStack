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
Les ``require`` *statements* vont s'assurer que l'utilisateur va être créé après le groupe, et que le groupe ne sera créé qu'après l'installation de paquet ``Apache``.

Ensuite, ``require`` *statement* sous *service* a été changé par ``watch``, et il vérifie maintenant 3 |state| au lieu d'un seul. 
Le *statement* ``watch`` fait la même chose que ``require``, il s'assure que les autres |state| soient démarrés avant le |state| ``watch``, mais il ajoute un composant supplémentaire.
Le *statement* ``watch`` va lancer le *state watcher function* pour tout changement effectué sur le *watched state*.
Donc si le paquet a été updaté, le fichier de configuration changé, ou l'UID modifié, alors le *service state watcher* sera lancé.
Le *service state watcher* va relancer le service, donc dans ce cas, un changement dans la configurationva également amorcer un redemarrage des services respectifs.
