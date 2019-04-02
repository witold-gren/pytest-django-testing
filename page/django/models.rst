=================
Testowanie modeli
=================

Testowanie modeli polega przede wszystkim na sprawdzeniu czy możemy pracować
na utworzonym modelu. Dlatego sprawdzamy możliwość przeprowadzenia podstawowych
operacji CRUD na modelu.

Podstawy
--------

Jeśli posiadamy model który nie zawiera dużej ilości pól z relacjami, możemy do
jego przetestowania wykorzystać `factory-boy` wraz z metodą `clean_fields()`.

.. code-block:: python

    ...

    def test_create_new_openerrors(self):
        open_errors = open_errors_factory.build()
        open_errors.clean_fields(exclude=['user'])  # EXCLUDE: FK, O2O, M2M Fields
        assert open_errors

Wywołując metodę `clean_fields` sprawdzimy czy pola modelu zawierają wszystkie
zdefiniowane i wymagane parametry (na. max_length dla CharField). Sprawdzimy
również wszystkie validatory, które zostały dodane do poszczególnych pól.

Można sprawdzić zapisując obiekt do bazy danych:

.. code-block:: python

    @pytest.mark.django_db
    def test_create_new_openerrors(self, open_errors_factory):
        # Create the venue
        openerror = open_errors_factory()

        # Check all field and validators
        openerror.clean_fields()

        # Check we can find it
        openerrors = OpenError.objects.all()
        assert len(openerrors) == 1

        first_openerrors = openerrors[0]
        assert first_openerrors == openerror


Testujemy również wszystkie utworzone metody.

.. code-block:: python

    @pytest.mark.django_db
    class TestOpenErrorsModel:
        ...

        def test_check_attribute_in_openerrors(self, open_errors_factory):
            # Check attributes
            venue = open_errors_factory(name='Wembley Arena')
            assert only_venue.name == 'Wembley Arena'

        def test_check_str_representation_of_openerrors(self, open_errors_factory):
            # Check string representation
            venue = open_errors_factory(name='Wembley Arena')
            assert only_venue.__str__() == 'Wembley Arena'  # or str(only_venue)

Jeżeli wykorzystujemy nadpisane metody np. `save()` warto również przetestować
je oddzielnie.

.. code-block:: python

    class Survey(models.Model):
        title = models.CharField(max_length=60)
        opens = models.DateField()
        closes = models.DateField()

        def save(self, **kwargs):
            if not self.pk and not self.closes:
                self.closes = self.opens + datetime.timedelta(7)
            super(Survey, self).save(**kwargs)

.. code-block:: python

    @pytest.mark.django_db
    class SurveySaveTest:
        t = "New Year"
        sd = datetime.date(2018, 1, 1)

        def test_closes_autoset(self, survey_factory):
            s = survey_factory(title=self.t, opens=self.sd)
            assert s.closes == datetime.date(2018, 1, 4))

        def test_closes_honored(self, survey_factory):
            s = survey_factory(title=self.t, opens=self.sd, closes=self.sd)
            assert s.closes == self.sd

        def test_closes_reset(self, survey_factory):
            s = survey_factory(title=self.t, opens=self.sd)
            s.closes = None
            with pytest.raises(IntegrityError):
                s.save()

        def test_title_only(self):
            with pytest.raises(IntegrityError):
                Survey.objects.create(title=self.t)
