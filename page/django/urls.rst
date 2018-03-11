=================
Testowanie linków
=================

W taki sposób możemy sprawdzić czy wywołanie konkretnego adresu url spowoduje
uruchomienie wskazanej przez nas klasy lub funkcji widoku.

Django Class Base View
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    from users.views import ReferralsView

    found = resolve(reverse('referrals'))
    assert found.func.view_class == ReferralsView


Django Function View
^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    from chat.views import get_chats

    found = resolve(reverse('referrals'))
    assert found2.func == get_chats


Django DRF Function View
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    from chat.views import get_chats

    found = resolve(reverse('referrals'))
    assert found2.func.__name__ == get_chats.__name__

