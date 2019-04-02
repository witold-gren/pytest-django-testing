===================
Pylama - audyt kodu
===================

Pylama jest narzędzie do audytu kodu dla pythona. Moduł ten integruje i wykorzystuje
kilka zewnętrznych modułów:

* `pycodestyle`_ (sprawdzanie poprawności kodu pod kątem PEP8)
* `pydocstyle`_ (sprawdzenie poprawności docstring którego opis znajduje się w PEP257)
* `pyflakes`_ (program, który sprawdza pliki źródłowe Pythona pod kątem błędów)
* `mccabe`_ (mikro-narzędzie do sprawdzania złożoności cyklomatycznej kodu)
* `pylint`_ (narzędzie do weryfikacji kodu)
* `radon`_ (narzędzie do obliczania różnych metryk kodu)

.. _`pycodestyle`: http://pycodestyle.pycqa.org/en/latest/
.. _`pydocstyle`: http://www.pydocstyle.org/en/2.1.1/
.. _`pyflakes`: https://github.com/PyCQA/pyflakes
.. _`mccabe`: https://github.com/pycqa/mccabe
.. _`pylint`: https://pylint.org/
.. _`radon`: http://radon.readthedocs.io/en/latest/


Instalacja
----------

.. code-block:: bash

    $ pip install pylama


Ustawienia
----------

Ustawienia tego modułu można dokonać w jednym z poniższych plików:

* pylama.ini
* setup.cfg
* tox.ini
* pytest.ini

Istnieje kilka konfiguracji. Pierwszą z nich jest sekcja "`pylama`" konfigurująca globalne opcje.

.. code-block:: bash

    [pylama]
    format = pylint
    skip = */.tox/*,*/.env/*
    linters = pylint,mccabe
    ignore = F0401,C0111,E731

Można również ustawić opcje specjalnego sprawdzania kodu dla poszczególnych konfiguracjami narzędzi.

.. code-block:: bash

    [pylama:pyflakes]
    builtins = _

    [pylama:pycodestyle]
    max_line_length = 100

    [pylama:pylint]
    max_line_length = 100
    disable = R

Ostatnią możliwością jest ustawienie opcji dla pliku (lub grupy plików) w sekcjach.
Opcje te mają wyższy priorytet niż sekcja "pylama".

.. code-block:: bash

    [pylama:*/pylama/main.py]
    ignore = C901,R0914,W0212
    select = R

    [pylama:*/tests.py]
    ignore = C0110

    [pylama:*/setup.py]
    skip = 1


Wykorzystanie
-------------

Pylama posiada wsparcie dla ``pytest``. Pakiet automatycznie rejestruje się jako dodatek
do pytest podczas instalacji.

.. code-block:: bash

    $ pytest --pylama ...


Więcej szczegółów konfiguracyjnych można znaleźć na stronie https://pylama.readthedocs.io/en/latest/


Złożoność cyklomatyczna McCabe'a
--------------------------------

Złożoność cyklomatyczna (CC), mimo swojej długiej historii – została zdefiniowana w 1976
roku z myślą o programowaniu strukturalnym – jest nadal podstawową miarą złożoności
dowolnego fragmentu kodu.

==================  ===============
wartość CC          Interpretacja
==================  ===============
1 - 10              prosta metoda
11 - 20             metoda złożona
21 - 50             metoda bardzo złożona
> 50                testowanie niemal niemożliwe
==================  ===============


Możliwości modułu Radon
-----------------------

* obliczenie złożoność cyklomatycznej
* całkowita liczba linii kodu (LOC)
* liczba logicznych linii kodu (LLOC)
* liczba linii źródłowych kodu (SLOC)
* liczba linii komentarza
* liczba linii reprezentujących wieloliniowe ciągi
* liczba pustych linii
* złożoność Halsteada (trudność, poziom programu, wysiłek, czas, szacunkowa liczba błędów itd.)
