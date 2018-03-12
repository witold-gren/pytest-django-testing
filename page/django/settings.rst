Ustawienia
----------

Testując aplikację lokalnie bardzo ważne jest aby testy uruchamiały się bardzo szybko,
sprawia to, że nasza uwaga jest poświęcona cały czas na pisaniu dobrego kodu. Aby
przyspieszyć wykonywanie testów w Django istnieje kilka dobrych praktyk które spowodują
że testy będą działać odczuwalnie szybciej.


Zmień hashowanie hasła
^^^^^^^^^^^^^^^^^^^^^^

Jest to najskuteczniejsze ustawienie, które można wykorzystać do poprawy szybkości testów.
Może się to wydawać śmieszne, ale hashowanie haseł w Django jest bardzo mocne, dlatego
korzysta on z kilku "hasherów", oznacza to jednak, że haszowanie jest bardzo powolne.
Najszybszym hasherem jest ``MD5PasswordHasher``, dlatego warto go użyć podczas testowania
aplikacji:

.. code-block:: python

    PASSWORD_HASHERS = (
        'django.contrib.auth.hashers.MD5PasswordHasher',
    )


Użyj szybszego systemu przechowywania danych
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Obecnie najszybszą bazą danych z której korzysta Django jest SQLite. Testujemy własną
implementację kodu, własne API i jeśli nie używamy surowych zapytań SQL,
bazowy mechanizm przechowywania danych nie powinien pokazywać różnic!

Warto więc zmienić go na silnik ``SQLite``:

.. code-block:: python

    # test_settings.py
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': ':memory:',
        }
    }

.. code-block:: python

    #if manage.py test was called, use test settings
    if 'test' in sys.argv:
        try:
            from test_settings import *
        except ImportError:
            pass

.. attention::

    Jeśli wykorzystujemy `continuous integration` nie powinniśmy podmieniać ustawień
    bazy danych. Takie środowisko powinno być najbardziej zbliżone do środowiska
    produkcyjnego.

    Również jeśli wykonujemy testy integracyjne (nie powinny one być połączone z testami
    jednostkowymi w tym samym pliku) nie powinniśmy również zmieniać ustawień bazy danych.

.. note::

    Jeśli wykorzystujemy specyficzne rozwiązania z silnika bazy danych z której korzystamy,
    możemy takować nasze testy markerami, zapewni nam to możliwość uruchomienia testów
    specyficznych dla danej bazy danych oraz do szybkie testowanie zapytań napisanych
    w Django ORM.


.. code-block:: python

    import pytest

    @pytest.mark.postgres
    class TestSpecificForPostgreSQL(TestCase):

        def test_save_json_field_from_api(self):
            ...


Warto w ustawieniach dodać brak możliwości podmiany ustawień bazy danych podczas
wykonywania testów.

.. code-block:: python

    # test_settings.py

    if not 'not postgres' in sys.argv:
        DATABASES = {
            'default': {
                'ENGINE': 'django.db.backends.sqlite3',
                'NAME': ':memory:',
            }
        }

Uruchomienie testów jest bardzo proste. Wystarczy w testach podać atrybut uruchamiający
wszystkie testy poza testami z markerem ``postgres``.

.. code-block:: bash

    $ pytest -v -m "not postgres"


Innym sposobem na rozwiązanie tego problemu jest napisanie nakładek na specyficzne pola
dla danego silnika bazy danych. Niestety nie miałem z tym większej styczności dlatego
przekierowuję do jednego z artykułów.

https://www.aychedee.com/2014/03/13/json-field-type-for-django/


Usuń niepotrzebne middleware
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Im więcej klas middleware, tym więcej czasu będzie potrzebne na wygenerowanie odpowiedzi (ponieważ
wszystkie warstwy pośredniczące muszą być wykonywane sekwencyjnie przed zwróceniem ostatecznej
odpowiedzi HTTP). Warto więc uruchomić tylko te warstwy których tak naprawdę potrzebujesz!

Szczególnie jeden middleware jest bardzo wolny:

.. code-block:: python

    django.middleware.locale.LocaleMiddleware

Możemy założyć, że wszystkie middleware z Django działają poprawnie, dlatego podczas
testowania możemy je usunąć, aby uniknąć wszystkich narzutów podczas wysyłania żądań.

.. code-block:: python

    MIDDLEWARE_CLASSES = [
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
    ]


Usuń niepotrzebne aplikacje
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Istnieje kilka aplikacji, które można usunąć podczas testowania, np. ``django-debug-toolbar``
czy ``django_extension`` spróbuj usunąć wszystkie nieużywane/niepotrzebne aplikacje podczas
wykonywania testów.


Wyłącz debugowanie
^^^^^^^^^^^^^^^^^^

Ustawienie parametru ``DEBUG=False`` podczas uruchamiania testów zmniejsza obciążenie
związane z debugowaniem, dzięki czemu poprawia się szybkość wykonywania testów.

.. code-block:: python

    DEBUG = False


Wyłącz informacje o logach
^^^^^^^^^^^^^^^^^^^^^^^^^^

Jest to znacząca modyfikacja tylko wtedy, gdy mamy ogromną ilość logowań i/lub dodatkowej
logiki związanej z logami (np. inspekcje obiektów, ciężkie manipulacje ciągami itd.).
Logowanie również jest niepotrzebne podczas wykonywania testów, dlatego nie ma potrzeby
dodawania dodatkowego narzutu pliku I/O do pakietu testowego.

.. code-block:: python

    import logging
    logging.disable(logging.CRITICAL)


Użyj szybszego zaplecza e-mail
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Domyślnie Django używa ``django.core.mail.backends.locmem.EmailBackend``, który jest
backendem przeznaczonym do testowania w pamięci, jednak czasem mogą z nim wystąpić problemy
z powodu sprawdzanie nagłówków. Warto więc skorzystąć z alternatywnego backendu mailowego.

.. code-block:: python

    EMAIL_BACKEND = "django.core.mail.backends.dummy.EmailBackend"


Używaj Celery uruchamianego w pamięci
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Jeśli wykorzystujesz Celery w swoich projektach warto zmienić ustawienia do testowania:

.. code-block:: python

    CELERY_ALWAYS_EAGER = True
    CELERY_EAGER_PROPAGATES_EXCEPTIONS = True
    BROKER_BACKEND = 'memory'


Mock, mock, mock!
^^^^^^^^^^^^^^^^^

Wykorzystując ``Mock`` możesz znacznie skrócić czas testowania swoich aplikacji.
Obiekty Mock można używać podczas każdych testów, najeży jednak pamiętać aby nie tworzyć
mocków do bazy danych jeśli nie posiadamy testów integracyjnych. Więcej szczegułów
na temat tworzenia ``Mock`` znajdziesz w module ``pytest-mock``.


Dodatkowe opcje
---------------

Domyślne ustawienie lokalizacj dla Faker
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    import pytest
    from requests_mock import MockerCore
    from factory.faker import Faker
    from faker import config

    Faker._DEFAULT_LOCALE = 'pl_PL'
    config.DEFAULT_LOCALE = 'pl_PL'


Funkcja testująca metody widoków
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    def setup_view(view, request, *args, **kwargs):
        """
        Mimic as_view() returned callable, but returns view instance.
        args and kwargs are the same you would pass to ``reverse()``

        Example:
        name = 'django'
        request = RequestFactory().get('/fake-path')
        view = HelloView(template_name='hello.html')
        view = setup_view(view, request, name=name)

        Example test ugly dispatch():
        response = view.dispatch(view.request, *view.args, **view.kwargs)
        """
        view.request = request
        view.args = args
        view.kwargs = kwargs
        return view


Funkcja testująca metody widoków API
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    def api_setup_view(view, request, action=None, *args, **kwargs):
        """
        request = HttpRequest()
        view = views.ProfileInfoView()
        view = api_setup_view(view, request, 'list')
        assert view.get_serializer_class() == view.serializer_class
        """
        view.request = request
        view.action = action
        view.args = args
        view.kwargs = kwargs
        return view


Klasa APIRequestFactory jako fixture
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    @pytest.fixture()
    def api_rf():
        """
        APIRequestFactory instance
        """
        skip_if_no_django()
        from rest_framework.test import APIRequestFactory
        return APIRequestFactory()


Biblioteka requests_mock jako fixture
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    import pytest
    from requests_mock import MockerCore

    # --------------------------------------------------------------------
    # dodatek pozwalający w łatwy sposób robić mock dla biblioteki request
    # --------------------------------------------------------------------

    @pytest.yield_fixture(scope="session")
    def requests_mock():
        """
        def test_get_tags(self, requests_mock):
            requests_mock.get(settings.MY_SERVICE + 'tag/', json=response)
            cron = ImportTriviaCromJob()
            assert list(cron.get_tags(name)) == result
        """
        mock = MockerCore()
        mock.start()
        yield mock
        mock.stop()


Fixture dla DjangoLiveServer w kontenerze Docker
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    import pytest
    from pytest_django.lazy_django import skip_if_no_django
    from pytest_django.live_server_helper import LiveServer

    # ------------------------------------------------------------------------
    # dodatek pozwalający na uruchomienie DjangoLiveServer w kontenerze Docker
    # ------------------------------------------------------------------------

    @pytest.fixture(scope='session')
    def live_server(request):
        server = DockerLiveServer()
        request.addfinalizer(server.stop)
        return server


    class DockerLiveServer(LiveServer):

        def __init__(self):
            import socket
            self.addr = socket.gethostbyname(socket.gethostname())

            import django
            from django.db import connections
            from django.test.testcases import LiveServerThread
            from django.test.utils import modify_settings

            connections_override = {}
            for conn in connections.all():
                # If using in-memory sqlite databases, pass the connections to
                # the server thread.
                if conn.vendor == 'sqlite' and conn.is_in_memory_db(conn.settings_dict['NAME']):
                    # Explicitly enable thread-shareability for this connection
                    conn.allow_thread_sharing = True
                    connections_override[conn.alias] = conn

            liveserver_kwargs = {'connections_override': connections_override}
            from django.conf import settings
            if 'django.contrib.staticfiles' in settings.INSTALLED_APPS:
                from django.contrib.staticfiles.handlers import StaticFilesHandler
                liveserver_kwargs['static_handler'] = StaticFilesHandler
            else:
                from django.test.testcases import _StaticFilesHandler
                liveserver_kwargs['static_handler'] = _StaticFilesHandler

            if django.VERSION < (1, 11):
                host, possible_ports = self.addr, [8081]
                self.thread = LiveServerThread(host, possible_ports, **liveserver_kwargs)
            else:
                host = self.addr
                self.thread = LiveServerThread(host, **liveserver_kwargs)

            self._live_server_modified_settings = modify_settings(
                ALLOWED_HOSTS={'append': host}
            )
            self._live_server_modified_settings.enable()

            self.thread.daemon = True
            self.thread.start()
            self.thread.is_ready.wait()

            if self.thread.error:
                raise self.thread.error

        @property
        def url(self):
            if self.thread.host == self.addr:
                return 'http://%s:%s' % ('localhost', self.thread.port)
            return 'http://%s:%s' % (self.thread.host, self.thread.port)
