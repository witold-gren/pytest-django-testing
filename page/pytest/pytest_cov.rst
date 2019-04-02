===============
Pytest Coverage
===============

Coverage jest pluginem sprawdzającym pokrycie kodu testami wyrażone w procentach.
pytest-cov_ jest dodatkiem do ``pytest`` ułatwiający pracę z `coverage`_ - oryginalnym
pluginem napisanym dla języka python. Dodatek ten oprócz wszystkich możliwości standardowej
biblioteki, pozwala dodatkowo uruchomić sprawdzenie pokrycia kodu podczas testowana
(dodatkowe parametry do pytest), pozwala wraz z pluginem `pytest-xdist` uruchomić
testowanie na kilku wątkach.


Instalacja
----------

.. code-block:: bash

    $ pip install pytest-cov


Konfiguracja
------------

Możemy skonfigurować oryginalny plik ``.coveragerc`` a później wskazać go podczas
uruchomienia konfiguracji. Więcej informacji o konfiguracji znajdziemy w `dokumentacji coverage`_


.. code-block:: bash

    [run]
    omit = tests/*


Uruchomienie pytest wraz ze wskazaniem pliku do konfiguracji dla ``coverage``.

.. code-block:: bash

    pytest --cov-config .coveragerc --cov=myproj myproj/tests/


Generowanie raportów
--------------------

Posiadamy kilka opcji podczas generowania raportów:

* term - uruchomienie raportu w terminalu
* term-missing - uruchomienie raportu w terminalu wraz z informacją jakie linie kodu zostały przetestowane
* html - generowanie raportu html
* xml - generowanie raportu xml
* annotate - tworzy nam kopię plików i oznacza w nich co zostało przetestowane
* :skip-covered - opcja z pominięciem kodu, który jest przetestowany w 100%


``skip-covered`` można łączyć wraz z innymi generatorami w celu pominięcia raportu,
który jest pokryty w 100%.


.. code-block:: bash

    $ pytest --cov-report term --cov=myproj tests/

    -------------------- coverage: platform linux2, python 2.6.4-final-0 ---------------------
    Name                 Stmts   Miss  Cover
    ----------------------------------------
    myproj/__init__          2      0   100%
    myproj/myproj          257     13    94%
    myproj/feature4286      94      7    92%
    ----------------------------------------
    TOTAL                  353     20    94%


.. code-block:: bash

    $ pytest --cov-report term:skip-covered --cov=myproj tests/


.. code-block:: bash

    $ pytest --cov-report html
             --cov-report xml
             --cov-report annotate
             --cov=myproj tests/


Debugowanie i IDE
-----------------

Podczas gdy mamy zainstalowany dodatek używa on ``sys.settrace`` jako dostęp do kontekstu.
Dlatego wykorzystując jakiekolwiek IDE należy wyłączyć plugin aby nie powodował problemów
podczas wykorzystywania np. ``breakpoint``.

.. code-block:: bash

    $ pytest --no-cov


Markery
-------

:code:`pytest.marker.no_cover` jest to marker pozwalający nam na całkowite wyłączenie konkretnego testu podczas tworzenia
raportu kodu pokrytego testami.

.. code-block:: python

    @pytest.marker.no_cover
    def test_foobar():
        ...


.. _`coverage`: https://coverage.readthedocs.io/en/latest/
.. _pytest-cov: https://pytest-cov.readthedocs.io/en/latest/
.. _`dokumentacji coverage`: https://coverage.readthedocs.io/en/latest/config.html
