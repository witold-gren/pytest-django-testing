============
Pytest Xdist
============

Wtyczka pytest-xdist rozszerza pytest o kilka unikalnych trybów wykonywania testów:

* ``testowanie ciągłe`` (looponfail): uruchamiaj swoje testy w podprocesie. Uruchamiając
  ``pytest`` w tym trybie, najpierw są uruchamiane wszystkie testy. Po ich przejściu ``pytest``
  oczekuje na zmiany w plikach. Po ich dokonaniu są uruchamiane tylko te testy które są
  zależne od zmienionych plików. Jeśli wszystko będzie poprawne, ponownie są uruchamiane
  wszystkie testy, jeśli natomiast będa błędy, ``pytest`` testuje tylko te wybrane testy
  do czasu aż nie przejdą poprawnie.

* ``wieloprocesowe testowanie`` (`multiprocess load-balancing`): jeśli posiadasz wiele
  procesorów lub hostów, możesz ich użyć do złożonego testowawnia. Pozwala to przyspieszyć
  wykonywanie testów oraz pozwala skorzystać ze zasobów zdalnych maszyn.

* ``pokrycie wieloplatformowe`` (`multi-platform coverage`): pozwala określić różne
  interpretery języka python lub różne platformy oraz pozwala uruchamiać testy
  równolegle na wszystkich z nich.


Instalacja
----------

.. code-block:: python

    pip install pytest-xdist


Uruchamianie
------------

Wykorzystanie wielu procesorów do uruchamiania testów

.. code-block:: bash

    $ pytest -n NUM


Uruchamianie testów w subprocesach pythona

.. code-block:: bash

    $ pytest -d --tx popen//python=python2.7
    $ pytest -d --tx 3*popen//python=python2.7


Uruchomienie testów w trybie `looponfailing`

Tryb ten jest wykorzystywany podczas refaktoryzacji kodu. Zakładając, że występują
testy które nie przechodzą, ``pytest`` będzie oczekiwać na zmiany w plikach które,
zmieniają test a natępnie ponownie uruchomią zestaw testów. Zmiany w plikach są wykrywane
przez przeglądanie katalogu głównego root komputerów oraz ich zawartości (rekursywnie).

Niestety nie ma możliwości wykluczenia katalogów, sprawdzamy korzeń root oraz wszystkie
fildery i pliki zagnieżdżone. Jest to problemem kiedy struktura projektu jest zbudowana
w niepoprawny sposób.

.. code-block:: bash

    $ pytest -f ...


Wykorzystanie kont ssh aby wysłać i uruchomić testy

.. code-block:: bash

    $ pytest -d --tx ssh=myhostpopen --rsyncdir paczka


Uruchamianie testów na zdalnych serwerach poprzez socket

.. code-block:: bash

    $ pytest -d --tx socket=192.168.1.102:8888 --rsyncdir paczka


Uruchamianie testów na wielu platformach jednoczesnie

.. code-block:: bash

    $ pytest -d --dist=each --tx=cos1 --tx=cos2


Określanie środowiska testowego w pliku ``.ini``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Można również ustawić automatyczne testowanie w trynie ``looponfailroots`` w pliku ini:

.. code-block:: bash

    [pytest]
    looponfailroots = paczka


W pliku ini można ustawić domyśle działanie podczas uruchamiania testu. Można np. ustawić
działanie z trzema podprocesami.

.. code-block:: bash

    [pytest]
    ...
    addopts = -n3
    addopts = --tx ssh=myhost//python=python2.7 --tx ssh=myhost//python=python3.4


Można również ustawić katalogi które chcemy aby były dołączane lub niedołączane podczas
synchronizacji (rsync) przed uruchomieniem testów na zdalnych maszynach.

.. code-block:: bash

    [pytest]
    rsyncdirs = . mypkg helperpkg
    rsyncignore = .hg
