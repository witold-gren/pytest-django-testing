==================================
Testowanie w Django REST framework
==================================

Aby przetestować działanie naszego API najprościej jest wykorzystać testu
funkcjonalne. Jednak warto również przetestować działanie naszych serializatorów
(jeśli posiadają nie standardową logikę) lub poszczególnych metody wykorzystywanych
w klasach `APIView` lub `ViewSet`.


Sprawdzamy czy jest zwracana poprawna odpowiedź poprzez klienta, są to testy
funkcjonalne wykorzystujące klienta WebTest.

.. code-block:: python

    class ProductAPITests:

        def test_can_get_product_details(self, django_app, product_factory):
            product = product_factory()
            response = django_app.get(f'/products/{product.id}/')
            assert response.status_code == 200
            assert response.data == ProductSerializer(instance=product).data

        def test_can_delete_product(self, django_app, product_factory):
            product = product_factory()
            response = django_app.delete(f'/products/{product.id}/delete/')
            assert response.status_code == 204
            assert Product.objects.count() == 0

        def test_can_update_product(self, django_app, product_factory):
            product = product_factory()
            response = django_app.patch_json(f'/products/{product.id}/update/', params={'name': 'Samsung Watch'})
            product.refresh_from_db()
            self.assertEqual(response.status_code, 200)
            self.assertEqual(product.name, 'Samsung Watch')


Testowanie widoków poprzez APIRequestFactory
--------------------------------------------

Możemy również napisać testy, tylko i wyłącznie dla konkretnego widoku z pominięciem
całego stosu zapytania i odpowiedzi (pomijając np. middleware, czy router dla adresu url).

.. code-block:: python

    from rest_framework import status, viewsets

    class UserContractViewSet(viewsets.ReadOnlyModelViewSet):
        serializer_class = UserContractSerializer

        def get_queryset(self):
            if self.request.user.is_staff:
                return UserContract.objects.all()
            return UserContract.objects.filter(user=self.request.user)


.. code-block:: python

    @pytest.mark.django_db
    def test_user_can_see_own_contracts(api_rf, user_factory, user_contract_factory):
        view = UserContractViewSet.as_view({'get': 'list'})
        user = user_factory()
        user_contracts = user_contract_factory.build_batch(3, user=user)

        request = api_rf.get('/user_contracts/')
        force_authenticate(request, user=user)
        response = view(request)

        assert response.status_code == status.HTTP_200_OK
        assert response.data.get('count') == 3
        assert response.data.get("results") == UserContractSerializer(
            user_contracts, many=True).data


.. code-block:: python

    @pytest.mark.django_db
    def test_user_can_see_own_single_contract(api_rf, user_factory, user_contract_factory):
        view = UserContractViewSet.as_view({'get': 'retrieve'})
        user = user_factory()
        user_contract = user_contract_factory(user=user)

        request = api_rf.get(f'user_contracts/{user_contract.pk}')
        force_authenticate(request, user=user)
        response = view(request, pk=user_contract.pk)

        assert response.status_code == status.HTTP_200_OK
        assert response.data == UserContractSerializer(user_contract).data


Testowanie dekoratora actions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bardzo często korzystając z `ViewSet` można stworzyć dodatkowe metody wywoływane
poprzez url dla obiektu lub listy obiektów. Warto również je przetestować
wywołując odpowiednio api widoku.


.. code-block:: python

    class UserViewSet(ModelViewSet):
        ...

        @action(methods=['post'], detail=True, permission_classes=[IsAdminOrIsSelf])
        def set_password(self, request, pk=None):
        ...

.. code-block:: python

    @pytest.mark.django_db
    def test_user_set_password(api_rf, user_factory):
        view = UserViewSet.as_view({'get': 'set_password'})
        user = user_factory()

        request = api_rf.post_json(
            f'/user_contracts/{user_contract.pk}/set_password',
            params={'password': '123123'})
        force_authenticate(request, user=user)
        response = view(request, pk=user.pk)

        assert response.status_code == status.HTTP_200_OK


Testowanie routera oraz url
---------------------------

Widoki funkcyjne
^^^^^^^^^^^^^^^^

.. code-block:: python

    from chat.views import get_chats

    found = resolve(reverse('referrals'))
    assert found2.func.__name__ == get_chats.__name__


Widoki klasowe
^^^^^^^^^^^^^^

.. code-block:: python

    def test_check_if_recent_url_exist_and_have_good_class(self):
        found = resolve('/notifications/recent/')
        assert found.func.cls == views.UserNotification


Przykłady ViewSet dla wybranych akcji
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    router = DefaultRouter()
    router.register(r'my-list', MyViewSet, base_name="my_list")

    urlpatterns = [
        url(r'^api/', include(router.urls, namespace='api'))
    ]

.. code-block:: python

    def test_color_field_content(self):
        # for list URL. e.g. /api/my-list/
        path = 'api:my_list-list'
        assert reverse(path) == '/api/my-list/'

        found = resolve(reverse(path))
        assert found2.func.__name__ == get_chats.__name__

    def test_color_field_content(self):
        # for detail URL. e.g. /api/my-list/<pk>/
        path = 'api:my_list-detail'
        assert reverse(path, args=[1]) == '/api/my-list/1/'

        found = resolve(reverse(path))
        assert found.func.cls == views.UserNotification


Testowanie serializatora
------------------------

Testując widok sprawdzamy czy zwrócowna wartość wykorzystuje konkretny serializator.
Nie sprawdzamy jednak samego działania serializatora, nie wiemy czy dodaliśmy do niego
nowe pola, czy może nie zmieniliśmy akcji utworzenia nowego obiektu. Jeśli nasz
serializator posiada co najmniej jedną rzecz, która powoduje, że mamy jakieś
ograniczenia na polu lub podmieniamy domyślną metodę, wtedy musimy przetestować
serializator.

Poniższy przykład pokazuje bardzo prosty serializator, jednak jak zobaczysz w testach,
jest kilka rzeczy które warto sprawdzić.

.. code-block:: python

    from django.db import models

    class Tool(models.Model):
        COLOR_OPTIONS = (
            ('yellow', 'Yellow'),
            ('red', 'Red'),
            ('black', 'Black')
        )

        color = models.CharField(
            max_length=255,
            null=True,
            blank=True,
            choices=COLOR_OPTIONS)
        size = models.DecimalField(
            max_digits=4,
            decimal_places=2,
            null=True,
            blank=True)


Do podanego powyżej modelu tworzymy prosty serializator.

.. code-block:: python

    from rest_framework import serializers
    from tools.models import Tool

    class ToolSerializer(serializers.ModelSerializer):
        COLOR_OPTIONS = ('yellow', 'black')

        color = serializers.ChoiceField(
            choices=COLOR_OPTIONS)
        size = serializers.FloatField(
            min_value=30.0,
            max_value=60.0)

        class Meta:
            model = Tool
            fields = ['color', 'size']


Najpierw przygotowujemy naszą klasę, która będzie zawierać podstawowe dane.
W każdej chwili tworzą test, będziemy mogli je podmienić.

.. code-block:: python

    @pytest.mark.django_db
    class TestToolSerializer:

        @pytest.fixture(autouse=True)
        def setup_method(self, db, tool_factory):
            self.tool_attributes = {
                'color': 'yellow',
                'size': Decimal('52.12')}

            self.serializer_data = {
                'color': 'black',
                'size': 51.23}

            self.tool = tool_factory(**self.tool_attributes)
            self.serializer = ToolSerializer(instance=self.tool)


Używam zbioru pól aby upewnić się, że dane wyjściowe z serializera mają
dokładnie te pola, którychy oczekujemy. Używanie zbioru do tej weryfikacji jest
bardzo ważne, ponieważ gwarantuje, że dodanie lub usunięcie dowolnego pola
do serializera zostanie zauważone podczas wykonywania testów.

.. code-block:: python

    def test_contains_expected_fields(self):
        data = self.serializer.data
        assert set(data.keys()) == set(['color', 'size'])

Teraz przechodzimy do sprawdzania, czy serialalizator generuje oczekiwane dane
do każdego pola. Pole kolor jest dość standardowe:

.. code-block:: python

    def test_color_field_content(self):
        data = self.serializer.data
        assert data['color'] == self.tool_attributes['color']

    def test_size_field_content(self):
        data = self.serializer.data
        assert data['size'] == float(self.tool_attributes['size'])


Atrybut `size` ma zarówno dolny, jak i górny limit. Bardzo ważne jest
testowanie przypadków brzegowych które określiliśmy.

.. code-block:: python

    def test_size_lower_bound(self):
        self.serializer_data['size'] = 29.9

        serializer = ToolSerializer(data=self.serializer_data)

        assert serializer.is_valid() is False
        assert set(serializer.errors) == set(['size'])

    def test_size_upper_bound(self):
        self.serializer_data['size'] = 60.1

        serializer = ToolSerializer(data=self.serializer_data)

        assert serializer.is_valid() is False
        assert set(serializer.errors) == set(['size'])

    def test_float_data_correctly_saves_as_decimal(self):
        self.serializer_data['size'] = 31.789

        serializer = ToolSerializer(data=self.serializer_data)
        serializer.is_valid()

        new_tool = serializer.save()
        new_tool.refresh_from_db()

        assert new_tool.size == Decimal('31.79')

    def test_color_must_be_in_choices(self):
        self.tool_attributes['color'] = 'red'

        serializer = ToolSerializer(instance=self.tool, data=self.tool_attributes)

        assert serializer.is_valid() is False
        assert set(serializer.errors.keys()) == set(['color'])


Do przygotowania
----------------

- Testowanie walidatorów oraz walidacji pól
- Testowanie własnych pól
