===============
Troubleshooting
===============

``pyngrok`` is a Python wrapper for `ngrok <https://ngrok.com/>`_, so often errors that occur (especially during
startup) are a result of a ``ngrok`` configuration error and not a bug in ``pyngrok``. Hopefully this page can
give you some useful tips to debug these issues.

Test on the Command Line
------------------------

When you install ``pyngrok`` with ``pip install pyngrok``, ``ngrok`` should available from the command
line. First ensure this is true by checking to see if ``pyngrok``'s version of ``ngrok`` is properly setup in
your path. Running ``ngrok`` with no args from the command line should show ``pyngrok`` version at the very
end.

.. code-block:: shell

    bash-3.2$ ngrok
    NAME:
       ngrok - tunnel local ports to public URLs and inspect traffic

    ...

    PYNGROK VERSION:
       4.0.0

.. note::

    If ``PYNGROK VERSION`` is not seen in the output here, something else is managing ``ngrok`` (perhaps
    another ``ngrok`` wrapper installed through `Homebrew <https://brew.sh/>`_ or `npm <https://www.npmjs.com/>`_).
    If you'd prefer ``pyngrok`` manage ``ngrok`` for you, you'll first need to
    `reorder things in your $PATH <https://stackoverflow.com/a/32170849/1128413>`_ to fix this, then you can continue
    troubleshooting on the command line.

With ``PYNGROK VERSION`` shown in your output here, you know things are setup properly. Next try starting
``ngrok`` headless:

.. code-block:: shell

    bash-3.2$ ngrok start --none --log stdout

If that works, try starting a simple HTTP tunnel:

.. code-block:: shell

    bash-3.2$ ngrok http 5000 --log stdout

If neither of these work, the logs should be dumped to the console for you to troubleshoot ``ngrok``
directly. If both of these work, you know ``pyngrok`` is properly installed on your system and able to access
the ``ngrok`` binary, meaning the problem is likely a configuration issue in your Python application.

Enable Logging to the Console
-----------------------------

Printing logs to the console can be a quick way to debug common issues by surfacing their root cause. To do this,
ensure you have a handler streaming logs and your level is set to ``DEBUG``. Here is a simple example:

.. code-block:: python

    import logging

    from pyngrok import ngrok

    # Setup a logger
    logger = logging.getLogger()
    logger.setLevel(logging.DEBUG)
    handler = logging.StreamHandler()
    handler.setFormatter(logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s"))
    logger.addHandler(handler)

    # Then call the pyngrok method throwing the error, for example
    ngrok.connect(5000)


Programmatically Inspect the Logs
---------------------------------

``ngrok`` logs are parsed by the :class:`~pyngrok.process.NgrokProcess`, and you can inspect them by iterating over
its ``logs`` variable or giving it a `log_event_callback <index.html#event-logs>`_.

If you're seeing the :class:`~pyngrok.process.NgrokProcess` fail with a :class:`~pyngrok.exception.PyngrokNgrokError`
exception, these logs are also available on the exception itself. Catch the exception and inspect ``ngrok_logs``
and ``ngrok_error`` for more insight in to where ``ngrok`` is failing.

Test in the Python Console
--------------------------

Try to execute the same code that is giving you an error from the Python console instead. Be sure to pair this with
enabling logging (as illustrated in the section above) so you can see where things are going wrong.

.. code-block:: shell

    bash-3.2$ python
    Python 3.7.6 (default, Dec 30 2019, 19:38:28)
    [Clang 11.0.0 (clang-1100.0.33.16)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import logging
    >>> from pyngrok import ngrok
    >>> logger = logging.getLogger()
    >>> logger.setLevel(logging.DEBUG)
    >>> handler = logging.StreamHandler()
    >>> handler.setFormatter(logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s"))
    >>> logger.addHandler(handler)
    >>> ngrok.connect()
    2020-10-11 14:50:38,612 - pyngrok.process - INFO - ngrok process starting: 77371
    2020-10-11 14:50:38,762 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:38-0700 lvl=info msg="no configuration paths supplied"
    2020-10-11 14:50:38,763 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:38-0700 lvl=info msg="using configuration at default config path" path=/Users/<username>/.ngrok2/ngrok.yml
    2020-10-11 14:50:38,763 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:38-0700 lvl=info msg="open config file" path=/Users/<username>/.ngrok2/ngrok.yml err=nil
    2020-10-11 14:50:38,773 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:38-0700 lvl=info msg="starting web service" obj=web addr=127.0.0.1:4040
    2020-10-11 14:50:39,191 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:39-0700 lvl=info msg="tunnel session started" obj=tunnels.session
    2020-10-11 14:50:39,192 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:39-0700 lvl=info msg="client session established" obj=csess id=27a50ddb9b03
    2020-10-11 14:50:39,215 - pyngrok.process - INFO - ngrok process has started: http://127.0.0.1:4040
    2020-10-11 14:50:39,217 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:39-0700 lvl=info msg=start pg=/api/tunnels id=40b25b366abdb68c
    2020-10-11 14:50:39,217 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:39-0700 lvl=info msg=end pg=/api/tunnels id=40b25b366abdb68c status=200 dur=455.974µs
    2020-10-11 14:50:39,219 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:39-0700 lvl=info msg=start pg=/api/tunnels id=1480f919bba94f98
    2020-10-11 14:50:39,220 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:39-0700 lvl=info msg=end pg=/api/tunnels id=1480f919bba94f98 status=200 dur=181.211µs
    2020-10-11 14:50:39,221 - pyngrok.ngrok - DEBUG - Connecting tunnel with options: {'name': 'http-80-6f08c18a-9986-4c6d-b565-049289b60210', 'addr': '80', 'proto': 'http'}
    2020-10-11 14:50:39,221 - pyngrok.ngrok - DEBUG - Making POST request to http://127.0.0.1:4040/api/tunnels with data: b'{"name": "http-80-6f08c18a-9986-4c6d-b565-049289b60210", "addr": "80", "proto": "http"}'
    2020-10-11 14:50:39,222 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:39-0700 lvl=info msg=start pg=/api/tunnels id=b6274e2ae38a6b7e
    2020-10-11 14:50:39,422 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:39-0700 lvl=info msg="started tunnel" obj=tunnels name="http-80-6f08c18a-9986-4c6d-b565-049289b60210 (http)" addr=http://localhost:80 url=http://<public_sub>.ngrok.io
    2020-10-11 14:50:39,423 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:39-0700 lvl=info msg="started tunnel" obj=tunnels name=http-80-6f08c18a-9986-4c6d-b565-049289b60210 addr=http://localhost:80 url=https://<public_sub>.ngrok.io
    2020-10-11 14:50:39,423 - pyngrok.process.ngrok - INFO - t=2020-10-11T14:50:39-0700 lvl=info msg=end pg=/api/tunnels id=b6274e2ae38a6b7e status=201 dur=199.017682ms
    2020-10-11 14:50:39,424 - pyngrok.ngrok - DEBUG - Response status code: 201
    2020-10-11 14:50:39,424 - pyngrok.ngrok - DEBUG - Response: {"name":"http-80-6f08c18a-9986-4c6d-b565-049289b60210","uri":"/api/tunnels/http-80-6f08c18a-9986-4c6d-b565-049289b60210","public_url":"https://<public_sub>.ngrok.io","proto":"https","config":{"addr":"http://localhost:80","inspect":true},"metrics":{"conns":{"count":0,"gauge":0,"rate1":0,"rate5":0,"rate15":0,"p50":0,"p90":0,"p95":0,"p99":0},"http":{"count":0,"rate1":0,"rate5":0,"rate15":0,"p50":0,"p90":0,"p95":0,"p99":0}}}
    'http://<public_sub>.ngrok.io'

Check the Inspector at http://localhost:4040
--------------------------------------------

Check to see if you are able to access the `traffic inspection interface <https://ngrok.com/docs#getting-started-inspect>`_
via a web browser. If so, this at least means ``ngrok`` is able to start before throwing the error.

``ngrok`` Documentation
---------------------------

Familiarize yourself with the `ngrok documentation <https://ngrok.com/docs>`_, especially the sections pertaining to
`the config file <https://ngrok.com/docs#config>`_ and `the client API <https://ngrok.com/docs#client-api>`_.
