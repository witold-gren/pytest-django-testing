===========
Pytest Pudb
===========

``pytest-pudb`` jest to wtyczka do ``pytest``, która ułatwia wruchomienie debagera ``pudb``
podczas testowania. Podczas uruchamiania debugera należy wyłączyć przechwytywanie konsoli
wykorzystaując flagę ``-s`` lub ``--capture=no``. Więcej informacji można uzyskać korzystając
z dokumentacji pudb_.


.. _pudb: https://documen.tician.de/pudb/index.html


.. hint::

    Korzystając z ``docker-compose`` należy uruchomić go w odpowiednim trybie. Dzięki temu
    konsola przekaże nam okno do debugowania: ``docker-compose run --service-ports CONTAINER_NAME``.


Instalacja
----------

.. code-block:: bash

    $ pip install pytest-pudb


Uruchomienie
------------

.. code-block:: bash

    $ pytest --pudb -s
    $ pytest --pdbcls pudb.debugger:Debugger --pdb -s


.. code-block:: python

    def test_set_trace_integration():
        import pudb; pudb.set_trace()
        assert 1 == 2

    def test_pudb_b_integration():
        import pudb.b
        # traceback is set up here
        assert 1 == 2