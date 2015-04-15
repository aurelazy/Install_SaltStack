*************************************
Apprendre à connaitre les fonctions
*************************************

.. |minion| replace:: ``Minion``
.. |master| replace:: ``Master``
.. |minions| replace:: ``Minions``
.. |salt| replace:: **Salt**


|salt| est installé avec une grande librairie de fonctions à éxécuter, et les fonctions |salt| sont auto-documentées. Pour voir quelles fonctions sont disponibles sur les |minions| il suffit d'éxécuter la fonction ``sys.doc``:

.. code-block:: bash

	[root@master ~] salt '*' sys.doc


Cela va afficher une très grande liste des fonctions disponibles et leur documentation.


Les fonctions utiles à connaitre
----------------------------------

Le module ``cmd`` contient des fonctions qui vont être lancées sur les |minions|, tel que ``cmd.run`` et ``cmd.run_all``:

.. code-block:: bash

    [root@master ~] salt '*' cmd.run 'ls -l /etc'


La fonction ``pkg`` récupère automatiquement le système de paquet local sur les |minions|. Cela veut dire que ``pkg.install`` va installer un paquet depuis ``yum`` sur les système Red Hat, ``apt`` sur Debian, etc, ... :


.. code-block:: bash

    [root@master ~] salt '*' pkg.install vim


Nous allons voir comment installer un serveur Web, puis lancer le service:


.. code-block:: bash

    [root@master ~] salt '*' pkg.install nginx
    [root@master ~] salt '*' service.start nginx


La fonction ``network.interfaces`` va lister toutes les interfaces sur un |minion|, avec leur adresse IP, le netmask, la MAC adresse, etc ...:


.. code-block:: bash

    [root@master ~] salt '*' network.interfaces


Changement du format de sortie
---------------------------------

Le format de sortie par defaut utilisé par les commandes |salt| est appelé *nestedoutputter*, mais il existe plusieurs autres *outputter* qui peuvent être utilisés pour changer les messages de sortie. Par exemple, l'*outputter* ``pprint`` peut être utilisé pour afficher les données retournées en utilisant le module Python ``pprint``:

.. code-block:: bash

    [root@master ~] salt <myminion> grains.item pythonpath --out=pprint

