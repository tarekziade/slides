Montreal Python
+++++++++++++++

----

Molotov - Load testing webservices
==================================

----


Mozilla QA requirements
-----------------------

- A tool to stress webservices
- Easy to deploy and run in our CI/CD
- Low entry barrier


----

Existing tools
--------------

- Apache Bench (ab)
- Boom!
- Apache JMeter (Java + XML)
- Locust (gevent)
- ...many more

----

Previous failed attempt
-----------------------

What I've learned from writing **Loads**:

- client-side metrics are too noisy
- distributed systems are hard, rely on existing infra
- KISS
- a load test is also functional test

----

Molotov design principles
-------------------------

- asyncio-based
- As close as possible to vanilla Python
- focuses on simplicity
- scope: one single script on one single machine.


----

Hello World
-----------

**Simplest scenario**

.. code-block:: python

    from molotov import scenario

    _API = "http://example.com"

    @scenario()
    async def my_scenario_one(session):
        async with session.get(_API) as resp:
            res = await resp.json()
            assert res["result"] == "OK"
            assert resp.status == 200

----

2 scenarios
-----------

**90% of reads, 10% of writes**

.. code-block:: python

    from molotov import scenario

    _API = "http://example.com"

    @scenario(weight=90)
    async def my_scenario_one(session):
        async with session.get(_API) as resp:
            res = await resp.json()
            assert res["result"] == "OK"
            assert resp.status == 200

    @scenario(weight=10)
    async def scenario_two(session):
        somedata = json.dumps({"OK": 1})
        async with session.post(_API, data=somedata) as resp:
            assert resp.status == 200

----

Run it
------


.. code-block:: bash

    # single run
    $ molotov --single-run script.py
    **** Molotov v2.0. Happy breaking! ****
    Preparing 1 workers...OK
    SUCCESSES: 1 | FAILURES: 0  | WORKERS: 1
    *** Bye ***

    # 150 concurrent workers
    $ molotov -w 150 --duration 60 script.py
    **** Molotov v2.0. Happy breaking! ****
    Preparing 150 workers...OK
    SUCCESSES: 110 | FAILURES: 0 | WORKERS: 150
    *** Bye ***

    # 4 processes, 10 workers each, 10 seconds
    $ molotov -w 10 -p 4 -d 10 loadtest.py
    **** Molotov v2.0. Happy breaking! ****
    Forking 4 processes
    [44553] Preparing 10 workers...OK
    [44554] Preparing 10 workers...OK
    [44555] Preparing 10 workers...OK
    [44556] Preparing 10 workers...OK
    SUCCESSES: 80 | FAILURES: 0
    *** Bye ***

    # just run it until it starts to fail
    $ molotov -x script.py
    **** Molotov v2.0. Happy breaking! ****
    Preparing 500 workers...OK
    SUCCESSES: 110 | FAILURES: 0 | WORKERS: 340
    *** Bye ***


----

Advanced usage : sizing
-----------------------

Ramp up until an error rate is reached.

.. code-block:: bash

    $ molotov --sizing script.py
    **** Molotov v2.0. Happy breaking! ****
    Preparing 500 workers...
    OK
    SUCCESSES: 1064 | FAILURES: 11 | WORKERS: 268
    Sizing is over!

    Error Ratio 5.24 % obtained with 268 workers.

    OVERALL: SUCCESSES: 1288 | FAILURES: 56
    LAST MINUTE: SUCCESSES: 453 | FAILURES: 56

    *** Bye ***

----

Automation with Moloslave 1/2
-----------------------------

**Automating runs from github**

A github repo with:

- your molotov scripts
- add a **molotov.json** that lists them + rurnning options

Then run it using **moloslave**.

.. code-block:: bash

    $ moloslave https://github.com/loads/molotov big


----

Automation with Moloslave 2/2
-----------------------------

molotov.json example:

.. code-block:: javascript

    {
      "molotov": {
          "env": {
              "SERVER_URL": "http://aserver.net"
          },
          "requirements": "requirements.txt",
          "tests": {
              "big": {
                  "console": true,
                  "duration": 10,
                  "exception": true,
                  "processes": 10,
                  "scenario": "molotov/tests/example.py",
                  "workers": 100
              },
              "test": {
                  "console": true,
                  "duration": 1,
                  "exception": true,
                  "verbose": 1,
                  "console_update": 0,
                  "scenario": "molotov/tests/example.py"
              }
            }
        }
    }


----


Docker
------

- Generic docker image to run **moloslave**
- Can be used for massive distributed test (Kubernetes, Amazon ECS)

Example of call::

    $ docker run -i --rm \
        -e TEST_REPO=https://github.com/loads/molotov \
        -e TEST_NAME=test tarekziade/molotov:latest

This will start docker and run **moloslave** directly.

From there:

- orchestrate Molotov nodes
- perform load testing
- collect Metrics (datadog, graylog etc)


----


Advanced scripting : Events
---------------------------

.. code-block:: python

    import molotov

    @molotov.events()
    async def print_request(event, **info):
        if event == "sending_request":
            print("=>")

    @molotov.events()
    async def print_response(event, **info):
        if event == "response_received":
            print("<=")

    @molotov.scenario(100)
    async def scenario_one(session):
        async with session.get("http://localhost:8080") as resp:
            res = await resp.json()
            assert res["result"] == "OK"
            assert resp.status == 200

----

Conclusion
----------

- Simple, yet powerful load testing tool
- Very close to vanilla Python
- Easy to use locally
- Easy to integrate into any CI/CD

Full doc: https://molotov.readthedocs.io/en/stable/

tarek@ziade.org

Thanks! Questions?


