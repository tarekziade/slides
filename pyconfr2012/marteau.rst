+++++++++++++++++++++++
Marteau - Pycon Fr 2012
+++++++++++++++++++++++

----

.. image:: marteau.png


.. class:: center

    **Marteau -- Distributed Load testing**

    Tarek Ziade - tarek@ziade.org - @tarek_ziade


----

Funkload
========

- unit/load testing tool
- UnitTest-based
- gplot-based reports
- distributed tests via SSH
- mature

----

Funkload limitations
====================

- distributed tests: no nodes management
- no queue
- no reports management

----

Marteau
=======

- redis-based Queue to store load jobs
- web UI : interaction, node management, reports

----

**Demo**

----

Mozilla's Marteau
-----------------

- 25 nodes, up to 10,000 concurrent users
- shared by several projects


----

Community instance ?
====================

-

----

Thanks !
========

Questions ?

- Docs: http://marteau.rtfd.org
- Repo: https://github.com/mozilla-services/marteau

