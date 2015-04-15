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


	
