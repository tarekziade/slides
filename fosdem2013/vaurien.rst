+++++++
Vaurien
+++++++

----

.. image:: monkey.png

.. class:: center

    **Break all things**

    Tarek Ziade - tarek@mozilla.com - @tarek_ziade


----

Firefox Sync Release
--------------------

----

Epic Fail!
----------

.. image:: atomic.gif

----

socket programming 101
----------------------

.. code-block:: python


   socket.send(data)


*Applications are responsible for checking that all data has been
sent; if only some of the data was transmitted, the application needs to
attempt delivery of the remaining data.*

.. image:: facepalm.gif


----

socket programming 101
----------------------

**The right pattern:**

.. code-block:: python

    total = 0
    while total < len(data):
        sent = self.sock.send(data[total:])
        if sent == 0:
            raise RuntimeError("socket connection broken")
        total += sent

**Or simply:**

.. code-block:: python


   socket.sendall(data)


----

Other usual failures
--------------------

- **Lost connection to MySQL server**
- **Timeout Error**
- **Too many connections**
- **Too many open files**
- **Too many Chimay**

----

How can we improve things ?
---------------------------

.. image:: confused.gif

----

Manual testing is not an option
-------------------------------

----

Vaurien
-------

1. proxy between your application & backend
2. simulates failures
3. design: protocols & behaviors
4. many built-in protocols & behaviors
5. test-driven or command-line driven
6. pluggable!

----

Proxy
-----

XXX

----

Simulates failures
-------------------

XXX

----

Design
------

XXX

----


Built-in
--------

* protocols: tcp, xxx
* behaviors: xxxx

----

Vaurien & Marketplace
----------------------

XXX

See https://blog.mozilla.org/webdev/2013/01/18/load-testing-the-marketplace/

----

Thanks !
========

Questions ?

- Docs: http://vaurien.rtfd.org
- Code: https://github.com/mozilla-services/vaurien

