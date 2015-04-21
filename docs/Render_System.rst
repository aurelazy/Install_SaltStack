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



Commençont à apprendre le moteur par défaut - ``yaml_jinja``
--------------------------------------------------------------

Le moteur par défaut- ``yaml_jinja``, permet d'utiliser le sytème de modélisation jinja.
Un guide sur jinja est disponible `ici <http://jinja.pocoo.org/docs>`_.

Lorsque nous travaillons avec un moteur de rendu quelques bits de données utiles sont écrites.
Dans le cas d'un moteur de modèlisation basé sur un modèle de rendu (?), trois composants critiques sont disponible, ``salt``, ``grains``, et ``pillar``.
L'objet ``salt`` permet à n'importe quel fonction |salt| d'appeler depuis un modèle, et un ``grains`` permet aux ``Grains`` d'être accessible depuis un modèle.
Quelques exemples:


``apache/init.sls``:

.. code-block:: yaml

	apache:
	  pkg.installed:
	    {% if grains['os'] == 'RedHat' %}
		- name: httpd
		{% endif %}
	  service.running:
	    {% if grains['os'] == 'RedHat' %}
		- name: httpd
	    {% endif %}	
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


Cet exemple est simple.
Si le grain ``os`` voit que le systèmed'exploitation est un RedHat, alors le nom du paquet Apache ainsi que son service sera ``httpd``.

Voici une façon plus aggressive d'utiliser Jinja, en paramétrant un module "MooseFS distributed filesystem chunkserver":

``mossefs/chunk.sls``:

.. code-block:: yaml

	include:
	  - moosefs

	{% for mnt in salt['cmd.run']('ls /dev/data/moose*').split() %}
	/mnt/moose{{ mnt[-1] }}:
	  mount.mounted:
	    - device: {{ mnt }}
		- fstype: xfs
		- mkmnt: True
	  file.directory:
	    - user: mfs
		- group: mfs
		- require:
		  - user: mfs
		  - group: mfs
	{% endfor %}


	/etc/mfshdd.cfg:
	  file.managed:
	    - source: salt://mossefs/mfshdd.cfg
		- user: root
		- group: root
		- mode: 644
		- template: jinja
		- require:
		  - pkg: mfs-chunkserver

	/etc/mfschunkserver.cfg:
	  file.managed:
	    - source: salt://mossefs/mfschunkserver.cfg
		- user: root
		- group: root
		- mode: 644
		- template: jinja
		- require:
		  - pkg: mfs-chunkserver

	mfs-chunkserver:
	  pkg.installed: []
	mfschunkserver:
	  service.running:
	    - require:
	{% for mnt in salt['cmd.run']('ls /dev/data/moose*') %}
	      - mount: /mnt/moose{{ mnt[-1] }}
		  - file: /mnt/moose{{ mnt[-1] }}
	{% endfor %}
		  - file: /etc/mfschunkserver.cfg
		  - file: /etc/mfshdd.cfg
		  - file: /var/lib/mfs


A CONTINUER
