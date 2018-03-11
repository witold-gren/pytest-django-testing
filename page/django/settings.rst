Ustawienia
----------

Change password hashing
^^^^^^^^^^^^^^^^^^^^^^^

This is the most effective setting you can use to improve the speed of tests, it may sounds
ridicolous, but password hashing in Django is designed to be very strong and it makes use of
several “hashers”, but this also means that the hashing is very slow. The fastest hasher is
the MD5PasswordHasher, so you can use just that one in this way:

.. code-block:: python

    PASSWORD_HASHERS = (
        'django.contrib.auth.hashers.MD5PasswordHasher',
    )


Use a faster storage system
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Currently the fastest database you can use with Django is SQLite. I was initially doubtful about switching from Postgres (my database of choice) to SQLite, but I’m not testing Django itself, I’m testing my own API and since I don’t use raw SQL statements the underlying storage backend should not make differences!
So, to configure SQLite:

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


Remove unnecessary middlewares
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Middleware layer is one of the Django features that I like the most, BUT the more middleware classes you use, the more your response time will increase (since all the middleware must be executed sequentially before returning the final HTTPResponse, ALWAYS!). So be sure to include only the stuff you really and absolutely need!
One middleware in particular that is very slow and that I removed for the tests is:

We can assume that all of Django's middelware works correctly, so while testing, remove it to avoid all that overhead when making requests with the test client. Our final testing middleware settings looked like this.

.. code-block:: python

    django.middleware.locale.LocaleMiddleware

    MIDDLEWARE_CLASSES = [
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
    ]


Remove unnecessary apps
^^^^^^^^^^^^^^^^^^^^^^^

There are several third party apps that you may remove during testing like the debug toolbar, try to remove all the unused/unnecessary apps.
In my project, we had over 30 apps installed. By overriding settings for tests, I could remove unneeded apps like django_debug_toolbar, django_extension, etc.


Turn debug off
^^^^^^^^^^^^^^

You might be surprised, but setting DEBUG = False while running tests reduces the amount of debugging overhead that django takes and will improve your the speed of your test suite.

.. code-block:: python

    DEBUG = False


Turn off logging
^^^^^^^^^^^^^^^^

This is a significant modification only if we have a huge amount of logging and/or additional logic involved in logs (such object inspections, heavy string manipulation and so on), but anyway logging is futile during testing, so:
There's no need to add file I/O overhead to your testing suite, so disable it!

.. code-block:: python

    import logging
    logging.disable(logging.CRITICAL)


Use a faster Email backend (by “patching” Django)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default Django will use django.core.mail.backends.locmem.EmailBackend, which is an in-memory backend designed for testing, however I had several problems with that backend during my tests, they did block unexplainably for ~30 seconds due to headers checking. So I decided to write my own in-memory backend which mimics the Django one but does not check email headers in order to be blazing fast:

.. code-block:: python

    EMAIL_BACKEND = "django.core.mail.backends.dummy.EmailBackend"


Use an in-memory backend for Celery
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are using Celery, these are my optimal settings for testing:

.. code-block:: python

    CELERY_ALWAYS_EAGER = True
    CELERY_EAGER_PROPAGATES_EXCEPTIONS = True
    BROKER_BACKEND = 'memory'


Mock, mock, mock!
^^^^^^^^^^^^^^^^^

A HUGE bottleneck of our tests was in the billing logic. We had many tests that were actually hitting billing APIs (on test accounts of course, but still really bad). By mocking those calls you can significantly reduce the testing time. Take this test that mocks if a customer has a card on file. By mocking that can_charge call and setting the return value, we avoid an API call and can still test that our code works as expected.

.. code-block:: python

    import mock
    from django.test import Client

    @mock.patch('billing.utils.can_charge')
    def test_cant_charge_redirect(can_charge):
        can_charge.return_value =False
        response = Client().get('/checkout/')
        self.assertRedirects(response, '/checkout/add-card/')

    @mock.patch('billing.utils.can_charge')
    def test_can_charge_ok(can_charge):
        can_charge.return_value = True
        response = Client().get('/checkout/')
        self.assertEqual(response.status_code, 200)


You can also mock simple model unit tests. Instead of hitting the database by creating models, use Mock objects to simulate your model


.. code-block:: python

    import mock
    from models import User

    def test_full_name():
        user = mock.Mock(spec=User)
        user.first, user.last = 'Test', 'Test'
        self.assertEqual(user.full_name, 'Test Test')


TOHETHER:
^^^^^^^^^

.. code-block:: python

    #!/usr/bin/env python
    import os
    import sys

    if __name__ == "__main__":
        os.environ.setdefault("DJANGO_SETTINGS_MODULE", "marketplace.settings")

        from django.core.management import execute_from_command_line
        from django.conf import settings

        if 'test' in sys.argv:
            import logging
            logging.disable(logging.CRITICAL)
            settings.DEBUG = False
            settings.TEMPLATE_DEBUG = False
            settings.PASSWORD_HASHERS = [
                'django.contrib.auth.hashers.MD5PasswordHasher',
            ]
            settings.DATABASES = {
                'default': {
                    'ENGINE': 'django.db.backends.sqlite3',
                    'NAME': 'test_database',
                }
            }
            settings.MIDDLEWARE_CLASSES = [
                'django.contrib.sessions.middleware.SessionMiddleware',
                'django.middleware.csrf.CsrfViewMiddleware',
                'django.contrib.auth.middleware.AuthenticationMiddleware',
                'django.contrib.messages.middleware.MessageMiddleware',
            ]

        if 'test' in sys.argv and '--time' in sys.argv:
            sys.argv.remove('--time')
            from django import test
            import time

            def setUp(self):
                self.startTime = time.time()

            def tearDown(self):
                total = time.time() - self.startTime
                if total > 0.5:
                    print("\n\t\033[91m%.3fs\t%s\033[0m" % (
                        total, self._testMethodName)

            test.TestCase.setUp = setUp
            test.TestCase.tearDown = tearDown

        execute_from_command_line(sys.argv)


Testowanie specyficznych pól z postgresql q SQLite
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

https://www.aychedee.com/2014/03/13/json-field-type-for-django/




.. code-block:: python

    import pytest
    from pytest_django.lazy_django import skip_if_no_django
    from pytest_django.live_server_helper import LiveServer
    from requests_mock import MockerCore
    from factory.faker import Faker
    from faker import config


    Faker._DEFAULT_LOCALE = 'pl_PL'
    config.DEFAULT_LOCALE = 'pl_PL'


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


    @pytest.fixture()
    def api_rf():
        """APIRequestFactory instance"""
        skip_if_no_django()
        from rest_framework.test import APIRequestFactory
        return APIRequestFactory()


    # ----------------------------------------------------------------------------------------
    # mój dodatek aby można było robić mock dla request do innych serwisów
    # ----------------------------------------------------------------------------------------

    @pytest.yield_fixture(scope="session")
    def requests_mock():
         mock = MockerCore()
         mock.start()
         yield mock
         mock.stop()


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
