============
PyTest Faker
============

``Faker`` jest pakietem generującym fałszywe dane. Faker może się przydać aby utworzyć
obiekty bazy danych, ładny dokument XML, dane potrzebne do testów czy do anonimizacji danych
pobranych z usługi produkcyjnej.

``pytest-faker`` jest dodatkiem zapewniającym dodatkowy fixture będący instancją obiektu faker.

Instalacja
----------

.. code-block:: bash

    $ pip install pytest-faker


Wykorzystanie
-------------

Aby utworzyć dane należy utworzyć obiekt klasy Faker, a następnie wywołać jedną z dostępnych
metod na tym obiekcie. Metod jest tak dużo że warto zerknąć do dokumentacji pod adresem
http://faker.readthedocs.io/en/master/providers.html. W niej mamy utworzone grupy w których
mamy wykorzystane konkretne metody. Najczęściej używane grupy to ``faker.providers.person``,
``faker.providers.address`` czy ``faker.providers.lorem``.


.. code-block:: python

    from faker import Faker

    fake = Faker()

    fake.name()
    # 'Lucy Cechtelar'

    fake.address()
    # "426 Jordy Lodge
    #  Cartwrightshire, SC 88120-6700"

Aby wykorzystać moduł ``pytest-faker`` należy wykorzystać dostarczony fixture.

.. code-block:: python

    #tests/test_faker.py:
    from faker.generator import Generator

    def test_faker(faker):
        """Faker factory is a fixture."""
        assert isinstance(faker, Generator)
        assert isinstance(faker.name(), str)


Lokalizacja
^^^^^^^^^^^

Aby ustawić język w jakim mają zostać wygenerowane dane należy podczas inicjalizacji
obiektu Faker podać dodatkowy argument będący kodem języka. Język polski jest oznaczony
kodem ``pl_PL``.

.. code-block:: python

    from faker import Faker

    fake = Faker('it_IT')

    for _ in range(10):
        print(fake.name())

    > Elda Palumbo
    > Pacifico Giordano
    > Sig. Avide Guerra
    > Yago Amato
    > Eustachio Messina
    > Dott. Violante Lombardo
    > Sig. Alighieri Monti
    > Costanzo Costa
    > Nazzareno Barbieri
    > Max Coppola

Aby ustawić lokalizację dla modułu ``pytest-faker`` należy w pliku ``conftest.py``
nadpisać domyślny fixture ``faker_locale``, który powinien zwracać kod języka.


.. code-block:: python

    # test/conftest.py
    @pytest.fixture(scope='session')
    def faker_locale():
        return 'pl_PL'


Dostęp do losowej instancji
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Właściwość .random na generatorze zwraca instancję ``random.Random`` używaną do generowania wartości.

.. code-block:: python

    from faker import Faker

    fake = Faker()

    fake.random
    fake.random.getstate()

Domyślnie wszystkie generatory współdzielą to samo wystąpienie ``random.Random``,
do którego można uzyskać dostęp za pomocą ``from faker.generator import random``.
Używanie tego może być przydatne w przypadku wtyczek, które chcą wpływać na wszystkie
instancje fakerów.


Seeding generatora
^^^^^^^^^^^^^^^^^^

Kiedy używasz Fakera do testowania, często będziesz chciał wygenerować ten sam zestaw danych.
Dla wygody generator dostarcza również metodę ``seed()``, która zapewnia generowanie
takiego samego zestawu testowego. Wywołanie tych samych metod generatora w tej samej
wersji fakera z taką samą wartością `seed` zwróci nam takie same wyniki.

.. code-block:: python

    from faker import Faker

    fake = Faker()
    fake.seed(4321)

    print(fake.name())
    > Margaret Boehm

Więcej szczegółów można znaleźć w dokumentacji http://faker.readthedocs.io/en/master/index.html#seeding-the-generator
