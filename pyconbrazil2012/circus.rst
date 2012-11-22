++++++++++++++++++++++++++
Circus - Pycon Brazil 2012
++++++++++++++++++++++++++

----

.. image:: circus-medium.png

.. class:: center

    **Process & Socket Manager**

    Tarek Ziade - tarek@mozilla.com - @tarek_ziade


----

Process manager ?
-----------------

- start/stop processes
- monitoring / stats
- automatic respawns


Example -- a Pyramid stack:

- 1 Gunicorn process with 4 Pyramid workers
- 1 Redis process
- 1 Solr process

----

Mozilla Use case
================

**Token Server** - auth key generation server

- encryption work => CPU-bound
- 175 processes / box, lots of boxes :)
- Respawn, flapping
- Web UI with real-time stats
- Stdout/stderr rediractions
- Fine-grained control through the command line


----

Existing tools
==============

- **Supervisord** -- Python, pretty cool, APIs: XML-RPC, good community,
  hard to extend, web UI so-so, no real-time stdout.

- **Bluepill** -- flapping, tedious DSL, less mature

- **upstart** -- system level - root access needed

- **daemontools** low-level like upstart, not very interactive

- god, monit, runit, etc..

----

Missing features
================

- realtime stdout/stderr
- realtime stats
- powerful web console
- remote
- clustering
- scalable


=> Circus project launched !

----

Technical choices
=================

- process management : **psutil**
- Message passing : **ZeroMQ**
- Web console : **socket.io** & **Bottle** & **Gevent**
- Everything else: Python

----

psutil
======

- portable (not Circus)
- speedup fixes (> 0.6.1)
- clean & simple API


.. code-block:: python

   >>> import psutil
   >>> p = psutil.Process(7384)
   >>> p.name
   'Address Book'
   >>> p.create_time
   1346710439.681407
   >>> p.uids
   user(real=501, effective=501, saved=501)


----


ZeroMQ
======

- async library for message passing == *smart* socket
- highly scalable
- transports: ITC, IPC, TCP, PGM (multicast)
- principal patterns

 - request/reply
 - pub/sub
 - push/pull

- used by IPython
- PyZMQ = zmq bind + nice I/O event loop adapted from Tornado


----



Commands
========

- **circusd** - principal daemon
- **circus-top** - top-like command
- **circusctl** - command line

----


Example
=======

.. code-block:: ini

    [circus]
    httpd = 1
    stats_endpoint = tcp://localhost:5557

    [watcher:pyramid]
    cmd = bin/pserve development.ini
    singleton = 1
    working_directory = /var/myapp

    [watcher:redis]
    cmd = /usr/local/bin/redis-server /usr/local/etc/redis.conf
    singleton = 1

    [watcher:retools-workers]
    cmd = /var/myapp/bin/retools-worker main
    numprocesses = 5


Launch

.. code-block:: sh

  $ circusd webapp.ini


----

Circus Architecture
===================

.. image:: circus-architecture.png


----

Go son, deploye yer apps
========================


.. image:: devops.jpg


**DEMO** - http://blog.ziade.org/circus-screencast-1.html


----

Mozilla Use Case #2 - Manage full web stacks
--------------------------------------------


----

**Pb. Current Stack** *2 levels of process managment...*

.. image:: classical-stack.png

----

**Solution** *Socket management within Circus*

.. image:: circus-stack.png


----

Circus sockets
==============

Like Apache or Gunicorn - **pre-fork model**:

- Every process managed by Circus is forked from **circusd**
- **circusd** creates & open sockets
- child processes can *accept()* connection on those sockets
- system-level load balancing


----

Real-world use case: WSGI apps
===============================

- **Chaussette** : WSGI server that reuses already opened sockets
- Launched with the socket file descriptor number
- the socket object is recreated with *socket.fromfd()*
- several backends: gevent, meinheld, waitress, wsgiref, eventlet

http://chaussette.readthedocs.org

----

Example

.. code-block:: ini

    [circus]
    ...

    [watcher:web]
    cmd = chaussette --fd $(circus.sockets.web) --backend meinheld mycool.app
    use_sockets = True
    numprocesses = 5

    [socket:web]
    host = 0.0.0.0
    port = 8000


----

**Demo #2 : web stack**

**DEMO** - http://blog.ziade.org/circus-screencast-2.html

----

Benchmarks
==========

Faster to slowest:

- Circus + fastgevent
- Circus + gevent
- Circus + meinheld
- Gunicorn + gevent
- Circus + waitress

c.f. http://tinyurl.com/cykvgmo

----

Features being added
====================

- Clustering
- stderr/stdout streaming in the web dashboard
- ...

----

Thanks !
========

Questions ?

- Docs: http://circus.io
- IRC: #mozilla-circus on Freenode
- ML : http://tech.groups.yahoo.com/group/circus-dev
- Code: https://github.com/mozilla-services/circus

