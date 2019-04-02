=============
Pytest Django
=============

``pytest-django`` jest dodatkiem do ``pytest``, udostępniającą zestaw przydatnych narzędzi
do testowania aplikacji i projektów Django. Bez tego dodatku bardzo ciężko będzie nam
przetestować architekturę django bez wykorzystywania ``mock``.


Instalacja
----------

.. code-block:: bash

    $ pip install pytest-django


Konfiguracja
------------

W katalogu głównym naszej aplikacji w pliku ``pytest.ini`` dodajemy zmienną
``DJANGO_SETTINGS_MODULE`` w której ustawiamy ścieżkę do ustawień Django.

.. code-block:: bash

    [pytest]
    DJANGO_SETTINGS_MODULE=config.settings.test

Istnieją jeszcze inne sposoby konfiguracji których opis można znaleść pod adresem:
http://pytest-django.readthedocs.io/en/latest/configuring_django.html


Uruchomienie testów
-------------------

Testy uruchamiamy w taki sam sposób jak testy w ``pytest``. Mamy natomiast możliwość
uruchomienia ``manage.py test`` a w tle uruchamiamy ``pytest``. Aby tego dokonać tworzymy
plik ``runner.py`` i w środku zamieszczamy poniższy kod:

.. code-block:: python

    class PytestTestRunner(object):
    """Runs pytest to discover and run tests."""

    def __init__(self, verbosity=1, failfast=False, keepdb=False, **kwargs):
        self.verbosity = verbosity
        self.failfast = failfast
        self.keepdb = keepdb

    def run_tests(self, test_labels):
        """Run pytest and return the exitcode.

        It translates some of Django's test command option to pytest's.
        """
        import pytest

        argv = []
        if self.verbosity == 0:
            argv.append('--quiet')
        if self.verbosity == 2:
            argv.append('--verbose')
        if self.verbosity == 3:
            argv.append('-vv')
        if self.failfast:
            argv.append('--exitfirst')
        if self.keepdb:
            argv.append('--reuse-db')

        argv.extend(test_labels)
        return pytest.main(argv)

Następnie należy ustawić zmienną ``TEST_RUNNER`` w pliku ``settings.py``.

.. code-block:: python

    TEST_RUNNER = 'my_project.runner.PytestTestRunner'


Teraz możemy uruchomić nasze testy w podobny sposób w jaki uruchamia się je normalnie w Django.

.. code-block:: bash

    $ ./manage.py test <django args> -- <pytest args>


.. attention::

    W takiej konfiguracji nie działają parametry ``--ds`` oraz ``--dc``. Należy ustawić
    te parametry w pliku ``pytest.ini`` lub wykorzystać zmienną ``--settings`` znaną z komend django.


Dodatkowe komendy
^^^^^^^^^^^^^^^^^

Uruchamiając nasze testy mamy możliwość utworzenia dodatkowych komend.

``--fail-on-template-vars`` dzięki której zostanie podniesiony wyjątek dla niepoprawnych zmiennych w szablonach django.

``--reuse-db`` - ponownie wykorzystanie testowej bazy danych pomiędzy kolejnymi testami.
Testowa baza danych nie zostanie usunięta, ponowne uruchomienie testu spowoduje wykorzystanie tej bazy.
Opcja ta nie będzie przechwytywać zmian schematu między testami. Można tę opcję użyć razem z
`--create-db` aby ponownie utworzyć bazę danych zgodnie z nowym schematem.

``--create-db`` - wymuszenie ponownego utworzenia testowej bazy danych, niezależnie od tego, czy istnieje, czy nie

``--nomigrations`` - spowoduje wyłączenie migracji Django

``--migrations`` - wymusi utworzenie bazy wraz z migracjami

Więcej szczegółów na temat konfiguracji i pracy z bazą danych można znaleźć pod adresem
http://pytest-django.readthedocs.io/en/latest/database.html


Markery
-------

pytest-django zapewnia kilka bardzo przydatnych markerów które można wykorzystać podczas pisania testów.
Wszystkie poniższe znaczniki można wykorzystać na funkcji lub klasie testującej.

@pytest.mark.django_db(transaction=False)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Dzięki temu markerowi uzyskujemy dostęp do bazy danych w testach. Każdy test zostanie
przeprowadzony w ramach własnej transakcji, która zostanie wycofana po zakończeniu testu.
Ustawienie zmiennej ``transaction`` na ``False`` - domyślne zachowanie - powoduje że nasz
test zachowuje się w taki sam sposób jak wykorzystanie ``django.test.TestCase``. Ustawienie
tej zmiennej na ``True`` powoduje zmianę zachowania na identyczną jak w ``django.test.TransactionTestCase``.

.. note::

    Aby uzyskać dostęp do bazy danych wewnątrz własnego ``fixture`` należy wykorzystać fixture ``db`` lub ``transactional_db``.


.. code-block:: python

    @pytest.mark.django_db
    def test_something():
        obj = MyObject.objects.get(id=1)
        assert obj.name == 'name'

    @pytest.mark.django_db
    class TestUsers:

        def test_my_user(self):
            me = User.objects.get(username='me')
            assert me.is_superuser

    class TestUsers:
        pytestmark = pytest.mark.django_db

        def test_my_user(self):
            me = User.objects.get(username='me')
            assert me.is_superuser


@pytest.mark.urls
^^^^^^^^^^^^^^^^^

Zastąpienie domyślnej konfiguracji url w django

.. code-block:: python

    @pytest.mark.urls('myapp.test_urls')
    def test_something(client):
        assert 'Success!' in client.get('/some_url_defined_in_test_urls/').content


@pytest.mark.ignore_template_errors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ignorowanie ​​niepoprawnych zmiennych w szablonie

.. code-block:: python

    @pytest.mark.ignore_template_errors
    def test_something(client):
        client('some-url-with-invalid-template-vars')


Fixtures
--------

pytest-django zapewnia kilka fixture które w znaczącym stopniu ułatwiają korzystanie
z wbudowanych w django dodatkowych narzędzi do testowania.

rf
^^

``rf`` jest instancją ``django.test.RequestFactory``, która jest wykorzystywana do pisania
testów widoków bez przechodzenia przez wszystkie middleware. Dzięki temu narzędziowi możemy
przetestować konkretną zmienną lub metodę w klasie widoku.


.. code-block:: python

    from myapp.views import my_view

    def test_details(rf):
        request = rf.get('/customer/details')
        response = my_view(request)
        assert response.status_code == 200


client
^^^^^^

``client`` jest instancją ``django.test.Client``. Można go wykorzystywać do pisania testów
integracyjnych jednak nie jest on polecany. Zamiast niego lepiej jest skorzystać z modułu ``WebTest``,
który również został opisany.

.. code-block:: python

    def test_with_client(client):
        response = client.get('/')
        assert response.content == 'Foobar'


admin_client
^^^^^^^^^^^^

``admin_client`` jest instancją ``django.test.Client`` zalogowanego jako administrator.

.. code-block:: python

    def test_an_admin_view(admin_client):
        response = admin_client.get('/admin/')
        assert response.status_code == 200


admin_user
^^^^^^^^^^

``admin_user`` jest obiektem użytkownika (administratora) utworzonego w bazie danych.
Jego nazwa to ``admin`` a hasło ``password``.


django_user_model
^^^^^^^^^^^^^^^^^

``django_user_model`` jest modelem (nie instancją) użytkownika ustawionego poprzez settings.AUTH_USER_MODEL.

.. code-block:: python

    def test_new_user(django_user_model):
        django_user_model.objects.create(username="someone", password="something")


django_username_field
^^^^^^^^^^^^^^^^^^^^^

Jest to fixture wyodrębniający nazwę pola, używaną dla nazwy użytkownika w modelu użytkownika.
Pobranie ustawienia z settings.USERNAME_FIELD.


db
^^

fixture który powinien być wykorzystywany tylko w innym fixture który wymaga dostępu do bazy danych.


transactional_db
^^^^^^^^^^^^^^^^

fixture który powinien być wykorzystywany tylko w innym fixture który wymaga dostępu do bazy danych.


live_server
^^^^^^^^^^^

Ten fixture uruchamia aplikację django w oddzielnym wątku. Dostęp do adresu url można uzyskać
poprzez komendę ``live_server.url``. Ten fixture będzie przydatny w przypadku kiedy będziemy
chcieli uruchomić testy poprzez wykorzystanie biblioteki ``selenium``.


settings
^^^^^^^^

Ten fixture zapewnia możliwość modyfikacji ustawień Django oraz automatycznie
przywróci wszelkie zmiany dokonane w ustawieniach (modyfikacje, dodatki i usunięcia).

.. code-block:: python

    def test_with_specific_settings(settings):
        settings.USE_TZ = True
        assert settings.USE_TZ


django_assert_num_queries
^^^^^^^^^^^^^^^^^^^^^^^^^

Ten fixture pozwala sprawdzić oczekiwaną liczbę zapytań do bazy danych.
Obecnie jest obsługiwana tylko domyślną baza danych.

.. code-block:: python

    def test_queries(django_assert_num_queries):
        with django_assert_num_queries(3):
            Item.objects.create('foo')
            Item.objects.create('bar')
            Item.objects.create('baz')


mailoutbox
^^^^^^^^^^

Skrzynka nadawcza wiadomości e-mail, do której wysyłane są e-maile generowane przez Django.

.. code-block:: python

    from django.core import mail

    def test_mail(mailoutbox):
        mail.send_mail('subject', 'body', 'from@example.com', ['to@example.com'])
        assert len(mailoutbox) == 1
        m = mailoutbox[0]
        assert m.subject == 'subject'
        assert m.body == 'body'
        assert m.from_email == 'from@example.com'
        assert list(m.to) == ['to@example.com']


Metoda setUp znana z UnitTest
-----------------------------

Niestety największym dotąd nie rozwiązanym problemem jest brak możliwości tworzenia
obiektów w bazie danych z wykorzystaniem metody `setup_class` a znanej z biblioteki
``unittest`` pod nazwą `setUpClass`.

`setUpClass()` ==> `setup_class`
`tearDownClass()` ==> `teardown_class`

Natomiast aby skorzystać z znanej metody `setUp()` oraz skorzystać z bazy danych
do tworzenia obiektów, należy nieco zmienić działanie.
Oba poniższe przykłady działają dokładnie tak samo.

.. code-block:: python

    class AnimalTestCase(TestCase):

        def setUp(self):
            Animal.objects.create(name="lion", sound="roar")
            Animal.objects.create(name="cat", sound="meow")

        def test_animals_can_speak(self):
            """Animals that can speak are correctly identified"""
            lion = Animal.objects.get(name="lion")
            cat = Animal.objects.get(name="cat")
            self.assertEqual(lion.speak(), 'The lion says "roar"')
            self.assertEqual(cat.speak(), 'The cat says "meow"')


.. code-block:: python

    @pytest.mark.django_db
    class AnimalTestCase:

        @pytest.fixture(autouse=True)
        def setup_method(self, db):
            Animal.objects.create(name="lion", sound="roar")
            Animal.objects.create(name="cat", sound="meow")

        def test_animals_can_speak(self):
            """Animals that can speak are correctly identified"""
            lion = Animal.objects.get(name="lion")
            cat = Animal.objects.get(name="cat")
            assert lion.speak() == 'The lion says "roar"'
            assert cat.speak() == 'The cat says "meow"'


https://stackoverflow.com/questions/34089425/django-pytest-setup-method-database-issue
https://github.com/pytest-dev/pytest-django/issues/297
