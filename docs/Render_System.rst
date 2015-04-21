***********************************
Comprendre le système de rendu
***********************************

.. |minion| replace:: ``Minion``
.. |master| replace:: ``Master``
.. |minions| replace:: ``Minions``
.. |salt| replace:: **Salt**
.. |state| replace:: **state**
.. |sls| replace:: *SLS*
.. |id| replace:: ``ID``


Section à faire, voir `la documentation Salt <http://salt.readthedocs.org/en/latest/topics/tutorials/starting_states.html#understanding-the-render-system>`_

Comme les données |sls| sont simplement des données, elles n'ont pas nécécessairement besoin d'être écrite en langage YAML. 
|salt| écrit par défaut en YAML, parcque que ce langage est simple et facile à apprendre et à écrire.
Mais les fichier |sls| peuvent être rendu depuis n'importre quel moyen, du moment que le module est installé.

Le système de rendu par défaut est ``yaml_jinja``.
``yaml_jinja`` va d'abord passer le modèle à travers le système de modèle ``Jinja2``, puis à travers l'analyseur YAML.
Les benefices sont que la condtruction du programme sont accessibles lorsque l'on crée les fichiers |sls|.

Les autres moteur de rendu que l'on peut utiliser sont ``yaml_mako`` et ``yaml_wempy`` qui utilisent respectivement le ``Mako`` ou ``Wempy`` système de modèle à la place de ``Jinja`` et plus particulièrement, Python ou ``py``, ``pyds1`` et ``pyobjects``.
Le moteur de rendu ``py`` accepte Python pour écrire les fichier |sls| pour un maximum de flexibilité et puissance lorsque l'on écrit les données |sls|; 
quand le moteur de rendu ``pydsl`` amène la flexibilité, et un langage pour un domaine spécifique lors de la création des fichier |sls|;
et le moteur de rendu ``pyobjects`` apporte une interface ``Pythonic`` pour construire des données |state|.


.. note::

	Les moteurs de modèle décrient au-dessus ne sont pas uniqueent utilisable dans les fichier |sls|.
	Ils peuvent être utilisé dans les |state| ``file.manged``, ce qui fait la configuration des fichiers bien plus dynamiques et flexibles.
	Quelques exemples de l'utilisation de modèles dans les fichiers de configurationpeuvent être trouvés dans la documentation des :ref:`fichiers |state| <ref_salt.state.file>`.


