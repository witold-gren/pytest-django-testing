======
PyTest
======


Dlaczego pytest
---------------

* pozwala pisać małe i łatwe testy
* posiada mnogość pluginów jeszcze bardziej upraszczających testowanie
* można w nim uruchomić również UnitTest
* do porównywania wykorzystujemy tylko słowo ``assert`` a otrzymujemy bardzo szczegółowe informacje o błędach
* posiadamy ``fixture`` - wstrzykiwanie zależności
* posiadamy możliwość tworzenia markerów ``pytest.mark.skipif`` czy ``pytest.mark.xfail``
* możemy parametryzować testy zmniejszając ilość napisanego kodu
* automatycznie wykrywa moduły testowe, klasy i funkcje bez zbędnego dziedziczenia klas
* ``pytest`` pozwala na konfigurację projektu ``pytest.ini`` oraz każdego modułu w inny sposób poprzez wykorzystanie ``conftest.py``
* implementacja ``hooks``


.. hint::
    Do czego służą i na co nam pozwalają fixture?

    * unikanie samopowtarzalności w kodzie
    * można je w łatwy sposób modyfikować
    * wstrzykiwanie zależności
    * można je tworzyć w bardzo łatwy sposób


.. hint::

    Co to są markery i jak możemy je wykorzystać?

    * pozwalają kontrolować co ma zostać uruchomione w teście
    * `pytest` zawiera wbudowanych kilka markerów np. ``pytest.mark.skipif``, ``pytest.mark.xfail``, ``pytest.mark.parametrize``, ``pytest.mark.tryfirst``, ``pytest.mark.trylast`` i inne.
    * można tworzyć swoje markery, podczas uruchamiania testów można oznaczyć które mają zostać uruchomione


Instalacja
----------

.. code-block:: bash

    $ pip install pytest


Konfiguracja
------------

W katalogu głównym naszej aplikacji tworzymy plik ``pytest.ini`` w którym będzie znajdować
się konfiguracja ``pytest``. Uzywająć ``Django`` plik ``pytest.ini`` powinien znaleźć się
w katalogu w którym mamy umieszczony plik ``manage.py``, Nie jest ona wymagana,
jednak czasem przydaje się aby zmiejszyć ilość wpisywanych komend podczas uruchamiania testów.
Poniżej zamieszczona jest przykładowa konfiguracja:

.. code-block:: bash

    [pytest]
    python_files = tests.py test_*.py
    addopts = -s -q --disable-warnings --doctest-modules
    norecursedirs = .git .cache tmp*


Więcej szczegułów dotyczących konfiguracji można znaleźć w `Konfiguracji pytest`_ lub w dokumentacji
do poszczególnych pluginów. Przykładowa powyższa konfiguracja zawiera nagłówek ``[pytest]``,
oraz trzy ustawienia:

* python_files - ustawienie informujące ``pytest`` w jakich plikach ma poszukiwać testów,
* addopts - uruchamiając komendę ``pytest`` nie musimy za każdym razem podawać całego ciągu znaczników którymi chcemy ustawić test, w tym miejscu ustawiamy je jednorazowo i będą one automatycznie dołączane podczas uruchamiania testów. Wyjaśnienia: ``-s`` jest to skrót od ``--capture=no`` który wyłącza przechwytywanie wyjścia komunikatów np. print, ``-q`` zmniejsza szczegółowość danych podczas uruchomienia testu, ``--disable-warnings`` oznaca wyłączenie podsumowania o ostrzeżenie w kodzie, ``--doctest-modules`` uruchamia wszystkie `doctests` we wszystkich plikach ``.py``.
* norecursedirs - informacja które foldery należy wykluczyć podczas poszukiwania plików z testami

.. tip::

    Inne popularne ustawienia addopts to:
    * ``-x``, ``--exitfirst`` zamknięcie testów podczas pierwszej nieudanej próby wykonania testu
    * ``--maxfail=num`` wyjście po przekroczeniau ``num`` ilości błędnych testów
    * ``--fixtures`` pokazanie aktualnie dostępnych `fixtures`
    * ``--markers`` pokazanie wszystkich zainstalowanych `marks`
    * ``--pdb`` uruchomienie debugera kodu
    * ``-p no:warnings`` wyłączenie ostrzeżeń podczas testów
    * ``-v``, ``-vv``, ``-vvv``, ``-vvvv`` szczegułowość komunikatów o błędach


.. tip::

    Inne ustawienia w pliku ``pytest.ini``:
    * ``python_classes = *Suite`` stawienie typu klasy, w której będą poszukiwanie testy
    * ``python_functions = *_test`` - ustawienie typu funkcji które będą uruchamiane jako testy


.. _`Konfiguracji pytest` : https://docs.pytest.org/en/latest/customize.html


Uruchomienie testów
-------------------

Uruchomienie ``pytest`` dla konkretnego pliku

.. code-block:: bash

    $ pytest test_mymodule.py
    $ pytest -vsl test_mymodule.py


Uruchomienie wszystkiego co ma w nazwie `special_run`

.. code-block:: bash

    $ pytest -k 'special_run'


Uruchomienie testów które są udekorowane wybranym markerem `marker_name`

.. code-block:: bash

    $ pytest -m 'marker_name'


Jeśli posiadamy plugin `xdist` uruchomi on testy na 4 procesorach

.. code-block:: bash

    $ pytest -n 4


Oznaczanie całych klas lub modułów markerem
-------------------------------------------

Jeśli utworzymy dekorator markera na klasie, wszystkie testy klasy będą oznaczone tym markerem.

.. code-block:: python

    # content of test_mark_classlevel.py
    import pytest
    @pytest.mark.webtest
    class TestClass(object):
        def test_startup(self):
            pass
        def test_startup_and_more(self):
            pass

Dla zachowania kompaktybilności wstecznej z wersją 2.4 możemy również użyć zmienne
``pytestmark``. Jest to równoznaczne z utowrzeniem dekoratora z merkerem na klasie.

.. code-block:: python

    import pytest

    class TestClass(object):
        pytestmark = pytest.mark.webtest


Można równiez podać kilka markerów w liście.

.. code-block:: python

    import pytest

    class TestClass(object):
        pytestmark = [pytest.mark.webtest, pytest.mark.slowtest]


Oznaczenie całego modułu markerem można wykonać w nastepujący sposób.

.. code-block:: python

    import pytest
    pytestmark = pytest.mark.webtest


Łapanie wyjątków
----------------

.. code-block:: python

    def test_zero_division():
        with pytest.raises(ZeroDivisionError):
            1 / 0

    def test_recursion_depth():
        with pytest.raises(RuntimeError) as excinfo:
            def f():
                f()
            f()
        assert 'maximum recursion' in str(excinfo.value)


Jak pisać kod
-------------

.. code-block:: python

    class TestCals:

        def test_add_method(self):
            calc = Cals()
            assert calc.add(1, 1) == 2
            assert calc.add(0, 3) == 3


.. code-block:: python

    @pytest.fixture(scope='function')
    def calc(request):
        c = Calc()
        return c

    class TestCals:

        def test_add_method(self, calc):
            assert calc.add(1, 1) == 2
            assert calc.add(0, 3) == 3


.. code-block:: python

    @pytest.fixture(scope='function')
    def calc(request):
        c = Calc()
        return c

    class TestCals:

        @pytest.mark.parametrize('a, b, exp', [
            (1, 1, 2), (0, 3, 3)
        ])
        def test_add_method(self, calc, a, b, exp):
            assert calc.add(1, 1) == 2
            assert calc.add(0, 3) == 3


.. code-block:: python

    @pytest.fixture(scope='function')   # or 'session'
    def calc(request):
        c = Calc()
        return c

    class TestCals:

        @pytest.mark.parametrize('a, b, exp', [
            (1, 1, 2), (0, 3, 3)
        ])
        def test_add_method(self, calc, a, b, exp):
            assert calc.add(1, 1) == 2
            assert calc.add(0, 3) == 3

        # pytest-quickcheck
        @pytest.mark.randomize(a=int, ncalls=4)
        def test_add_method(self, calc, a):
            assert calc.add(a, a) == 2 * a


.. code-block:: python

    api_mark = pytest.mark.on_api
    local = pytest.mark.local

    @pytest.fixture(scope='session')
    def calc(request):
        c = Calc()
        return c

    @local
    class TestCals:

        @pytest.mark.parametrize('a, b, exp', [
            (1, 1, 2), (0, 3, 3)
        ])
        def test_add_method(self, calc, a, b, exp):
            assert calc.add(1, 1) == 2
            assert calc.add(0, 3) == 3

        # pytest-quickcheck
        @pytest.mark.randomize(a=int, ncalls=4)
        def test_add_method(self, calc, a):
            assert calc.add(a, a) == 2 * a

    @api_mark
    class TestServer:

        def test_on_api(self):
            assert False

    # $ pytest -v -m local file_name.py
    # $ pytest -v -m on_api file_name.py


.. code-block:: python

    api_mark = pytest.mark.on_api
    local = pytest.mark.local

    @pytest.fixture(scope='session')
    def calc(request):
        c = Calc()
        return c

    @pytest.fixture(scope='session')
    def api(request):
        def api_cal(a, b):
            res = request.get('http://127.0.0.1:3007/add/', params={'a':a, 'b': b})
            res.raise_for_status()
            return res.json()
        return api_cal

    @local
    class TestCals:

        @pytest.mark.parametrize('a, b, exp', [
            (1, 1, 2), (0, 3, 3)
        ])
        def test_add_method(self, calc, a, b, exp):
            assert calc.add(1, 1) == 2
            assert calc.add(0, 3) == 3

        # pytest-quickcheck
        @pytest.mark.randomize(a=int, ncalls=4)
        def test_add_method(self, calc, a):
            assert calc.add(a, a) == 2 * a

    @api_mark
    class TestServer:

        def test_on_api(self, api):
            assert api(1, 2) == 3

    # $ pytest -v -m local file_name.py
    # $ pytest -v -m on_api file_name.py


.. code-block:: python

    mode = pytest.mark.mode

    ...

    @mode('local')
    class TestCals:
        ...

    @mode('api')
    class TestServer:
        ...

    # $ pytest -v -R local file_name.py
    # $ pytest -v -R api file_name.py


.. code-block:: python

    def local_calc(request):
        c = Calc()
        return x

    def api(request):
        def api_cal(a, b):
            ...

    @pytest.fixture(scope='session')
    def calc(request):
        mode = request.config.getoption('-R)
        if mode == 'local':
            return local_calc(request)
        elif mode == 'api':
            return api(request)
        else:
            raise Exception('local or api allowed')

    @mode('local')
    class TestCals:
        def test_add(self, calc):
            ...

    @mode('api')
    class TestServer:
        def test_add(self, calc):
            ...


styl xUnit
----------

.. code-block:: python

    def setup_module(module):
        print('\nsetup_module()')

    def teardown_module(module):
        print('teardown_module()')

    def setup_function(function):
        print('\nsetup_function()')

    def teardown_function(function):
        print('\nteardown_function()')

    def test_1():
        print('-  test_1()')

    def test_2():
        print('-  test_2()')


    class TestClass:

        @classmethod
        def setup_class(cls):
            print ('\nsetup_class()')

        @classmethod
        def teardown_class(cls):
            print ('teardown_class()')

        def setup_method(self, method):
            print ('\nsetup_method()')

        def teardown_method(self, method):
            print ('\nteardown_method()')

        def test_3(self):
            print('- test_3()')

        def test_4(self):
            print('- test_4()')
