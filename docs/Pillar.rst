*******************************
Pas à pas sur les Pillars
*******************************

.. |state| replace:: **State**
.. |states| replace:: **States**
.. |pillar| replace:: **Pillar**
.. |grains| replace:: **Grains**
.. |pillars| replace:: **Pillars**
.. |minions| replace:: ``Minions``
.. |minion| replace:: ``Minion``
.. |sls| replace:: **SLS**
.. |salt| replace:: ``Salt``
.. |topfile| replace:: "*top file*"

Les |pillars| sont des arborescences de données définies sur le **Salt Master** et envoyées aux |minions|.
Ils permettent la confidentialité, l'envoie sécurisé des données ciblées seulement sur le |minion| approprié.



.. note::

	|grains| et |pillars| peuvent amener à confusion, il faut se souvenir que les |grains| sont des données sur un |minion| qui sont stockées ou générées depuis un |minion|.
	C'est pouqruoi les informations tel que l'OS et la CPU se trouvent dans les |grains|.
	|pillar| est une information sur un |minion| ou plusieurs |minions| stockées ou générées sur le **Salt Master**.



Les Données |pillar| sont utiles pour:

**Des données hautement sensibles**:
	L'information transférée depuis |pillar| est garantie être seulement présenté aux |minions| qui sont ciblés, ce qui fait de |pillar| un objet adapté pour gérer des informations sécurisées, tel que des clés cryptographiques et de mots de passe.

**La configuration des |minions|**:
	Les modules |minion| tel que les modules d'éxécution, d'état, et "*returners*" peuvent souvent être configurés depuis des données stockées dans le |pillar|.

**Les variables**:
	Les variables qui ont besoins d'être assignées à des |minions| spécifique ou des groupes de |minions| peuvent être défini dans un |pillar| pour ensuite être accesibles dans une formule |sls| et des fichiers "*template*".

**Des données arbitraires**:
	|pillar| peut contenir une structure de données basique, donc une liste de valeurs, ou un couple clé/valeur peut être défini ce qui le fait plus simple à le répéter dans un groupe de valeurs dans une formule |sls|.


|pillar| est donc un des plus important système lorsque nous utilisons |salt|.
Ce tutoriel est construit pour créer un |pillar| simple et opérationnel en peu de temps puis se plonger dans les capacités de |pillar| et trouver où sont les données.


Configurer |pillar|
----------------------

|pillar| est démarré par défaut avec |salt|.
Pour voir les données |pillar| des |minions|:

.. code-block:: bash

	salt '*' pillar.items


.. note::
	
	Avant la version 0.16.2, cette fonction est nommé ``pillar.data``.
	Ce nom de fonction est toujours supporté pour assurer la comptabilité.


Par défaut le contenu du fichier de configuration maître est chargé dans le |pillar| de tous les |minions|.
Ce qui permet que le fichier de configuration maître soit utilisé pour une configuration globale des |minion|.

|pillar| comprend des fichiers |sls| et un ``top file``, comme l'arborescence des ``states``.
L'emplacement par défaut de |pillar| est: ``/srv/pillar``.


.. note::
	
	L'emplacement de |pillar| peut être configuré depuis l'option ``pillar_roots`` dans le fichier de configuration du maître. Il ne doit pas être dans un sous-répertoire de l'arborescence ``state``.


Pour commencer à configurer le |pillar|, le repertoire ``srv/pillar`` doit être présent:

.. code-block:: bash

	mkdir /srv/pillar


Maintenant créons un |topfile|, en suivant le même format que notre |topfile| utilisé pour les ``states``:

``/srv/pillar/top.sls``:

.. code-block:: yaml

	base:
	  '*':
	  		- data


Ce |topfile| associe le fichier ``data.sls`` à tous les |minions|.
Maintenant, le fichier ``/srv/pillar/data.sls`` doit être rempli:

``/srv/pillar/data.sls``:

.. code-block:: yaml

	info: some data


Pour s'assurer que tous les minions ont les données du nouveau |pillar|, il faut lancer une commande sur eux leur demandant de récupérer leurs |pillars| depuis le *master*:

.. code-block:: bash

	salt '*' saltutil.refresh_pillar


Maintenant que tous les |minions| ont le nouveau |pillar|, on peut le récupérer:

.. code-block:: bash

	salt '*' pillar.items


La clé ``info`` devrait apparaitre dans le retour des données du |pillar|.



Des données un peu plus complexes
----------------------------------

Contrairement aux |states|, |pillar| n'a pas besoin de définir des formules.
Cet exemple ajoute des utilisateurs avec leur UID respectif:

``/srv/pillar/users/init.sls``:

.. code-block:: yaml

	users:
	  thatch: 1000
	  shouse: 1001
	  utahdave: 1002
	  redbeard: 1003

	
.. note::

	La même recherche sur les répertoire que pour les |states| existe pour |pillar|, donc le fichier ``users/init.sls`` peut être référencé en tant que ``users`` dans le fichier |topfile|.


Nous allons devoir modifier le |topfile| pour inclure notre nouveau fichier |sls|:

``/srv/pillar/top.sls``:

.. code-block:: yaml

	base:
	  '*':
	    - data
		-users


Maintenant les données seront accessible aux |minions|.
Pour utiliser les données |pillar| dans un |state|, nous pourrons utiliser *Jinja*:

``/srv/salt/users/init.sls``:

.. code-block:: jinja

	{% for user, uid in pillar.get('users', {}).items() %}
	{{ user }}:
	  user.present:
	    - uid: {{ uid }}
	{% endfor %}


Cette approche permet de définir d'une manière sécurisé les utilisateurs dans un |pillar| pour ensuite appliquer les données de l'utilisateur dans un fichier |sls|.


Paramétrer les |states| avec un |pillar|
-------------------------------------------

Les données d'un |pillar| peuvent accéssibles depuis un fichier |state| pour customiser le comportement de chaque |minion|.
Tous les données d'un |pillar| (et |grains|) qui sont applicables pour chaque |minion| sont remplacées dans le fichier |state| à travers des modèles avant d'être lancées.
Typiquement, il va inclure les répertoires appropriés pour le |minion| et passer sur les |states| qu'il ne doit pas appliquer.

Un exemple simple est de configurer un plan de noms de paquets dans |pillar| pour chaque distribution Linux:


``/srv/pillar/pkg/init.sls``:

.. code-block:: jinja

	pkgs:
	  {% if grains['os_family'] == 'RedHat' %}
	  apache: httpd
	  vim: vim-enhanced
	  {% elif grains['os_family'] == 'Debian' %}
	  apache: apache2
	  vim: vim
	  {% elif grains['os'] == 'Arch' %}
	  apache: apache
	  vim: vim
	  {% endif %}


Le nouveau |sls| ``pkg`` doit maintenant être ajouté à notre |topfile|:

``/srv/pillar/top.sls``:

.. code-block:: yaml

	base:
	  '*':
	    - data
		- users
		- pkg


A présent les |minions| vont se baser automatiquements d'après leur OS dans le |pillar|, donc nous pouvont paramétrer sans risque le fichier |sls|:

``/srv/salt/apache/init.sls``:

.. code-block:: jinja

	apache:
	  pkg.installed:
	    - name: {{ pillar['pkgs']['apache'] }}


Par contre, si aucun |pillar| n'est créé(disponible, référencé), nous pouvons le créer dans le |state| directement:

.. note::

	La fonction ``pillar.get`` utilisé dans cet exemple a été ajouté dans la version 0.14.0


``/srv/salt/apache/init.sls``:

.. code-block:: jinja

	apache:
	  pkg.installed:
	    - name {{ salt['pillar.get']('pkgs:apache', 'httpd') }}


Dans cet exemple précédent, si la valeur du |pillar| ``pillar['pkgs']['apache']`` n'est pas configuré dans le |pillar| du |minion|, alors l'option par défaut ``httpd`` sera utilisé.

.. note::
	
	Sous le capot, |pillar| est simplement un dictionnaire Python, donc les méthodes du dictionnaire Python tel que ``get`` et ``items`` peuvent être utilisées.


PILLAR MAKES SIMPLE STATES GROW EASILY
------------------------------------------

http://docs.saltstack.com/en/latest/topics/tutorials/pillar.html#pillar-makes-simple-states-grow-easily

 
