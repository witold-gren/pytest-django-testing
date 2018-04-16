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


Pisanie własnych fixture
------------------------

W większości frameworków testowych `fixture` są powszechne. Są to w zasadzie
obiekty, które możemy wykorzystać w naszych testach. Ostatecznie zapewniają
stałą linię bazową, na której testy mogą być wykonywane niezawodnie i wielokrotnie.
W pytest fixture, które wykraczają poza typową konfigurację i funkcjonalność.

- `fixture` posiadają jawne nazwy i są aktywowane poprzez deklarowanie ich
w funkcjach testowych, modułach, klasach lub całych projektach.
- `fixture` są modułowe, a każde `fixture` wyzwala funkcję urządzenia,
które może korzystać z innych `fixture`.
- Możesz sparametryzować `fixture` i testy zgodnie z opcjami konfiguracji
lub ponownie wykorzystać `fixture` w obrębie zakresów klasy, modułu lub
całej sesji testowej.


Tworzenie własnego fixture
^^^^^^^^^^^^^^^^^^^^^^^^^^

Aby utworzyć własny `fixture` należy wykorzystać dekorator `pytest.fixture`.

.. code-block:: python

    import pytest

    @pytest.fixture()
    def my_fixture():
        print "\nI'm the fixture"


Używanie fixture w kodzie
^^^^^^^^^^^^^^^^^^^^^^^^^

Aby użyć wcześniej napisanego `fixture`, wystarczy że przekażemy go jako
parametr w funkcji testowej. Należy pamiętać, że zawsze jako pierwszy zostanie
wykonana funkcja `fixture` a dopiero potem funkcja testująca.

.. code-block:: python

    def test_my_fixture(my_fixture):
        print "I'm the test"


Pytest podaje nam kilka innych sposobów korzystania z naszych `fixture`.
Metoda standardowego parametru jest świetna i używana najczęściej, ale mamy
również dekorator `usefixtures()`.

.. code-block:: python

    @pytest.mark.usefixtures('my_fixture')
    def test_my_fixture():
        print "I'm the test"

Oznaczamy test, aby użyć naszego `fixture`, a wyniki jego działania są takie
same jak wcześniej. Warto zwrócić uwagę, że można przekazać wiele `fixture`
za pomocą wartości rozdzielanych przecinkami. Metoda ta jest przydatna w
klasach testowych.

.. code-block:: python

    @pytest.mark.usefixtures('my_fixture', 'my_fixture2')
    class Test:
        def test1(self):
            print "I'm the test 1"

        def test2(self):
            print "I'm the test 2"


Używamy `fixture` dla całej klasy, a następnie każdy test w klasie użyje tego `fixture`.
Oszczędza to czas oznaczania wszystkich testów, jeśli mają korzystać z tego
samego urządzenia. Innym sposobem uzyskania tego samego efektu na całym pliku
testowym jest ustawienie zmiennej `pytestmark`.

.. code-block:: python

    import pytest

    pytestmark = pytest.mark.usefixtures('my_fixture')

    def test_my_fixture():
        print "I'm the test"

    class Test:
        def test1(self):
            print "I'm the test 1"

        def test2(self):
            print "I'm the test 2"

W ten sposób ustawiamy `fixture` globalnie dla tego pliku, a wszystkie
funkcje testowe znajdujące się w nim, będą go używać.

Należy pamiętać, że wszystkie funkcje testowe mogą nie wymagać tego `fixture`.
Jeśli tak jest, lepiej jest bezpośrednio określić każdy `fixture` osobno dla
funkcji testującej, zamiast wybierać leniwe drogi i oznaczać je z góry na
wszystkich funkcjach. W przypadku większych `fixture` może to spowodować,
że testy będą ładować się wolniej.


Ostatnim sposobem użycia `fixture`, jest ustawienie parametru `autouse` w deklaracji `fixture`.
`Fixture` będzie automatycznie wywoływany bez jawnego deklarowania argumentów
funkcji lub dekoratora usefixtures.

`Fixture` - `transact` na poziomie klasy jest oznaczone jako `autouse=True`, co
oznacza, że ​​wszystkie metody testowe w klasie będą używać tego `fixture` bez
potrzeby podawania go w sygnaturze funkcji testowej lub przy użyciu dekoratora
klasy używanej na poziomie klasy.

.. code-block:: python

    import pytest

    class DB(object):
        def __init__(self):
            self.intransaction = []
        def begin(self, name):
            self.intransaction.append(name)
        def rollback(self):
            self.intransaction.pop()

    @pytest.fixture(scope="module")
    def db():
        return DB()

    class TestClass(object):

        @pytest.fixture(autouse=True)
        def transact(self, request, db):
            db.begin(request.function.__name__)
            yield
            db.rollback()

        def test_method1(self, db):
            assert db.intransaction == ["test_method1"]

        def test_method2(self, db):
            assert db.intransaction == ["test_method2"]


Używanie `autouse` może być wspaniałe, ale może być również niebezpieczne,
jak pokazano w ostatnim przykładzie. `Autouse`, o ile nie jest ograniczone do zakresu,
będzie działać na wszystkich testach w bieżącej sesji.

Praca i zakres `autofocus`:
- ustawienia `autouse=True` jest zgodne z `scope=`argument: jeśli `fixture`
ma `scope='session'`, to zostanie ono uruchomione tylko raz, bez względu na to,
gdzie zostało ono zdefiniowane.
scope = 'class' oznacza, że ​​będzie uruchamiany raz na klasę, itd.
- jeśli zdefiniowano `fixture` z parametrem `autouse=True` w module testowym,
wszystkie jego funkcje testowe automatycznie będą go używać.
- jeśli zdefiniowano `fixture` z parametrem `autouse=True` w pliku conftest.py,
wówczas wszystkie testy we wszystkich modułach testowych poniżej jego katalogu wywołają tego `fixture`.


Warto zauważyć, że powyższy `fixture` z argumentem `autouse` również może zostać zwykłym `fixture`,
którego można użyć w projekcie bez automatycznej aktywacji. Kanonicznym sposobem
na to, jest umieszczenie definicji transakcji w pliku conftest.py
bez użycia funkcji `autouse=True`:

.. code-block:: python

    # content of conftest.py
    @pytest.fixture
    def transact(self, request, db):
        db.begin()
        yield
        db.rollback()

a następnie np. w klasie `TestClass`, deklarujesz jego użycie:

.. code-block:: python

    @pytest.mark.usefixtures("transact")
    class TestClass(object):
        def test_method1(self):
            ...

Wszystkie metody testowe w tej klasie testowej będą używać `fixture` transakcyjnego,
podczas gdy inne klasy testowe lub funkcje w tym samym module nie będą go używać.


Zwracanie wartości
^^^^^^^^^^^^^^^^^^

`fixture` są używane przede wszystkim do zwracania danych, którymi można manipulować podczas testów.
Tak jak zwykła funkcja, możemy zwrócić coś, a następnie w naszym teście możemy z tego skorzystać.

.. code-block:: python

    import pytest

    @pytest.fixture()
    def my_fixture():
        data = {'x': 1, 'y': 2, 'z': 3}
        return data

    def test_my_fixture(my_fixture):
        assert my_fixture['x'] == 1


Dodawanie finalizerów
^^^^^^^^^^^^^^^^^^^^^

Jeśli chcesz uruchomić coś po zakończeniu testu z `fixture`, możesz użyć finalizatorów.
W tym celu uzyskujemy dostęp do `request fixture` z pytest.
Finalizator to funkcja wewnątrz `fixture`, która będzie uruchomiona po każdym teście,
w którym znajduje się dany `fixture`.

.. code-block:: python

    @pytest.fixture()
    def my_fixture(request):
        data = {'x': 1, 'y': 2, 'z': 3}

        def fin():
            print "\nMic drop"
        request.addfinalizer(fin)

        return data

`request` posiada metodę `addfinalizer()`, która może przyjąć funkcję.
Nasza funkcja może po prostu wypisywać coś na ekranie lub np. możemy odłączyć
się od bazy danych. Daje nam to kontrolę nad `fixture` po zakończeniu testu,
które go wykorzystuje.

Zakres fixture
^^^^^^^^^^^^^^

Wielokrotnie możemy chcieć mieć `fixture`, który chcemy uruchomić na przykład na wszystkich
funkcjach lub we wszystkich klasach. Pytest podaje nam zestaw kilka zmiennych, które dokładnie określają zakres,
kiedy chcemy korzystać z naszego `fixture`.

- `function`: uruchomienie `fixture` jeden raz na przypadek testowy
- `class`: uruchomienie jeden raz na klasę
- `module`: uruchomienie jeden raz na moduł
- `session`: uruchomienie jeden raz na sesję

Aby z nich skorzystać, definiujemy argument `scope`.

.. code-block:: python

    @pytest.fixture(scope="class")

Domyślnie `scope` jest ustawione na `function`. Gdzie chciałbyś użyć każdego z nich?

- Możesz użyć `function`, jeśli chcesz, aby urządzenie działało po każdym pojedynczym teście.
Jest to dobre rozwiązanie w przypadku utowrzenia małych `fixture`.
- Zakres `class`, jest wykorzystywany jeśli chcesz, aby działał on w każdej klasie.
Zazwyczaj grupujemy testy w jednej klasie kiedy są podobne. Ten zakres jest wykorzystywany
własnie wtedy kiedy chcemy wykonać coś jeden raz dla całej grupy testów.
- Zakres `module`, można użyć jeśli chcemy, aby `fixture` był uruchamiany na początku
bieżącego pliku, a następnie zakończony po uruchomieniu wszystkich testów znajdujących się wewnątrz pliku.
Ten zakres można wykorzystać jeśli masz `fixture`, który uzyskuje dostęp do bazy danych
i konfiguruje bazę danych na początku modułu, a następnie finalizator zamyka połączenie.
- Zakres `session` jest wykorzystywany, jeśli chcemy uruchomić `fixture` w pierwszym teście a następnie
uruchomić finalizator po uruchomieniu ostatniego testu. Jeśli zakres ustawimy na `session` a `autouse=True`,
to nasz `fixture` zostanie uruchomiony tylko na początku sesji.


Używanie informacji o fixture w testach
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pytest zapewnia dostęp do informacji o `fixture` poprzez wykorzystanie argumentu `request`.

- `scope`: request.scope
- `function name`: request.function.__name__
- `class`: request.cls
- `module`: request.module.__name__
- `filesystem path`: request.fspath


Dodawanie parametrów do fixture
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pytest zapewnia również wielokrotne używanie pojedynczego `fixture`.
Poprzez przekazanie parametru `params=[]` do definicji `fixture` możemy stworzyć
wiele `fixture`. Poniższy przykład pokazuje jak to zrobić.

.. code-block:: python

    import pytest

    @pytest.fixture(params=[
        # Tuples with password string and expected result
        ('password', False),
        ('p@ssword', False),
        ('p@ssw0rd', True)
    ])
    def password(request):
        """Password fixture"""
        return request.param


    def password_contains_number(password):
        """Checks if a password contains a number"""
        return any([True for x in range(10) if str(x) in password])


    def password_contains_symbol(password):
        """Checks if a password contains a symbol"""
        return any([True for x in '!,@,#,$,%,^,&,*,(,),_,-,+,='.split(',') if x in password])


    def check_password(password):
        """Check the password"""
        return password_contains_number(password) and password_contains_symbol(password)


    def test_password_verifier_works(password):
        """Test that the password is verifyied correctly"""
        (input, result) = password
        print '\n'
        print input

        assert check_password(input) == result

Mimo iż uruchomiliśmy tylko jeden test (`test_password_verifier_works`),
w sumie został on uruchomiony trzy krotnie, każdy z innymi wartościami.


Pomijanie testów
----------------

Wiele razy wiemy, że test zakończy się niepowodzeniem. W takich przypadkach
chcemy zmodyfikować test lub zmodyfikować kod, jednak nadal posiadanie testu
który nie przechodzi może zablokować zestaw testowy, aby tego uniknąć pytest
daje nam narzędzia `skip` oraz `xfail`, które pozwolą nam kontrolować takie zachowanie.

`skip` oznacza, że test zostanie uruchomiony, chyba że środowisko (np. nieprawidłowy
interpreter języka Python, brak zależności) zapobiegają jego uruchomieniu.

`xfail` natomiast oznacza, że test zostanie uruchomiony zawsze, ale spodziewamy
się niepowodzenia, ponieważ wystąpił problem z implementacją.

.. code-block:: python

    import pytest
    import sys

    @pytest.mark.skipif(sys.platform != 'win32', reason="requires windows")
    def test_func_skipped():
        """Test the function"""
        assert 0

    @pytest.mark.xfail
    def test_func_xfailed():
        """Test the function"""
        assert 0

skip
^^^^

Powyżej przeprowadziliśmy dwa testy, jeden został pominięty (ponieważ nie został
uruchomiony w systemie windows), a drugi był nieudany, ponieważ wiedzieliśmy,
że to nie zadziała. Przyjżyjmy sie najpierw pomijaniu testów.

Podczas pomijania testów, podajemy warunek który musi zostać spełniony. Jeśli
warunek nie zostanie spełniony, test nie zostanie uruchomiony oraz zostanie
oznaczony jako pominięty. Jest to idealne rozwiązanie do testów, które mogą
wymagać konkretnych wersji modułów i oprogramowania. Może to być kłopotliwe,
jeśli mamy szereg testów, które wymagają takiej samej konfiguracji pomijania,
warto stworzyć dekorator ułatwiający oznaczanie testów.

.. code-block:: python

    import sys
    import pytest

    windows = pytest.mark.skipif(sys.platform != 'win32', reason="requires windows")

    @windows
    def test_func_skipped():
        """Test the function"""
        assert 0

Możemy zastosować dekorator `@windows` do dowolnej funkcji testującej.
Dodatkowym sposobem pozwalającym na pominięcie jest wykorzystanie `importorskip`.

.. code-block:: python

    docutils = pytest.importorskip("docutils", minversion="0.3")

Jeśli nie można zaimportować `docutils`, spowoduje to pominięcie testu.


xfail
^^^^^

Wykorzystując `xfail` również możemy skorzystać z podobnych warunków jakie
występują w funckji `skip`.

.. code-block:: python

    import pytest
    import sys


    @pytest.mark.xfail(sys.version_info >= (3,3), reason="python3.3 api changes")
    def test_func_xfailed():
        """Test the function"""
        assert 0

Poniżej znajduje się kila przykładów pokazujących w jaki sposób można wykorzystać
funckję `xfail`.

.. code-block:: python

    import pytest
    xfail = pytest.mark.xfail

    @xfail
    def test_hello():
        assert 0

    @xfail(run=False)
    def test_hello2():
        assert 0

    @xfail("hasattr(os, 'sep')")
    def test_hello3():
        assert 0

    @xfail(reason="bug 110")
    def test_hello4():
        assert 0

    @xfail('pytest.__version__[0] != "17"')
    def test_hello5():
        assert 0

    def test_hello6():
        pytest.xfail("reason")

    @xfail(raises=IndexError)
    def test_hello7():
        x = []
        x[1] = 1

Określając `run=False` test nie zostanie uruchomiony. Możemy również użyć
wyrażenia tekstowego jako testu, aby sprawdzić, czy test nie powiedzie się.
Możemy również w samym teście wywołać funkcjię `pytest.xfail("reason")`, która
spowoduje, że się nie powiedzie.

Korzystając z `xfail` i `skip`, możesz podać powód dlaczego test się nie powiedzie
lub dlaczego zostaje on pominięty. Kiedy uruchomimy testy, nie zobaczymy tych powodów.
Aby zobaczyć opisy dla tych funkcji należy uruchomić testy z następującą komendą:


.. code-block:: bash

    $ pytest -rxs


Parametryzacja testów
---------------------

W niektórych przypadkach wystarczy utworzyć jednorazowy test lecz często zdarza
się że chcemy sprawdzić kila przypadków zmieniając wartości wybranych zmiennych.
W takiej sytuacji nie ma potrzeby pisać kolejnych przypadków testowych, ale warto
skorzystać z parametryzacji jednego przypadku testowego.

.. code-block:: python

    import pytest

    @pytest.mark.parametrize('input, expected', [
        ('2 + 3', 5),
        ('6 - 4', 2),
        pytest.mark.xfail(('5 + 2', 8))
    ])
    def test_equations(input, expected):
        """Test that equation works"""
        assert eval(input) == expected


Ustawienia xUnit - konfiguracja i odłogowanie
---------------------------------------------

Testując w stylu XUnit zawsze wykonujemy ustawienie (`setting up`) oraz
czyszczenie (`tearing down`) przypadków testowych. Pytest również obsługuje
ten styl pisania testowów.

.. code-block:: python

    def setup_module(module):
        """Run at the start of a testing module (module)"""
        pass

    def teardown_module(module):
        """Run at the end of a testing module (file)"""
        pass

    def setup_function(function):
        """Setup a function"""
        pass

    def teardown_function(function):
        """Teardown a function"""
        pass

    class TestClass:

        @classmethod
        def setup_class(cls):
            """Setup the class"""
            pass

        @classmethod
        def teardown_class(cls):
            """Teardown the class"""
            pass

        def setup_method(self, method):
            """Setup a method"""
            pass

        def teardown_method(self, method):
            """Teardown a method"""
            pass


Praca z wyjątkami
-----------------

Jeśli wiemy, że dany kod powinien podnieść wyjątek i chcemy go przetestować, czy
na pewno został wywołany, musimy użyć funkcji `pytest.raises`.

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


Przykład pisania kodu
---------------------

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
