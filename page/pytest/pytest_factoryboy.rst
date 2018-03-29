=================
Pytest FactoryBoy
=================

`Factory Boy`_ jest narzędziem tworzącym fabryki dla obiektów, co oznacza, że nie musimy ręcznie
tworzyć potrzebnych obiektów do testów, ale możemy je wygenerować od razu w podanej ilości
w bardzo prosty sposób. Możemy ustawiać na tworzonych obiektach własne wartości tylko dla
zmiennych które chcemy przetestować.

`pytest-factoryboy`_ jest dodatkiem do ``pytest`` pozwalającym na wykorzystanie fabryk jako
fixture. Pozwala również na wykorzystanie markera ``parametrize`` do parametryzowania tworzonych fabryk.
Dzięki takiemu rozwiązaniu nie musimy importować do naszych testów fabryk a wykorzystując
odpowiednią składnię możemy importować gotowe obiekty utworzone w bazie danych.

Instalacja
----------

.. code-block:: bash

    $ pip install pytest-factoryboy



.. attention::

    Warto zainstalować najnowszą wersję 2.0.1. Obecnie tylko ta wersja zapewnia wsparcie dla najnowszej wersji factoryboy 2.10.0


Daklarowanie i tworzenie fabryk
-------------------------------

Tworzenie fabryki polega na utworzeniu klasy która odzwierciedla pola klasy dla której tworzymy
fabrykę. Ważne jest aby w klasie ``Meta`` w atrybucie ``model`` zdefiniować dla jakiego
modelu budujemy fabrykę.


.. code-block:: python

    import factory
    from . import models

    class UserFactory(factory.Factory):
        first_name = 'John'
        last_name = 'Doe'
        admin = False

        class Meta:
            model = models.User

    # Another, different, factory for the same object
    class AdminFactory(factory.Factory):
        first_name = 'Admin'
        last_name = 'User'
        admin = True

        class Meta:
            model = 'user.User'


Nową fabryję możemy utworzyć na 3 sposoby:

.. code-block:: python

    # Zwrócenie instancji, która nie jest zapisana
    user = UserFactory.build()

    # Zwrócenie instancji, która została zapisana
    user = UserFactory.create()

    # Zwrócenie obiektu stub (tylko kilka atrybutów)
    obj = UserFactory.stub()

Możemy również utworzyć kilka obiektów:

.. code-block:: python

    # Zwrócenie 10 instancji, które nie są zapisane
    users = UserFactory.build_batch(10, first_name="Joe")

    # Zwrócenie 10 instancji które zostały zapisane w bazie danych
    users = UserFactory.create_batch(10, first_name="Joe")

    # Zwrócenie 10 obiektu stub posiadających tylko kilka atrybutów
    users = UserFactory.stub_batch(10, first_name="Joe")


Typy tworzonych pól
-------------------

FactoryBoy zawiera duża liczbę typów pól dzięki którym możemy przygotować fabrykę która,
w odpowiedni sposób będzie generować obiekty.

Faker
^^^^^

Aby łatwo zdefiniować realistycznie wyglądające fabryki, najczęściej wykorzystywany zostaje atrybutu Faker.
Działanie tego atrybutu jest bardzo proste, jako pierwszy argument podajemy funkcję modułu
Faker http://faker.readthedocs.io/en/master/providers.html

Przykładowo z modułu ``faker.providers.person`` wybieramy funkcję ``name``.
Jako dodatkowy argument możemy podać język w jakim ma zostać utworzony atrybut.

.. code-block:: python

    class UserFactory(factory.Factory):
        class Meta:
            model = models.User

        username = factory.Faker('name', locale='pl_PL')

Z modułu ``faker.providers.lorem`` wybierając funckję ``paragraph`` możemy jako argument
przekazać dodatkowe parametry.

.. code-block:: python

    class UserFactory(factory.Factory):
        class Meta:
            model = models.User

        about_me = factory.Faker('paragraph', nb_sentences=3, variable_nb_sentences=True, locale='pl_PL')


Słownik
^^^^^^^

Jeśli nasze pole oczekuje słownika możemy je utworzyć w poniższy sposób. Chcąc odwołać się
do atrybutów obiektu musimy wpisać ``..is_superuser``.

.. code-block:: python

    class UserFactory(factory.Factory):
        class Meta:
            model = User

        is_superuser = False
        roles = factory.Dict({
            'role1': True,
            'role2': False,
            'role3': factory.Iterator([True, False]),
            'admin': factory.SelfAttribute('..is_superuser'),
        })


Lista
^^^^^

Możemy również utworzyć listę. Wewnętrznie, pola są konwertowane na `indeks=wartość`,
co umożliwia zastąpienie niektórych wartości w czasie ich użycia.

.. code-block:: python

    class UserFactory(factory.Factory):
        class Meta:
            model = User

        flags = factory.List([
            'user',
            'active',
            'admin',
        ])

.. code-block:: python

    >>> u = UserFactory(flags__2='superadmin')
    >>> u.flags
    ['user', 'active', 'superadmin']


Sekwencje
^^^^^^^^^

Jeśli pole ma posiadać unikalny klucz, każdy obiekt generowany przez fabrykę powinien
mieć inną wartość dla tego pola. Aby osiągnąć taki efekt wykorzystujemy deklarację sekwencji:

.. code-block:: python

    class UserFactory(factory.Factory):
        class Meta:
            model = models.User

        username = factory.Sequence(lambda n: 'user%d' % n)

Jeśli jes ona bardziej skomplikowana można ją również zapisać w poniższy sposób.

.. code-block:: python

    class UserFactory(factory.Factory):
        class Meta:
            model = models.User

        @factory.sequence
        def username(n):
            return 'user%d' % n

Każde wywołanie obiektu wygeneruje nam nowy niepowtarzalny atrybut.

.. code-block:: python

    >>> UserFactory()
    <User: user0>
    >>> UserFactory()
    <User: user1>


Maybe
^^^^^

Czasami sposób budowania danego pola może zależeć od wartości innego, na przykład parametru.
W takich przypadkach można użyj deklaracji ``Maybe``: przyjmuje nazwę pola "decydującego" oraz dwie deklaracje.
w zależności od wartości pola, którego nazwa jest przechowywana w parametrze "decydującym",
zastosuje efekty jednej lub drugiej deklaracji.

.. code-block:: python

    class UserFactory(factory.Factory):
        class Meta:
            model = User

        is_active = True
        deactivation_date = factory.Maybe(
            'is_active',
            yes_declaration=None,
            no_declaration=factory.fuzzy.FuzzyDateTime(timezone.now() - datetime.timedelta(days=10)),
        )

.. code-block:: python

    >>> u = UserFactory(is_active=True)
    >>> u.deactivation_date
    None
    >>> u = UserFactory(is_active=False)
    >>> u.deactivation_date
    datetime.datetime(2017, 4, 1, 23, 21, 23, tzinfo=UTC)


LazyFunction
^^^^^^^^^^^^

W prostych przypadkach wywołanie funkcji wystarcza aby utworzyć wartości dla pól.
Jeśli ta funkcja nie zależy od budowanego obiektu, najlepiej użyć LazyFunction, aby
wywołać tę funkcję. LazyFunction otrzymuje funkcję, która nie przyjmuje żadnych argumentów.

.. code-block:: python

    class LogFactory(factory.Factory):
        class Meta:
            model = models.Log

        timestamp = factory.LazyFunction(datetime.now)

.. code-block:: python

    >>> LogFactory()
    <Log: log at 2016-02-12 17:02:34>

    >>> # The LazyFunction can be overriden
    >>> LogFactory(timestamp=now - timedelta(days=1))
    <Log: log at 2016-02-11 17:02:34>


LazyAttribute
^^^^^^^^^^^^^

Gdy mamy sytuację w której nasze pole jest zależne od innych najlepiej wykorzystać LazyAttribute.
Dobrym przykładem może być generowanie adresu e-mail w oparciu o nazwię użytkownika.

.. code-block:: python

    class UserFactory(factory.Factory):
        class Meta:
            model = models.User

        username = factory.Sequence(lambda n: 'user%d' % n)
        email = factory.LazyAttribute(lambda obj: '%s@example.com' % obj.username)

Jeśli posiadamy bardziej rozbudowaną logikę możemy wykorzystać dekorator

.. code-block:: python

    class UserFactory(factory.Factory):
        class Meta:
            model = models.User

        username = factory.Sequence(lambda n: 'user%d' % n)

        @factory.lazy_attribute
        def email(self):
            return '%s@example.com' % self.username

.. code-block:: python

    >>> UserFactory()
    <User: user1 (user1@example.com)>

    >>> # The LazyAttribute handles overridden fields
    >>> UserFactory(username='john')
    <User: john (john@example.com)>

    >>> # They can be directly overridden as well
    >>> UserFactory(email='doe@example.com')
    <User: user3 (doe@example.com)>


FileField
^^^^^^^^^

Specialnie dla modelu Django został przygotowany atrybut ``factory.django.FileField``.
Pozwala on na utworzenie pliku dla generowanej fabryki.

.. code-block:: python

    class MyFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.MyModel

        the_file = factory.django.FileField(filename='the_file.dat')

.. code-block:: python

    >>> MyFactory(the_file__data=b'uhuh').the_file.read()
    b'uhuh'
    >>> MyFactory(the_file=None).the_file
    None


ImageField
^^^^^^^^^^

Istnieje również atrybut ``django.db.models.ImageField`` pozwalający na tworzenie obrazków.

.. code-block:: python

    class MyFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.MyModel

        the_image = factory.django.ImageField(color='blue')

.. code-block:: python

    >>> MyFactory(the_image__width=42).the_image.width
    42
    >>> MyFactory(the_image=None).the_image
    None


Non-kwarg arguments
^^^^^^^^^^^^^^^^^^^

Niektóre klasy pobierają najpierw kilka `non-kwarg` argumentów.
Taki typ pola można obsłużyć za pomocą atrybutu inline_args.

.. code-block:: python

    class MyFactory(factory.Factory):
        class Meta:
            model = MyClass
            inline_args = ('x', 'y')

        x = 1
        y = 2
        z = 3

.. code-block:: python

    >>> MyFactory(y=4)
    <MyClass(1, 4, z=3)>


Parametry
^^^^^^^^^

Jeśli tworzone pole jest zależne od atrybutu nie będącego polem w rzeczywistym modelu
tworzonym przez fabrykę należy wykorzystać deklarację Paramtru.

.. code-block:: python

    class RentalFactory(factory.Factory):
        class Meta:
            model = Rental

        begin = factory.fuzzy.FuzzyDate(start_date=datetime.date(2000, 1, 1))
        end = factory.LazyAttribute(lambda o: o.begin + o.duration)

        class Params:
            duration = 12


.. code-block:: python

    >>> RentalFactory(duration=0)
    <Rental: 2012-03-03 -> 2012-03-03>
    >>> RentalFactory(duration=10)
    <Rental: 2008-12-16 -> 2012-12-26>


Cechy
^^^^^

Jeśli natomiast wiele pól ma zostać zaktualizowanych na podstawie flagi należy
wykorzystać deklarację Cechy.

.. code-block:: python

    class OrderFactory(factory.Factory):
        status = 'pending'
        shipped_by = None
        shipped_on = None

        class Meta:
            model = Order

        class Params:
            shipped = factory.Trait(
                status='shipped',
                shipped_by=factory.SubFactory(EmployeeFactory),
                shipped_on=factory.LazyFunction(datetime.date.today),
            )

.. code-block:: python

    >>> OrderFactory()
    <Order: pending>
    >>> OrderFactory(shipped=True)
    <Order: shipped by John Doe on 2016-04-02>


Fabryki w Django
----------------

Wszystkie fabryki modelu ``Django`` powinny używać klasy bazowej ``DjangoModelFactory``.
Jeśli zachodzi potrzeba utworzenia całkiem nie standardowej fabryki warto skorzystać z
dokumentacji FactoryBoy https://factoryboy.readthedocs.io/en/latest/recipes.html


Deklarowanie fabryk
^^^^^^^^^^^^^^^^^^^

Deklaracja przebiega w dokładnie taki sam sposób jak tworzenie fabryki z prostej klasy.
Dziedzicząc jednak z DjangoModelFactory otzymujemy do ustawień 2 dodatkowe paramtery.
``django_get_or_create`` oraz ``database``. Pierwszy z nich pokreśla w jaki sposób mają
zostać tworzone obiekty a drugi określa jakie bazy danych chcemu używać.

.. code-block:: python

    class UserFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = 'myapp.User'  # Equivalent to ``model = myapp.models.User``
            django_get_or_create = ('username',)

        username = 'john'


.. code-block:: python

    >>> UserFactory()                   # Creates a new user
    <User: john>
    >>> User.objects.all()
    [<User: john>]

    >>> UserFactory()                   # Fetches the existing user
    <User: john>
    >>> User.objects.all()              # No new user!
    [<User: john>]

    >>> UserFactory(username='jack')    # Creates another user
    <User: jack>
    >>> User.objects.all()
    [<User: john>, <User: jack>]


Strategie tworzenia
^^^^^^^^^^^^^^^^^^^

Tworząc obiekt posiadamy tylko dwie podstawowe strategie określające w jaki sposób ma
on zostać utworzony obiekt podczas wywołania fabryki. Pierwsza z nich ``build`` tworzy
obiekt lokalnie, natomiast druga ``create`` tworzy lokalny obiekt i zapisuje go
w bazie danych.

Domyślną strategią wywołania fabryki jest ``create``, można jednak to zmienić
ustawiając atrybut strategii Meta klasy.

Podstawowe strategie to ``factory.BUILD_STRATEGY`` oraz ``factory.CREATE_STRATEGY``.

.. code-block:: python

    class ImageFactory(factory.Factory):
        # The model expects "attributes"
        form_attributes = ['thumbnail', 'black-and-white']

        class Meta:
            model = Image
            strategy = factory.BUILD_STRATEGY


Dziedziczenie fabryk
^^^^^^^^^^^^^^^^^^^^

Po zdefiniowaniu "bazowej" fabryki dla danej klasy, alternatywne wersje mogą być łatwo zdefiniowane poprzez podklasę.
Podklasowana Fabryka dziedziczy wszystkie deklaracje od rodzica i aktualizuje je własnymi deklaracjami.

.. code-block:: python

    class UserFactory(factory.Factory):
        class Meta:
            model = base.User

        firstname = "John"
        lastname = "Doe"
        group = 'users'

    class AdminFactory(UserFactory):
        admin = True
        group = 'admins'


.. code-block:: python

    >>> user = UserFactory()
    >>> user
    <User: John Doe>
    >>> user.group
    'users'

    >>> admin = AdminFactory()
    >>> admin
    <User: John Doe (admin)>
    >>> admin.group  # The AdminFactory field has overridden the base field
    'admins'


Pole ForeignKey
^^^^^^^^^^^^^^^

Jeśli atrybut jest złożonym polem (np. ForeignKey do innego modelu), należy użyć deklaracji SubFactory.

.. code-block:: python

    # models.py
    class User(models.Model):
        first_name = models.CharField()
        group = models.ForeignKey(Group)


    # factories.py
    import factory
    from . import models

    class UserFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.User

        first_name = factory.Sequence(lambda n: "Agent %03d" % n)
        group = factory.SubFactory(GroupFactory)


Jeśli wartości klucza ForeignKey muszą zostać wybrane z już wypełnionej tabeli
(np. ``django.contrib.contenttypes.models.ContentType``), należy użyć ``fabryki.Iterator``.

.. code-block:: python

    import factory, factory.django
    from . import models

    class UserFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.User

        language = factory.Iterator(models.Language.objects.all())


Odwrotne relacje ForeignKey
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Jeśli obiekt powiązany powinien zostać utworzony podczas tworzenia obiektu
(np. odwrócona relacja ForeignKey z innego Modelu), należy użyć deklaracji ``RelatedFactory``.

.. code-block:: python

    # models.py
    class User(models.Model):
        pass

    class UserLog(models.Model):
        user = models.ForeignKey(User)
        action = models.CharField()


    # factories.py
    class UserFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.User

        log = factory.RelatedFactory(UserLogFactory, 'user', action=models.UserLog.ACTION_CREATE)


Po utworzeniu instancji `UserFactory`, pole `factory_boy` wywoła
``UserLogFactory(user=that_user, action=...)`` tuż przed zwróceniem utworzonego użytkownika.


Pole ManyToMany
^^^^^^^^^^^^^^^

Zbudowanie odpowiedniego połączenia między dwoma modelami zależy w dużej mierze od
przypadku użycia. `factory_boy` niestety nie zapewnia narzędzia działającego w podobniy
sposób jak w przypadku `SubFactory` lub `RelatedFactory`, dlatego programista musi
tworzyć własne zależności od modelu. Aby utworzyć relację M2M należy wykorzystać hook
``post_generation``.

.. code-block:: python

    # models.py
    class Group(models.Model):
        name = models.CharField()

    class User(models.Model):
        name = models.CharField()
        groups = models.ManyToManyField(Group)


    # factories.py
    class GroupFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.Group

        name = factory.Sequence(lambda n: "Group #%s" % n)

    class UserFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.User

        name = "John Doe"

        @factory.post_generation
        def groups(self, create, extracted, **kwargs):
            if not create:
                # Simple build, do nothing.
                return

            if extracted:
                # A list of groups were passed in, use them
                for group in extracted:
                    self.groups.add(group)


Podczas wywoływania funkcji ``UserFactory()`` lub ``UserFactory.build()`` nie zostanie
utworzone powiązanie z grupą. Natomiast po wywołaniu ``UserFactory.create(groups=(group1, group2, group3))``
deklaracja ``groups`` doda przekazane grupy do użytkownika.

.. code-block:: python

    class ClinicFactory(factory.django.DjangoModelFactory):
        name = 'Some name'

        street = factory.Faker('street_name')
        postal_code = factory.Faker('postcode')
        place = factory.Faker('city')
        voivodship = factory.Faker('region')
        country = 'Polska'

        @factory.post_generation
        def domains(self, create, data=None, **kwargs):
            if not create:
                return

            if data is None:
                data = 1

            if isinstance(data, int):
                domain_factory = getattr(DomainFactory, 'create')
                for i in range(data):
                    self.domains.add(domain_factory())
            elif data:
                for domain in data:
                    self.domains.add(domain)

        class Meta:
            model = 'clinics.Clinic'

Innnym przykładem jest możliwość utworzenia deklaracji która będzie przyjmowała liczbę lub
obiekt iterowalny aby utworzyć obiekty powiązane. Nie podając żadnej wartości zostanie
utworzony i dołączony 1 obiekt ``DomainFactory``.


Pole ManyToMany (through)
^^^^^^^^^^^^^^^^^^^^^^^^^

Aby utworzyć relację Many2Many poprzez własną tabelę (throw) należy wykorzystać
deklarację ``RelatedFactory``.

.. code-block:: python

    # models.py
    class User(models.Model):
        name = models.CharField()

    class Group(models.Model):
        name = models.CharField()
        members = models.ManyToManyField(User, through='GroupLevel')

    class GroupLevel(models.Model):
        user = models.ForeignKey(User)
        group = models.ForeignKey(Group)
        rank = models.IntegerField()


    # factories.py
    class UserFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.User

        name = "John Doe"

    class GroupFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.Group

        name = "Admins"

    class GroupLevelFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.GroupLevel

        user = factory.SubFactory(UserFactory)
        group = factory.SubFactory(GroupFactory)
        rank = 1

    class UserWithGroupFactory(UserFactory):
        membership = factory.RelatedFactory(GroupLevelFactory, 'user')

    class UserWith2GroupsFactory(UserFactory):
        membership1 = factory.RelatedFactory(GroupLevelFactory, 'user', group__name='Group1')
        membership2 = factory.RelatedFactory(GroupLevelFactory, 'user', group__name='Group2')


Niestandardowa metoda tworząca fabrykę
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Czasami zachodzi potrzeba aby tworząc fabrykę zachowywała się ona inaczej niż domyślna
metoda Model.objects.create(). Aby uzyskać żądane zachowanie należy utworzyć własną metodę
klasy ``_create(...)``.

.. code-block:: python

    class UserFactory(factory.DjangoModelFactory):
        class Meta:
            model = UserenaSignup

        username = "l7d8s"
        email = "my_name@example.com"
        password = "my_password"

        @classmethod
        def _create(cls, model_class, *args, **kwargs):
            """Override the default ``_create`` with our custom call."""
            manager = cls._get_manager(model_class)
            # The default would use ``manager.create(*args, **kwargs)``
            return manager.create_user(*args, **kwargs)


Wyłaczanie sygnałów
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    # foo/factories.py

    import factory
    import factory.django

    from . import models
    from . import signals

    @factory.django.mute_signals(signals.pre_save, signals.post_save)
    class FooFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.Foo


.. code-block:: python

    def make_chain():
        with factory.django.mute_signals(signals.pre_save, signals.post_save):
            # pre_save/post_save won't be called here.
            return SomeFactory(), SomeOtherFactory()


Konwertowanie fabryki do słownika
---------------------------------

.. code-block:: python

    class UserFactory(factory.django.DjangoModelFactory):
        class Meta:
            model = models.User

        first_name = factory.Sequence(lambda n: "Agent %03d" % n)
        username = factory.Faker('username')

.. code-block:: python

    >>> factory.build(dict, FACTORY_CLASS=UserFactory)
    {'first_name': "Agent 001", 'username': 'john_doe'}


Inicjalizacja fabryk w pytest
-----------------------------

Funkcje dostarczane wraz z pytest-factoryboy pozwalają na używanie fabryk bez ich importowania.
Konwencja wykorzystywana do uruchamiania fixture z zarejestrowanej klasy wykorzystuj podkreślenia i małe litery.
Najlepszym miejscem rejestracji fabryki jest plik ``conftest.py``.

.. code-block:: python

    # tests/factories.py
    import factory

    class AuthorFactory(factory.Factory):

        class Meta:
            model = Author

    class GroupForSuperUserFactory(factory.Factory):

        class Meta:
            model = Group


    # tests/conftest.py
    from pytest_factoryboy import register
    from .factories import AuthorFactory, GroupForSuperUserFactory

    register(AuthorFactory)
    register(GroupForSuperUserFactory)


    # tests/test_models.py
    def test_factory_fixture(author_factory):
        author = author_factory(name="Charles Dickens")
        assert author.name == "Charles Dickens"

    def test_factory_fixture(group_for_super_user_factory):
        author = group_for_super_user_factory(name="Super Group")
        assert author.name == "Super Group"


Istnieje również możliwość rejestracji modelu pod określoną nazwą wraz z ustawionymi parametrami.


.. code-block:: python

    register(BookFactory)  # book
    register(BookFactory, "second_book")  # second_book

    register(AuthorFactory) # author
    register(AuthorFactory, "second_author") # second_author

    register(AuthorFactory, "male_author", gender="M", name="John Doe")
    register(AuthorFactory, "female_author", gender="F")

    register(BookFactory, "other_book")  # other_book, book of another author

    @pytest.fixture
    def other_book__author(second_author):
        """
        Make the relation of the second_book to another (second) author.
        """
        return second_author

    @pytest.fixture
    def female_author__name():
        """Override female author name as a separate fixture."""
        return "Jane Doe"


Fabryki w testach
-----------------

Wykorzystująć fabryki w testach mamy możliwość w dwojaki sposób wykorzystania
zarejestrowanego fixture. Pierwszy do podanie pełnej nazwy klasy w konwencji małe litery
oraz podkreśleniem np. mając fabrykę ``GroupForSuperUserFactory`` należy utworzyć fixture
``group_for_super_user_factory``. W teście będzie to obiekt fabryki, który należy najpierw
wywołać aby utworzyć obiekt z właściwymi wartościami.

.. code-block:: python

    def test_factory_fixture(group_for_super_user_factory):
        assert isinstance(group_for_super_user, GroupForSuperUserFactory)
        author = group_for_super_user_factory(name="Super Group")
        assert author.name == "Super Group"

Istnieje również druga możliwość, która pozwala na bezpośrednie utworzenie modelu w teście
bez tworzenia fabryki. Posiłkując się powyższym przykładem, aby utworzyć model dla fabryki
``GroupForSuperUserFactory`` tworzymy fixture, jednak bez nazwy `factory`, czyli ``group_for_super_user``.

.. code-block:: python

    def test_factory_fixture(group_for_super_user):
        assert isinstance(group_for_super_user, Group)

.. code-block:: python

    from app.models import Book
    from factories import BookFactory

    def test_book_factory(book_factory):
        """Factories become fixtures automatically."""
        assert isinstance(book_factory, BookFactory)

    def test_book(book):
        """Instances become fixtures automatically."""
        assert isinstance(book, Book)

    @pytest.mark.parametrize("book__title", ["PyTest for Dummies"])
    @pytest.mark.parametrize("author__name", ["Bill Gates"])
    def test_parametrized(book):
        """You can set any factory attribute as a fixture using naming convention."""
        assert book.name == "PyTest for Dummies"
        assert book.author.name == "Bill Gates"


Atrybuty w fixture
^^^^^^^^^^^^^^^^^^

Tworząc testy możemy parametryzować utworzone fabryki poprzez wykorzystanie markera ``parametrize``.
Aby uaktualnić konkretną wartość musimy wykorzystać podwójne podkreślenie wraz z nazwą pola.

.. code-block:: python

    @pytest.mark.parametrize("author__name", ["Bill Gates"])
    def test_model_fixture(author):
        assert author.name == "Bill Gates"

Czasami konieczne jest przekazanie instancji innego fixture jako wartości atrybutu do fabryki.
Możliwe jest przesłonięcie wygenerowanego urządzenia atrybutów, gdzie żądane wartości
mogą być wymagane jako zależność fixture. Istnieje również leniwy wrapper dla fixture,
które może być użyte w parametryzacji bez definiowania fixture w module.

.. code-block:: python

    import pytest
    from pytest_factoryboy import register, LazyFixture

    @pytest.mark.parametrize("book__author", [LazyFixture("another_author")])
    def test_lazy_fixture_name(book, another_author):
        """Test that book author is replaced with another author by fixture name."""
        assert book.author == another_author


    @pytest.mark.parametrize("book__author", [LazyFixture(lambda another_author: another_author)])
    def test_lazy_fixture_callable(book, another_author):
        """Test that book author is replaced with another author by callable."""
        assert book.author == another_author


    # Can also be used in the partial specialization during the registration.
    register(BookFactory, "another_book", author=LazyFixture("another_author"))


Przykłady
---------

Poniżej przykład w jaki sposób utworzyć pole własnego typu, pozwalający fabryce na generyczne
tworzenie wartości dla wskazanego pola.

.. code-block:: python

    # fuzzy_geo.py
    from factory.fuzzy import BaseFuzzyAttribute

    class FuzzyPoint(BaseFuzzyAttribute):

        def fuzz(self):
            return Point(random.uniform(-180.0, 180.0), random.uniform(-90.0, 90.0))


    # factories.py
    from .fuzzy_geo import FuzzyPoint


    class UserFactory(factory.django.DjangoModelFactory):
        ...
        last_location = FuzzyPoint()


Poniżej bardziej skomplikowany przykład pokazujący w jaki sposób możemy utworzyć fabrykę
dla użytkownika aplikacji.

.. code-block:: python

    import random
    import datetime
    import factory

    from faker import Faker
    from django.utils.text import slugify
    from ..models import User


    fake = Faker('pl_PL')


    class UserFactory(factory.django.DjangoModelFactory):
        first_name = factory.Faker('first_name')
        last_name = factory.Faker('last_name')
        username = factory.LazyAttribute(
            lambda o: slugify(o.first_name + '.' + o.last_name))
        email = factory.LazyAttribute(
            lambda o: o.username + "@" + fake.free_email_domain())
        password = factory.Faker('password', length=10)
        birthday = factory.Faker('date_between_dates',
                                 date_start=datetime.date(1960, 1, 1),
                                 date_end=datetime.date(1998, 1, 1))
        gender = factory.LazyAttribute(
            lambda o: random.choice([User.FEMALE, User.MALE]))

        notifications_enabled = True
        region = factory.Faker('region')
        city = factory.Faker('city')
        description = factory.Faker('sentences')
        level = 1
        registration_status = 2
        score = 0

        # brands = factory.LazyAttribute(lambda o: random.choice([]))
        profile_photo = 0
        instagram_url = factory.Faker('uri')

        class Meta:
            model = 'users.User'
            django_get_or_create = ('username',)

        @factory.lazy_attribute
        def date_joined(self):
            return datetime.datetime.now() - datetime.timedelta(
                days=random.randint(5, 50))

        last_login = factory.LazyAttribute(
            lambda o: o.date_joined + datetime.timedelta(days=4))

        is_staff = False
        is_active = True
        is_superuser = False


.. _`Factory Boy`: https://factoryboy.readthedocs.io/en/latest/
.. _`pytest-factoryboy`: http://pytest-factoryboy.readthedocs.io/en/latest/
