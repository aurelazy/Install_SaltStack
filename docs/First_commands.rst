*****************************
Mes premières commandes
*****************************

.. |minion| replace:: ``Minion``
.. |minions| replace:: ``Minions``
.. |master| replace:: ``Master``
.. |salt| replace:: **Salt**

La communication entre le |master| et le |minion| peut être vérifié en lançant la commande ``test.ping``:

.. code-block:: bash

	[root@master ~]# salt alpha test.ping
	alpha: True

La communications entre le |master| et tous les |minions| peut etre testé d'une façon similaire:

.. code-block:: bash

	[root@master ~]# salt '*' test.ping
	alpha: True
	bravo: True
	charlie: True
	delta: True

Chacun des |minion| devrait envoyer une réponse ``True`` comme le montre l'exemple précédent.

Par exemple, la commande suivante doit retourner l'usage du disque de tous nos |minions|:

.. code-block:: bash

	[root@master ~]# salt '*' disk.usage

