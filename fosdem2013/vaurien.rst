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


.. image:: design.png

----

Protocol & behaviors
--------------------

.. image:: protocol.png


----

Built-in protocols & behaviors
------------------------------


- protocols: http, memcache, mysql, redis, smtp, generic tcp
- behaviors: blackout, delay, dummy, error, hang

Create your own.

----

Command line
------------


An SSL SMTP proxy with a 5% error rate and 10% delays

.. code-block:: bash

   $ pip install vaurien
   $ vaurien --proxy 0.0.0.0:6565 --backend mail.example.com:465 \
             --protocol smtp --behavior 5:error,10:delay


MySQL with 5% hangs :)

.. code-block:: bash

   $ vaurien --proxy 0.0.0.0:3307 --backend localhost:3306
             --protocol mysql --behavior 5:hang


----

Unit tests
----------

.. code-block:: python

    import unittest
    from vaurien import Client, start_proxy, stop_proxy


    class MyTest(unittest.TestCase):

        def setUp(self):
            self.proxy_pid = start_proxy(port=8080)

        def tearDown(self):
            stop_proxy(self.proxy_pid)

        def test_one(self):
            client = Client()
            options = {'inject': True}

            with client.with_behavior('error', \**options):
                # do something...
                pass

            # we're back to normal here


----

Used for Firefox Marketplace
----------------------------

Read http://tinyurl.com/marketplace-test

----

Thanks !
--------

Questions ?

- Docs: http://vaurien.rtfd.org
- Code: https://github.com/mozilla-services/vaurien

