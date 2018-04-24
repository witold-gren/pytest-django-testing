Efektywne pisanie testów w języku python
========================================

Niniejsza dokumentacja został przygotowana aby przedstawić najbardziej efektywne techniki
pisania testów w języku Python oraz w frameworku Django. Istnieje bardzo dużo artykułów
i samouczków w internecie, ale żaden z nich nie zbiera całej wiedzy w jednym dokumencie.
Są one skupione na pojedynczych elementach testowania, lub na specyficznych problemach.

Większość samouczków wykorzystuje domyślną bibliotekę ``UnitTest``. Brakuje jednak
dokumentu opisującego wspaniałe narzędzie jakim jest ``PyTest`` oraz opisu pluginów które
można wykorzystać podczas pisania testów, a które w znacznym stopniu ułatwiają pracę
programisty.

Niniejszy dokument został podzielony na trzy częsci. Pierwsza opisuje zastosowanie
modułu ``PyTest`` wraz z kilkoma bardzo przydatnymi dodatkami. Druga część opisuje
w jaki sposób efektywnie pisać testy dla frameworka Django. Trzecia część skupia się na
dodatkowych narzędziach, które można wykorzystać w pracy z Django.


Aby zainstalować wszystkie opisywane paczki, należy uruchomić poniższą komendę:

.. code-block:: bash

   $ pip install pytest pytest-xdist pytest-pudb pytest-cov pytest-django pytest-factoryboy
   pytest-faker pytest-selenium pytest-mock pytest-benchmark jupyter django-extensions
   django-debug-toolbar django-webtest pylama


.. toctree::
   :maxdepth: 1
   :caption: Poznaj PyTest:

   page/pytest/pytest
   page/pytest/pytest_sugar
   page/pytest/pytest_xdist
   page/pytest/pytest_pudb
   page/pytest/pytest_cov
   page/pytest/pytest_factoryboy
   page/pytest/pytest_faker
   page/pytest/pytest_mock
   page/pytest/pytest_benchmark
   page/pytest/pytest_selenium
   page/extra/pylama
   page/extra/locust

.. toctree::
   :maxdepth: 2
   :caption: Testowanie aplikacji Django:

   page/django/pytest_django
   page/django/django_webtest
   page/django/index


.. toctree::
   :maxdepth: 1
   :caption: Przydatne moduły do pracy z Django:

   page/modules/jupyter
   page/modules/django_extensions
   page/modules/django_debug_toolbar


Indices and tables
==================

* :ref:`genindex`
* :ref:`search`
* :ref:`bibliography`


.. _pytest: https://docs.pytest.org
.. _pytest-django: https://pytest-django.readthedocs.io/en/latest/
.. _pytest-factoryboy: http://pytest-factoryboy.readthedocs.io/en/latest/
.. _pytest-mock: https://github.com/pytest-dev/pytest-mock/
.. _pytest-faker: http://pytest-faker.readthedocs.io/en/latest/
.. _pytest-xdist: https://github.com/pytest-dev/pytest-xdist
.. _pytest-pudb: https://github.com/wronglink/pytest-pudb
.. _pytest-benchmark: https://pytest-benchmark.readthedocs.io/en/latest/
.. _pytest-cov: https://pytest-cov.readthedocs.io/en/latest/
.. _pytest-selenium: http://pytest-selenium.readthedocs.io/en/latest/index.html
.. _jupyter: http://jupyter.readthedocs.io/en/latest/index.html
.. _django-extensions: https://django-extensions.readthedocs.io/en/latest/
.. _django-debug-toolbar: https://django-debug-toolbar.readthedocs.io/en/stable/
.. _pylama: https://github.com/klen/pylama
