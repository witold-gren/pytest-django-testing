===================
Testowanie w Django
===================

Rozbudowana struktura aplikacji

.. code-block:: bash

    - app_name/
        ├──__init__.py
        ├── apps.py                     # rejestracja aplikacji
        ├── models.py                   # modele aplikacji
        ├── admin.py                    # admin panel
        ├── urls.py                     # linki do widoków oraz router
        ├── views.py                    # widoki aplikacji
        ├── forms.py                    # formularze
        ├── managers.py                 # managery i querysety dla modeli
        ├── serializers.py              # serializatory do modelów
        ├── services.py                 # domenowa logika aplikacji
        ├── context_processors.py       # kontekst procesory dla szablonów
        ├── middleware.py               # własne middleware
        ├── task.py                     # zadania do celery
        ├── signals/                    # sygnały
        │   ├── __init__.py
        │   ├── signals.py              # miejsce do tworzenia własnych sygnałów
        │   └── handlers.py             # miejsce do odbierania sygnałów
        ├── templates/
        │   └── app_name/               # dodanie "przestrzeni nazw" dla templatów
        ├── templatetags/               # tagi szablonów
        │   ├── __init__.py
        │   └── ...
        ├── migrations/                 # migracje
        │    ├── __init__.py
        │    ├── 0001_initial.py
        │    └── ...
        ├── management/                 # komendy django uruchamiane z bash
        │    ├── __init__.py
        │    └── commands/
        │         ├── __init__.py
        │         └── ...
        └── tests/                      # katalog z testami
             ├── __init__.py
             ├── conftest.py            # ustawienia dla pytest, fixtures
             ├── factories.py           # fabryki obiektów dla modeli
             ├── test_forms.py          # testy formularzy
             ├── test_models.py         # testy modeli
             ├── test_admin.py          # testy admin panelu
             ├── test_views.py          # testy widoków
             ├── test_serializers.py    # testy serializatorów
             ├── ...                    # generalnie tworzymy plik odpowiadający testowanemu modułowi
             └── functional/            # testy funkcjonalne (WebTest, Django Client, Selenium)
                  ├── __init__.py
                  ├── view_board_topics.py
                  └── view_home.py


W jaki sposób najlepiej pisać testy?

Pytanie nie jest banalne i na początku sprawiało mi to dużo trudności. Pierwszym problemem był brak wiedzy w jaki sposób
skonfigurować środowisko z Django pozwalające na uruchomienie testów. Kolejny problem, to bardzo często złożoność modeli,
w jaki sposób tworzyć do nich obiekty? Inne problemy z którymi się borykałem, to: sprawdzenie czy url działają poprawnie,
sprawdzenie czy utworzone tagi do szablonów są poprawne, sprawdzenie czy formularze działają poprawnie. Najtrudniejsze było
jednak testowanie widoków w sposób pozwalający na przetestowanie tylko części funkcjonalności a nie całości.

Zbierając jednak całą tę wiedzę, dalej miałem problemy z tym, w jaki sposób dobrze poukładać testy. Jak w tym wszystkim
wykorzystać moduł `mock`.

`Podwójna pętla`, to sposób w którym pierwsze piszemy test funkcjonalny, określający na bardzo wysokim poziomie nasze założenia.
Test ten z założenia powinien być błędny. Następnie tworzymy testy jednostkowe pozwalające określić dokładniej co chcemy
zrobić. Każdorazowe poprawne wykonanie testów jednostkowych, będzie przybliżało nas do poprawnego wykonania testu funkcjonalnego.

Takie podejście również pozwala nam osiągnąć dwie bardzo ważne rzeczy. Pierwsza z nich to pewność, że finalnie nasza aplikacja
będzie działać (patrząc z poziomu klienta). Wykonując testy funkcjonalne nie wykorzystujemy obiektów `mock` oraz
nie nadpisujemy ustawień bazy danych co daje nam pewność, że wszystko będzie OK. Druga ważna rzecz, to możliwość pisania
szybkich testów jednostkowych, które nie zawsze będą tworzyły zapytania do bazy danych.

.. toctree::
    :maxdepth: 1

    settings
    models
    managers
    admin
    forms
    urls
    views
    middleware
    template_tags
    django_rest_framework
    celery
    django_mock
