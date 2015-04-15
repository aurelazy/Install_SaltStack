**************
Installation
**************

.. |minion| replace:: ``Minion``
.. |master| replace:: ``Master``

Ce tuto va montrer comment installer Saltstack sur un serveur et un client Debian.

Pour wheezy, les lignes suivantes doivent être présente dans ``/etc/apt/sources.list`` ou dans un fichier dans ``/etc/apt/sources.list.d``:

.. code-block:: bash


	deb http://debian.saltstack.com/debian wheezy-saltstack main

Ensuite on importe le clé pour le dépot.
Nous aurons besoin de celle utilisé lors de la signature du paquet.

.. code-block:: bash

	wget -q -O- "http://debian.saltstack.com/debian-salt-team-joehealy.gpg.key" | apt-key add -

On fait un ``Update`` sur notre BDD de notre dépot:

.. code-block:: bash

	apt-get update

Installation des paquets
-------------------------

On instal le ``Salt master``, ``minion`` ou ``syndic`` depuis le dépot avec la commande ``apt-get``. Chacun de ces exemples montrent l'installation d'un seul ``daemon``, mais nous pouvons également le faire en une seul fois avec la même commande: 

Sur le serveur:
	
.. code-block:: bash

	apt-get install salt-master

Sur le client:

.. code-block:: bash

	apt-get install salt-minion

.. note::

	Un autre paquet est installé, mais je ne connait pas encore l'utilité, ``salt-syndic``

Configuration de Salt
----------------------

La configuration de Salt est très simple. La configuration par défaut du ``master`` marchera pour la plupart des installations et la seul exigence pour la mise en place d'un ``minion`` est de lui fixer l'adresse IP du ``master`` dans son fichier de configuration.

Le fichier de configuration sera dans le dossier ``/etc/salt`` et sera nommé ``/etc/salt/master``, et ``/etc/salt/minion``.



Master Configuration
~~~~~~~~~~~~~~~~~~~~~

Par défaut le ``Salt master`` écoute sur les ports 4505 et 4506 sur toutes les interfaces (0.0.0.0). Pour lier Salt à une IP spécifique, il faut redefinir l'instruction ``interface`` dans le fichier de configuration du ``master``, typiquement ``/etc/salt/master``, comme suit:

.. code-block:: bash

	- #interface: 0.0.0.0
	+ interface: 192.168.0.25

Après avoir réactualiser le fichier de configuration, il faut redémarrer le ``Salt master``


Configuration du minion
~~~~~~~~~~~~~~~~~~~~~~~~~

Bien qu'il y ai beaucoup d'options de configuration pour les ``Salt minion``, les configurer est très simple. Par défaut, un ``Salt minion`` va essayé de se connecter au nom DNS ``salt``; si le ``minion`` est capable de résoudre le nom correctement, auncune configuration n'est requise.

Si le nom DNS ``salt`` ne peut pas être résolu pour pointer sur le ``master``, il faudra redefinir l'instruction ``master`` dans le fichier de configuration du ``minion``, typiquement ``/etc/salt/minion``, comme suit:

.. code-block:: bash

	- #master: salt
	+ master: 192.168.0.25

Après avoir réactualiser le fichier de configuration, il faut redémarrer le ``Salt master``

Démarrer Salt
--------------

Démarrer le ``master`` en arrière plan (pour *daemonizer* le processus, ajouter l'option ``-d``):

.. code-block:: bash

	salt-master

Démarrer le ``minion`` en arrière plan (pour *daemonizer* le processus, ajouter l'option ``-d``):

.. code-block:: bash

	salt-minion

Quelques soucis ?
---------------------

La manière la plus simple pour dépanner **Salt** est des lancer le ``master`` et le ``minion`` en arrière plan avec un niveau de log fixé à ``debug``: 

.. code-block:: bash

	salt-master --log-level=debug

Au premier lancement du minion, un message d'erreur apparait:

.. code-block:: bash

	[ERROR ] The Salt Master has cached the public key for this node, this salt minion will wait for 10 seconds before attempting to re-authenticate

Il faut donc lancer la commande suivante sur le serveur:

.. code-block:: bash

	salt-key -a <nom_DNS_minion>


``nom_DNS_minion`` est le nom du minion


Gestion des clés
~~~~~~~~~~~~~~~~~~

**Salt** utilise un cryptage AES pour toutes les communications entre le |master| et le |minion|. Ce qui permet de s'assurer que les commandes envoyées aux |minion| ne peuvent être falsifiées, et la communication entre le |master| et le |minion| est authentifiée avec une clé de confiance et accéptée.

Avant que les commandes puissent être envoyées au |minion|, sa clé doit être accépté par le |master|. Lancer la commande ``salt-key`` pour lister les clés connus sur le |master|:

.. code-block:: bash

	[root@master ~]# salt-key -L
	Unaccepted Keys:alpha 
		bravo 
		charlie 
		delta
	Accepted Keys:

Cet exemple montre que le |master| est conscient qu'il y a 4 |minion|, mais aucunes des clé n'a été accépté. Pour accépter les clés et permettre aux |minion| d'être controllé par le |master|, on va utiliser encore une fois la commande ``salt-key``:

.. code-block:: bash

	[root@master ~]# salt-key -A
	[root@master ~]# salt-key -L
	Unaccepted Keys:
	Accepted Keys:alpha
		bravo
		charlie
		delta

La commande ``salt-key`` permet de signer les clés individuellement ou en lot. L'exemple ci-dessus, qui utilise l'option ``-A`` permet d'accepter toutes les clés en attente. Pour accepter chaque clé individuellement, il faudra utiliser la même option mais en minuscule, ``-a``.

