===========
Pytest Mock
===========

Pytest-mock jest pluginem ułatwiającym tworzenie mocków w testach. Nie musimy importować
modułu Mock, patch i innych, są one dostępne bespośrednio jako fixture. Jednak aby zacząć
z niego korzystać musimy zrozumieć czym jest Mock oraz w jaki wposób działa. Pytest-mock
nie robi żadnej magi wokoło modułu mocków, jednak jeśli nie rozumiemy jak działa obiekt
Mock będziemy mieli problem z zrozumieniem w jaki sposób z niego korzytać.

Czym jest mokowanie? Jesy to symulowaniem działania obiektu. Mokowanie obiektów jest
bardzo dobrym narzędziem. Jednak należy uważać z jego nadużywaniem. Dobrym miejscem do
ich wykorzystania są:

* systemowe wywołania (np. ``os.environ``)
* strumienie (np. ``sys.stdout``)
* sieć (np. ``request.open``)
* operacje wejścia/wyjścia (np. ``json.loads``)
* zegar, czas, data (np. ``time.sleep``)
* nieprzewidywalne wyniki (np. ``random.random``)

.. danger::

    Złym miejscem wykonywania mocków jest symulowanie działania bazy danych bez utworzenia
    testów integracyjnych. Również nie powinno sie dokonywać mokowania bazy danych podczas
    wykonywania samych testów integracyjnych. W Django poprzez test integracyjny rozumiemy
    korzystanie z narzędzia ``WebTest``, ``Selenium`` czy ``Django Client``.


Przykład rozwiązania, które może spowodować problemy podczas testowania aplikacji.

.. code-block:: python

    class DjangoFakeModel:
        def __init__(self, age=None):
            self.age = age

        def save_base(self, *args, **kwargs):
            assert NotImplementedError('call save_base')

    class DjangoFakeForm:
        def __init__(self, instance=None, data=None):
            self.instance = instance
            self.data = data

        def is_valid(self):
            return True

        def save(self):
            for key, value in self.data.items():
                setattr(self.instance, key, value)
            self.instance.save_base()
            return self.instance

    def test_solution1(mocker):
        mock_save = mocker.object(DjangoFakeModel, 'save_base')
        form = DjangoFakeForm(instance=DjangoFakeModel(), data={'age': 3})

        assert form.is_valid()
        saved_pony = form.save()

        assert saved_pony.age == 3
        mock_save.assert_called_once()   # <- a raczej powinno być assert_called_once_with


Problem z tworzeniem mocka polega na tym, że bardzo łatwo można przez pomyłkę wywołać funckję,
która będzie bardzo podobna do oryginalnej a jednak nie zwróci ona błędu. Przykładem może być
``mock_save.assart_called_once()``. Wykonanie powyższego testu będzie zawsze poprawne. Na szczęscie
wywołanie na MagicMock metody która rozpoczyna się od ``assert_`` będzie również sprawdzona
poprawność wywołania, co zabezpiecza nas przed popełnieniem błędu.


Jak działa Mock?
----------------

Aby skorzystać z obiektu Mock należy go zaimportować. W python 2 importujemy go poprzez ``import mock``
(wczesniej należy zainstalować bibliotekę ``pip install mock``) natomiast w pythonie 3
importujemy go z modułu unittest ``from unittest import mock``.

Utworzenie Mock odbywa się poprzez utworzene obiektu klasy Mock. Obiekt ten posiada szczegulną
własność, potrafi w locie utworzyć atrybuty i metody które są mu potrzebne. Warto tworząc
obiekt mock podać atrybut ``name``, dzięki temu będziemy wiedzieli jaki mock aktualnie
jest uruchomiony.

.. code-block:: python

    >>> m = mock.Mock(name='my_first_mock')
    >>> m
    <Mock name='my_first_mock' id='4622279400'> # normalnie mamy wartość <Mock id='4622279400'>

Obiekt Mock zawiera kilka specialnych metod i atrybutów.

.. code-block:: python

    >>> dir(m)
    ['assert_any_call', 'assert_called_once_with', 'assert_called_with', 'assert_has_calls',
     'attach_mock', 'call_args', 'call_args_list', 'call_count', 'called', 'configure_mock',
     'method_calls', 'mock_add_spec', 'mock_calls', 'reset_mock', 'return_value', 'side_effect']

Próbując odczytać nie istniejący atrybut nie otrzymamy błedu `AttributeError`, otrzymujemy
kolejny obiekt Mock. Nowy obiekt jest na stałe przypisany do wywołanego atrybutu.
Kilkukrotne wywołanie tego samego atrybutu zawsze zwróci ten sam Mock.

.. code-block:: python

    >>> m.some_attribute
    <Mock name='mock.some_attribute' id='140222043808432'>
    >>> dir(m)
    ['assert_any_call', 'assert_called_once_with', 'assert_called_with', 'assert_has_calls',
     'attach_mock', 'call_args', 'call_args_list', 'call_count', 'called', 'configure_mock',
     'method_calls', 'mock_add_spec', 'mock_calls', 'reset_mock', 'return_value', 'side_effect',
     'some_attribute']

Wywołanie nie istniejącej funkcji o takiej same nazwie jak atrybut zwróci inny obiekt Mock.

.. code-block:: python

    m.some_attribute()
    <Mock name='mock.some_attribute()' id='140247621475856'>

Jak możesz zauważyć, takie obiekty są doskonałym narzędziem do naśladowania innych obiektów,
ponieważ mogą ujawnić dowolny interfejs API bez zgłaszania wyjątków. Jednak aby je wykorzystać
w testach, muszą one zachowywać się tak, jak oryginał, co oznacza że muszą zwracać
rozsądne wartości lub wykonywanie operacje.

Atrybut ``spec``
^^^^^^^^^^^^^^^^

Tworząc mock możemy podać atrybut ``spec``. Efektem jego działanie jest utworzenie
obiektu Mock który będzie zawierał takie same metody, właściwości jak wskazany obiekt.
Taki obiekt mock, nie może fałszować dodatkowych atrybutów, które nie znajdują się
w klasie na podstawie której został zbudowany. Warto zwrócić uwagę na fakt, że mock
stworzony na podstawie klasy, która implementuje atrybuty wewnątrz swoich funkcji np.
funkcji `__init__` nie są dostępne w samym obiekcie.


.. code-block:: python

    >>> class MySuperClass:
    ...     def __init__(self, x=0, y=0):
    ...         self.x = x
    ...         self.y = y
    ...     def get_max():
    ...         return max(x, y)
    >>> m = mock.Mock(spec=MySuperClass)
    >>> m.some_attribute.side_effect = lambda x: print(x + 45)
    AttributeError: Mock object has no attribute 'some_attribute'
    >>> m.get_max()
    <Mock name='mock.get_max()' id='4617034216'>
    >>> m.x
    AttributeError: Mock object has no attribute 'x'


.. code-block:: python

    class A:
        SPECIAL = 1

        def get_special(self):
            return self.SPECIAL

        def set_special(self, value):
            self.SPECIAL = value

    >>> m2 = mock.Mock(spec=A)
    >>> dir(m2)
    ['SPECIAL', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__',
    '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__',
    '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__',
    '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__',
    '__subclasshook__', '__weakref__', 'assert_any_call', 'assert_called', 'assert_called_once',
    'assert_called_once_with', 'assert_called_with', 'assert_has_calls', 'assert_not_called',
    'attach_mock', 'call_args', 'call_args_list', 'call_count', 'called', 'configure_mock',
    'get_special', 'method_calls', 'mock_add_spec', 'mock_calls', 'reset_mock',
    'return_value', 'set_special', 'side_effect']
    >>> m2.get_other()
    AttributeError ...


Atrybut ``return_value``
^^^^^^^^^^^^^^^^^^^^^^^^

Jest to atrybut dzięki któremu określamy jaka powinna zostać zwrócona wartość dla
wywoływanego atrybutu lub metody.

.. code-block:: python

    >>> m.some attribute.return_value = 42
    >>> m.some attribute()
    42

Również tworząc nowy obiekt możemy podać parametr ``return_value``. Dzięki któremu,
wywołanie danego mocka zpowoduje zwrócenie konkretnej wartości.

.. code-block:: python

    >>> special_class = MySpeciallClass()
    >>> special_class.my_method = mock.Mock(return_value=3)
    >>> special_class.my_method(2, 4)
    3
    >>> m.variable.assert_called_with(2, 4)
    >>> m.variable.assert_called_with(2, 4, 6)
    ...
    AssertionError: Expected call: variable(2, 4, 6)
    Actual call: variable(2, 4)


Należy pamiętać, że przypisująć do ``return_value`` konkretną funkcję zostanie zwrócony jej
obiekt a sama funkcja nie zostanie wywołana.

.. code-block:: python

    >>> def print_answer():
    ...  print("42")
    ...
    >>>
    >>> m.some_attribute.return_value = print_answer
    >>> m.some_attribute()
    <function print_answer at 0x7f8df1e3f400>

Aby zwrócić wartość funkcji musimy wykorzystać inny atrybut ``side_effect``.


Atrybut ``side_effect``
^^^^^^^^^^^^^^^^^^^^^^^

Jest atrybutem który akceptuje trzy różne wartości obiektów:
* obiekty wywoływalne (callable)
* obiekty iterowalne (iterable)
* wyjątki (exceptions)

.. code-block:: python

    >>> m.some_attribute.side_effect = ValueError('A custom value error')
    >>> m.some_attribute()
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/usr/lib/python3.4/unittest/mock.py", line 902, in __call__
        return _mock_self._mock_call(*args, **kwargs)
      File "/usr/lib/python3.4/unittest/mock.py", line 958, in _mock_call
        raise effect
    ValueError: A custom value error

Podając jako wartość listę, krotkę lub obiekt podobny to przy każdym wywołaniu tej metody
zostanie zwrócony kolejna wartość znajdująca się w obiekcie iterowalnym.

.. code-block:: python

    >>> m.some_attribute.side_effect = range(2)
    >>> m.some_attribute()
    0
    >>> m.some_attribute()
    1
    >>> m.some_attribute()
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/usr/lib/python3.4/unittest/mock.py", line 902, in __call__
        return _mock_self._mock_call(*args, **kwargs)
      File "/usr/lib/python3.4/unittest/mock.py", line 961, in _mock_call
        result = next(effect)
    StopIteration

Ostatnią i najważniejszą możliwością jest oczywiście wywołanie obiektu `callable``. Można
również ustawić konkretną klasę, a wywołanie takiej metody spowoduje utworzenie obiektu.

.. code-block:: python

    >>> def print_answer():
    ...     print("42")
    >>> m.some_attribute.side_effect = print_answer
    >>> m.some_attribute.side_effect()
    42

.. code-block:: python

    >>> m.some_attribute.side_effect = lambda x: print(x)
    >>> m.some_attribute.side_effect(45)
    45

.. code-block:: python

    >>> class MyObject:
    ...     def __repr__(self):
    ...        return '<MyObject object at {}>'.format(id(self))
    ...     def get_only_id(self):
    ...         print(id(self))
    >>> m.some_attribute.side_effect = MyObject
    >>> m.some_attribute()
    <MyObject object at 4622375904>
    >>> m.some_attribute().get_only_id()
    4622375904

Tworząc nowy mock również możemy ustawić wartość ``side_effect`` dzięki której wywołanie
takiego moka spowoduje np. wyrzucenie wyjątku, lub przeliczenie konkretnej wartości.

.. code-block:: python

    >>> special_class = MySpeciallClass()
    >>> special_class.my_method = mock.Mock(side_effect=lambda x: x * 10)
    >>> special_class.my_method(10)
    100
    >>> special_class.my_method(10, 10)
    TypeError: <lambda>() takes 1 positional argument but 2 were given


Mock vs MagicMock
^^^^^^^^^^^^^^^^^
MagicMock jest podklasą klasy Mock.

.. code-block:: python

    class MagicMock(MagicMixin, Mock)

W rezultacie MagicMock zapewnia wszystko, co zapewnia Mock oraz jak można się spodziewać potrafi nieco więcej.
Zamiast myśleć o Mocku jako o uboższej wersji MagicMocka, pomyśl o MagicMock jako rozszerzonej wersji Mock.
To powinno odpowiedzieć na pytanie o to, dlaczego Mock istnieje i co zapewnia Mock a co MagicMock.

Jedną i najważniejszą różnicą jest fakt, że MagicMock zapewnia tworzenie "magicznych" metod
pythona jeśli są one potrzebne. Poprzez magiczne metody rozumiemy wszystkie metody interfejsu
zawierające podwójne podkreślenie w swojej nazwie (np. ``__init__``, ``__len__`` itd.)

.. note::

    https://docs.python.org/3/library/unittest.mock.html#magicmock-and-magic-method-support


.. code-block:: python

    >>> int(Mock())
    TypeError: int() argument must be a string or a number, not 'Mock'
    >>> int(MagicMock())
    1
    >>> len(Mock())
    TypeError: object of type 'Mock' has no len()
    >>> len(MagicMock())
    0

Możesz "zobaczyć" metody dodane do MagicMock, ponieważ metody te są wywoływane po raz pierwszy:


.. code-block:: python

    >>> magic1 = MagicMock()
    >>> dir(magic1)
    ['assert_any_call', 'assert_called_once_with', ...]
    >>> int(magic1)
    1
    >>> dir(magic1)
    ['__int__', 'assert_any_call', 'assert_called_once_with', ...]
    >>> len(magic1)
    0
    >>> dir(magic1)
    ['__int__', '__len__', 'assert_any_call', 'assert_called_once_with', ...]

Dlaczego więc nie używać MagicMock przez cały czas? Postaram się postawić inne pytanie:
Czy rzeczywiście potezebujemy domyślnych implementacjami metod magicznych?
Przykład? Czy wywołanie indeksu na obieknie ``mocked_object[1]`` rzeczywiście powinno
zwrócić wartość zamiast błędu? Czy możesz zaakceptować wszystkie niezamierzone
konsekwencje z powodu zastosowania automatycznie utworzonych metod magicznych?
Jeśli odpowiedź na te pytania brzmi "tak", możesz korzystać z ``MagicMock``.
W przeciwnym razie korzystaj z ``Mock``.


.. code-block:: python

    def test_setup():
        external_obj = mock.Mock()
        obj = myobj.MyObj(external_obj)
        obj.setup()
        external_obj.setup.assert_called_with(cache=True, max_connections=256)


Specialne metody i atrybuty obiektu
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* `called`_ — zwraca wartość ``True`` / ``False`` pokazując czy obiekt był wywołany
* `call_count`_ — zwraca wartość ilość wywołań obiektu
* `call_args`_ — zwraca argumenty z ostatniego wywołania
* `call_args_list`_ — zwraca listę wywołań
* `method_calls`_ — zwraca ścieżkę wywołań metod i atrybutów oraz ich metod i atrybutów
* `mock_calls`_ — zwraca zapis wywołań do symulowanego obiektu, jego metod, atrybutów i zwracanych wartości
* `attach_mock`_ - pozwala dołączyć do obiektu nowy atrybut, metodę
* `configure_mock`_ - pozwala skonfigurować wartości obiektu poprzez wykorzystanie słownika
* `mock_add_spec`_ - pozwala na podstawie stringu lub obiektu ustawić wartości dla obiektu
* `reset_mock`_ - resetuje wartości wywołania obiektu
* `return_value`_ - zwraca jedną wartość niezależnie czy wywołamy ją jako zmienną czy metodę
* `side_effect`_ - zwraca wywołanie funkcji, przekazanie list zwraca po każdym elemencie

.. _`called`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.called
.. _`call_count`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.call_count
.. _`call_args`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.call_args
.. _`call_args_list`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.call_args_list
.. _`method_calls`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.method_calls
.. _`mock_calls`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.mock_calls
.. _`attach_mock`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.attach_mock
.. _`configure_mock`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.configure_mock
.. _`mock_add_spec`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.mock_add_spec
.. _`reset_mock`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.reset_mock
.. _`return_value`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.return_value
.. _`side_effect`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.side_effect


Specialne aseracje dostępne w obiekcie
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

W testach jednostkowych powszechnie stosowane są aseracje. Aby to poprawić komfort pracy
biblioteka ``mock`` zawiera wbudowane funkcje asercji, które odwołują się do wyżej
wymienionych atrybutów:

* `assert_called`_ - sprawdzenie czy ``Mock`` kiedykolwiek został wywołany
* `assert_called_once`_ - sprawdzenie czy ``Mock`` został wywołany dokładnie jeden raz
* `assert_called_with`_ - sprawdzenie konkretne argumenty użyte w ostatnim wywołaniu ``Mock``
* `assert_called_once_with`_ - sprawdzenie czy konkretne argumenty są używane dokładnie jeden raz w ``Mock``
* `assert_any_call`_ - sprawdzenie czy konkretne argumenty zostały używane w każdym wywołaniu ``Mock``
* `assert_has_calls`_ - tak samo jak ``any_call`` ale z wieloma wywołaniami ``Mock``
* `assert_not_called`_ - sprawdzenie czy ``Mock`` nigdy nie został wywołany

.. _`assert_called`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_called
.. _`assert_called_once`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_called_once
.. _`assert_called_with`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_called_with
.. _`assert_called_once_with`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_called_once_with
.. _`assert_any_call`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_any_call
.. _`assert_has_calls`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_has_calls
.. _`assert_not_called`: https://docs.python.org/3/library/unittest.mock.html#unittest.mock.Mock.assert_not_called


Jak działa Patch?
-----------------

Mocki można bardzo prosto wprowadzić do testów w przypadku gdy obiekty przyjmują klasy
lub instancje z zewnątrz. Wystarczy utworzyć instancję klasy Mock i przekazać ją jako
obiekt do systemu. Jednakże, gdy utworzony kod wykorzystuje wewnątrz inn moduły które
są zaszyte w kodzie, takie proste przekazanie obiektu Mock nie zadziała. W takich
przypadkiach pomaga nam `patch` obiektu.

Patch oznacza zastąpienie obiektu wywoływalnego wewnątrz kodu. Dzięki temu możemy
fałszować obiekty będące zaszyte w kodzie, nie modyfikując samego kodu. Patchowanie jest
wykonywane w czasie wykonywania testu.

W jaki sposób tworzyć patch?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Domyślnie tworzymy patch nie w miejscu deklaracji funkcji lub klasy ale w miejscu gdzie
została ona użyta. Poniższy przykład pokazuje tę zależność.

.. code-block:: python

    # models.py
    class SpecialModel:
        pass

.. code-block:: python

    #services.py
    from models import SpecialModel
    my_model = SpecialModel()

.. code-block:: python

    @patch('services.SpecialModel')
    def test_patch_pony(mockspecialmodel):
        mockspecialmodel.return_value = 42


Jednakże jeśli nie importujemy samej klasy tylko moduł ``import models`` sam sposób
tworzenia patch wygląda nieco inaczej.

.. code-block:: python

    # models.py
    class SpecialModel:
        pass

.. code-block:: python

    #services.py
    import models
    my_model = models.SpecialModel()

.. code-block:: python

    @patch('models.SpecialModel')
    def test_patch_pony(mockspecialmodel):

Więcej szczegułów znajdziemy w dokumentacji https://docs.python.org/3/library/unittest.mock.html#where-to-patch.


Jak działa ``autospec``?
^^^^^^^^^^^^^^^^^^^^^^^^

Efekt jego wykorzystania jest bardzo podobny do atrybutu ``spec`` podczas tworzenia Mock.
Tworząc patch ``patch_A`` z klasy ``A`` będzie on miał takie same metody czy atrybuty
jak klasa ``A``. Wykorzystując ``autospec`` nie można fałszować żadnych innych atrybutów,
które nie są zdefiniowane w rzeczywistej klasie.

``autospec`` można wywołać na dwa sposoby: ``autospec=True`` lub ``autospec=some_object``.
Podanie wartości ``True`` będzie na tworzyć Mock z dokładnymi parametrami na podstawie
patchowanej klasy/funkcji. Podanie wartości konkretnego obiektu utworzy nam taki właśnie
obiekt.

.. code-block:: python

    pass


Proste testowanie z Mock
------------------------

Celem metod dostarczanych przez pozorowane obiekty jest umożliwienie nam sprawdzenia,
jakie metody wywoływaliśmy na próbce i jakie parametry wykorzystaliśmy w wywołaniu.

.. note::

    Według Sandy Metza musimy przetestować tylko trzy typy komunikatów (połączeń) między obiektami:

    * Przychodzące zapytania (asercja na wynik)
    * Polecenia przychodzące (asercja na bezpośrednich publicznych efektach ubocznych)
    * Polecenia wychodzące (oczekiwanie na połączenie i argumenty)


Pierwszą rzeczą jaką chcemy przetestować jest sprawdzenie czy została wywołana jakaś metoda.
Aby tego dokonać wykorzystujemy jedną z specjalnych metod. Utworzyliśmy klasę która jako
argument przyjmuje obiekt który nawiazuje połączenie poprzez metodę `connect`. Również
posidamy drugą metodę `setup`, która będzie ustawiać odpowiednie argumenty dla naszego obiektu.


.. code-block:: python

    class MyObj():
        def __init__(self, repo):
            self._repo = repo
            repo.connect()

        def setup(self):
            self._repo.setup(cache=True, max_connections=256)

W pierwszym teście sprawdzimy czy podczas utworzenie obiektu klasy `MyObj` zostało nawiązane
połączenie - czyli czy została wywołana metoda `connect`. Obiekt, który przekazujemy jest
mock. Wywołując metodę `assert_called_with` sprawdzamy czy dana metoda została wywołana.

.. code-block:: python

    def test_instantiation():
        external_obj = mock.Mock()
        MyObj(external_obj)
        external_obj.connect.assert_called_with()


W drugim teście sprawdzimy czy zostały przekazane odpowiednie parametry dla wywoływanej metody.
Aby to sprawdzić wykorzystujemy inną metodę specialną `assert_called_with`.

.. code-block:: python

    def test_setup():
        external_obj = mock.Mock()
        obj = MyObj(external_obj)
        obj.setup()
        external_obj.setup.assert_called_with(cache=True, max_connections=256)


Moduły powiązane
----------------

.. code-block:: python

    import os

    class UnixFS:

        @staticmethod
        def rm(filename):
            os.remove(filename)

    def test_unix_fs(mocker):
        mocker.patch('os.remove')
        UnixFS.rm('file')
        os.remove.assert_called_once_with('file')


.. code-block:: python

    def test_foo(mocker):
        mocker.patch('os.remove')
        mocker.patch.object(os, 'listdir', autospec=True)
        mocked_isfile = mocker.patch('os.path.isfile')


    def test_create_mock(mocker):
        request.user = mocker.Mock(User)





def get_example():
    r = requests.get('http://example.com/')
    return r.status_code == 200

@mock.patch('requests.get', autospec=True)
def test_get_example_passing(mocked_get):
   mocked_req_obj = mock.Mock()
   mocked_req_obj.status_code = 200
   mocked_get.return_value = mocked_req_obj
   assert(get_example())

   mocked_get.assert_called()
   mocked_get.assert_called_with('http://example.com/')

