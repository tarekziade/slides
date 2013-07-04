+++++++
Vaurien
+++++++

----

.. image:: monkey.png

.. class:: center

    **Break all things**

    Tarek Ziade - tarek@mozilla.com - @tarek_ziade


----

In theory
---------

Natural web server cycle:

1. dev server
2. stage server - load testing, QA etc
3. production server - everything's smooth


----

In practice
-----------

Something will break in the TCP stack in production at some point no matter what.

.. image:: atomic.gif

----

Real life example 1/2
---------------------

.. code-block:: python


   socket.send(data)


*Applications are responsible for checking that all data has been
sent; if only some of the data was transmitted, the application needs to
attempt delivery of the remaining data.*

.. image:: facepalm.gif


----

Real life example 2/2
---------------------


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
- **Too many Chianti**

----

Testing  the I/O is often ignored
---------------------------------

----

How can we improve things ?
---------------------------

.. image:: confused.gif

----

Manual testing is tedious
-------------------------

----

Vaurien
-------

**TCP failures simulation tool**

----

Vaurien Design
--------------


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

Config file
-----------

.. code-block:: ini

    [vaurien]
    backend = google.com:80
    proxy = localhost:8000
    protocol = http
    behavior = 20:delay

    [behavior:delay]
    sleep = 2


And then:

.. code-block:: bash

   $ vaurien --config vaurien.ini


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

            with client.with_behavior('error', \\**options):
                # do something...
                pass

            # we're back to normal here


----

Writing a behavior
------------------


A simple class:

.. code-block:: python

    from vaurien.behaviors.dummy import Dummy
    import time

    class Delay(Dummy):
        name = 'delay'
        options = {'sleep': ("Delay in seconds", int, 1)}
        options.update(Dummy.options)

        def on_before_handle(self, protocol, source, dest, to_backend):
            time.sleep(self.option('sleep'))
            return True

        def on_after_handle(self, protocol, source, dest, to_backend):
            pass

----

Writing a protocol
------------------

.. code-block:: python

    from vaurien.protocols.base import BaseProtocol


    class MinitelProtocol(TCP):
        name = 'minitel'

        def _handle(self, source, dest, to_backend):
            # source = source socket
            # dest = destination socket
            # to_backend = direction
            ... implement a protocol ...

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

