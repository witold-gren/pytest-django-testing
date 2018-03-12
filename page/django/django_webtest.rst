==============
Django WebTest
==============

WebTest jest frameworkiem do pisania testów funkcjonalnych/integracyjnych czy akceptacyjnych.
Pierwotnie został napisany dla środowiska ``Pyramid`` jednak można go wykorzystywać również
w innych frameworkach napisanych w języku python. WebTest posiada proste API, jest szybki,
komunikuje się z aplikacją django przez WSGI (nie przez HTTP). Wykorzystując WebTest
dostarcza również mechanizm bardzo dobrej pracy z odpowiedzią otrzymanego widoku.

**Dlaczego WebTest a nie Django Client?**

Poniższy kod przedstawia utworzenie testu poprzez wykorzystanie `Django Client`.

.. code-block:: python

    url = "/case/edit/{0}".format(case.pk)
    step = case.steps.get()
    response = self.client.post(url, {
        "product": case.product.id,
        "name": case.name,
        "description": case.description,
        "steps-TOTAL_FORMS": 2,
        "steps-INITIAL_FORMS": 1,
        "steps-MAX_NUM_FORMS": 3,
        "steps-0-step": step.step,
        "steps-0-expected": step.expected,
        "steps-1-step": "Click link.",
        "steps-1-expected": "Account active.",
        "status": case.status,
    })


Drugi szablon, przedstawia wykonanie tego samego testu poprzez `WebTest`.

.. code-block:: python

    url = "/case/edit/{0}".format(case.pk)
    form = self.app.get(url).forms["case-form"]
    form["steps-1-step"] = "Click link."
    form["steps-1-expected"] = "Account active."
    form["product"] = case.product.id
    form["name"] = case.name
    form["description"] = case.description
    form["steps-TOTAL_FORMS"] = 2
    form["steps-INITIAL_FORMS"] = 1
    form["steps-MAX_NUM_FORMS"] = 3
    form["steps-0-step"] = step.step
    form["steps-0-expected] = step.expected
    form["steps-1-step"] = "Click link."
    form["steps-1-expected"] = "Account active."
    form["status"] = case.status
    response = form.submit()

Problem z utworzeniem testu poprzez django client polega na tym że może on pokazać nie
poprawne wykonanie testu. Wystarczy, że ktoś w szablonie html w którym znajduje się
formularz, przypadkiem usunie tag z specialnymi danymi dla ``formset`` wtedy test
zostanie poprawnie zaliczony, pomimo iż realnie nie działa. W przypadku wykorzystania
WebTest otrzymamy błąd o brakujących danych. Niestety przekazywane wartości poprzez
klient django pomijają renderowanie szablonu, co jest problemem podczas testowania
aplikacji.

.. note::

    Więcej szczegółów można znaleść pod adresami:
    https://github.com/django-webtest/django-webtest oraz
    https://docs.pylonsproject.org/projects/webtest/en/latest/.


Instalacja
----------

.. code-block:: bash

    $ pip install django-webtest


Opis działania
--------------

Domyślne metody to ``django_app.get()`` oraz ``django_app.post()`` czy ``django_app.post_json()`` z opcjonalnym argumentem ``user``.
Wywołanie metody ``django_app.reset()`` powoduje wyczyszczenie wszystkich ciasteczek oraz
wylogowanie użytkownika.

Aby sprawdzić status odpowiedzi:

.. code-block:: python

    >>> assert response.status == '200 OK'
    >>> assert response.status_int == 200

Nagłówki odpowiedzi:

.. code-block:: python

    >>> assert response.content_type == 'text/html'
    >>> assert response.content_type == 'application/json'
    >>> assert response.content_length > 0

Treść odpowiedzi:

.. code-block:: python

    >>> resp.mustcontain('<html>')  # zwraca błąd jeśli nie znaleziono łańcucha znaków
    >>> assert 'form' in response
    >>> assert response.json == {'id': 1, 'value': 'value'}
    >>> assert response.text == ''
    >>> assert response.body == ''

Można również pobrać ``request`` z odpowiedzi jednak jest on klasą ``webob.request.BaseRequest`` [#f1]_:


.. code-block:: python

    >>> response.request.url
    >>> response.request.remote_addr


Korzystając z django-webtest otrzymujesz również zmienne ``response.templates`` oraz ``response.context``, z których
można skorzystać w taki sam sposób jak z klienta django. Atrybuty te zawierają listę szablonów wykorzystanych do renderowania odpowiedzi
oraz kontekst używany do renderowania tych szablonów.

Sesja jest dostępna pod ``django_app.session``. Domyślnie WebTest w każdym zapytaniu w formularzy automatycznie również
wysyła zmienną ``CSRF``. Aby go wyłączyć należy skorzystać z fikstury ``django_app_factory`` i przekazać parametr ``csrf_checks=False``.


Zmienne z Django Client
-----------------------

.. code-block:: python

    # response.templates
    >>> assert response.templates.name == 'index.html'

    # response.context
    >>> assert response.context['user'] == user_obj

    response.status_code    # --> response.status_int
    response.content        # --> response.body
    response.url            # --> response['location']
    response._charset       # --> response.charset
    response.client         # --> return django.test.Client


Parsowanie odpowiedzi
---------------------

``response.html`` - zwraca obiekt ``BeautifulSoup``, którego można wykorzystać w testach

.. code-block:: python

    def test_login(django_app):
        resp = django_app.get('/')
        assert resp.html.find("a", title="Login").href == "/login/"


``response.xml`` - zwraca obiekt ``ElementTree``

.. code-block:: python

    def test_response_from_user_info(django_app):
        resp = django_app.get('/user/info/')
        assert resp.xml[0].text == 'hey!'


``response.lxml`` - zwraca obiekt ``lxml``

.. code-block:: python

    def test_response_from_user_info(django_app):
        resp = django_app.get('/user/info/')
        assert resp.lxml.xpath('//body/div')[0].text == 'hey!'


``response.pyquery`` - zwraca obiekt ``PyQuery`` (inna implementacja obsługi dokumentów xml)

.. code-block:: python

    def test_response_from_user_info(django_app):
        resp = django_app.get('/user/info/')
        assert resp.pyquery('message').text() == 'hey!'


``response.json`` - zwraca obiekt ``simplejson``

.. code-block:: python

    def test_login(django_app):
        resp = django_app.get('/')
        assert sorted(res.json.values()) == [1, 2]


Django Rest Framework
---------------------

Aby skorzystać z WebTest wraz z ``django rest framework`` należy utworzyć własną
implementację autoryzacji użytkownika. Dzięku temu, podając w zapytaniu ``user=str(user.username)``
użytkownik zostanie poprawnie zalogowany bez zbędnych dodatkowo wykonywanych metod.

Najpierw musimy przygotować moduł autoryzacyjny. Należy utworzyć plik ``webtest.py``
najlepiej w katalogu konfiguracji projektu. Następnie dodajemy do niego poniższą zawartość:

.. code-block:: python

    # config/webtest.py
    from rest_framework.compat import authenticate
    from rest_framework.authentication import BaseAuthentication


    class WebTestAuthentication(BaseAuthentication):
        """
        Auth backend for tests that use webtest with Django Rest Framework.
        """
        header = 'WEBTEST_USER'

        def authenticate(self, request):
            assert ValueError('exist')
            value = request.META.get(self.header)
            if value:
                user = authenticate(django_webtest_user=value)
                if user and user.is_active:
                    return user, None

Po utworzeniu pliku należy dodać utworzoną metodę autoryzacji do ``Django Rest Framework``.
Aby tego dokonać w pliku settings dodajemy naszą klasę ``WebTestAuthentication`` do słownika
z kluczem ``DEFAULT_AUTHENTICATION_CLASSES``.

.. code-block:: python

    # settings.py
    if ENVIRONMENT == 'tests':
        REST_FRAMEWORK['DEFAULT_AUTHENTICATION_CLASSES'] += [
            'config.webtest.WebTestAuthentication',
        ]

To wystarczy aby utworzyć test wraz z podaniem zalogowanego użytkownika.


.. code-block:: python

    # test
    resp = app.post_json('/resource/', params=dict(id=1, value='value'), user=str(user.username))



Praca z błędami odpowiedzi
--------------------------

Domyślnie jeśli w naszej odpowiedzi uzyskamy status odpowiedzi będący w innym przedziale
niż 200 <= STATUS < 400 zostanie podniesiony błąd. Aby świadomie przetestować takie rodzaje
odpowiedzi musimy złapać wyjątek ``webtest.AppError`` oraz sprawdzić jego status odpowiedzi.

.. note::

    Więcej szczegółów można znaleść pod adresami:
    https://stackoverflow.com/questions/21829962/test-for-http-405-not-allowed
    http://webtest.pythonpaste.org/en/latest/testapp.html#what-is-tested-by-default


.. code-block:: python

    def test_mainpage_post(self):
        with pytest.raises(webtest.AppError) as exc:
            response = self.testapp.post('/')

        assert str(exc).startswith('Bad response: 405')


    def test_mainpage_post(self):
        response = self.testapp.post('/', expect_errors=True)
        assert response.status_int == 405


    def test_mainpage_post(self):
        response = self.testapp.post('/', status=405)


Przykład wykorzystania
----------------------

.. code-block:: python

    def test_login(django_app):
        resp = django_app.get('/')
        assert resp.html.find("a", title="Login").href == "/login/"


.. code-block:: python

    def test_login_with_app_factory(django_app_factory):
        app = django_app_factory(csrf_checks=False, extra_environ={})
        resp = app.get('/')
        assert resp.html.find("a", title="Login").href == "/login/"


.. code-block:: python

    def test_blog(django_app):
        # pretend to be logged in as user `kmike` and go to the index page
        index = django_app.get('/', user='kmike')

        # All the webtest API is available. For example, we click
        # on a <a href='/tech-blog/'>Blog</a> link, check that it
        # works (result page doesn't raise exceptions and returns 200 http
        # code) and test if result page have 'My Article' text in
        # its body.
        assert 'My Article' in index.click('Blog')

.. code-block:: python

    def test_login(django_app):
        form = django_app.get(reverse('auth_login')).form
        form['username'] = 'foo'
        form['password'] = 'bar'
        response = form.submit().follow()
        assert response.context['user'].username == 'foo'


.. [#f1] `https://docs.pylonsproject.org/projects/webob/en/latest/api/request.html`_

.. _`https://docs.pylonsproject.org/projects/webob/en/latest/api/request.html` : https://docs.pylonsproject.org/projects/webob/en/latest/api/request.html
