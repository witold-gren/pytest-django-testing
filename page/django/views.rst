==================
Testowanie widoków
==================

Zadania widoku:

* gromadzi dane do renderowania przy użyciu metod menedżera
* inicjowanie formularzy
* renderowanie szablonów


W widokach nie powinieneś:

* sprawdzać poprawność danych - za to jest odpowiedzialność formularz
* zapisywać danych - to jest odpowiedzialność formularza
* budować złożonych zapytań - to jest odpowiedzialność menedżera


Jak testować:

* Testując widoki wykorzystuj ``RequestFactory``
* jeśli możesz wykonuj bezpośrednie wywołanie widoku
* Konfiguruj zależności w sposób jawny (np. request.user, request.session)


Podstawy
--------

.. code-block:: python

    from django.contrib.auth.models import User
    from django.http import HttpResponseRedirect
    from django.views.generic import TemplateView

    from .forms import AddTaskForm


    class AddTaskView(TemplateView):

        template_name = 'tasks/index.html'
        form_class = AddTaskForm
        users_manager = User.objects

        def get(self, request, *args, **kwargs):
            author = request.user
            owner = self._get_owner()
            form = self.form_class(author=author, owner=owner)
            context = {
                'form': form,
                'owner': owner,
                'author': author,
            }
            return self.render_to_response(context)

        def post(self, request, *args, **kwargs):
            author = request.user
            owner = self._get_owner()
            form = self.form_class(request.POST, author=author, owner=owner)
            if form.is_valid():
                obj = form.save()
                return HttpResponseRedirect(obj.get_absolute_url())
            context = {
                'form': form,
                'owner': owner,
                'author': author,
            }
            return self.render_to_response(context)

        def _get_owner(self):
            try:
                profile_login = self.kwargs['profile']
            except KeyError:
                owner = self.request.user
            else:
                owner = self.users_manager.get(username=profile_login)
            return owner


.. code-block:: python

    class TestView:

        def setUp(self):
            self._form_class = Mock(AddTaskForm)
            self._users_manager = Mock(User.objects)
            self._view = AddTaskView.as_view(
                form_class=self._form_class,
                users_manager=self._users_manager)

        def test_should_display_task_creation_form(self):
            url = reverse('my_tasks')
            request = self._factory.get(url)
            request.user = Mock(User)

            response = self._view(request)

            self.assertTrue('form' in response.context_data)

        def test_should_save_form_and_redirect_on_success(self):
            url = reverse('my_tasks')
            form = self._form_class.return_value
            form.is_valid.return_value = True
            redirect_url = '/some/url'
            obj = form.save.return_value
            obj.get_absolute_url.return_value = redirect_url
            data = {
                'title': sentinel.title,
            }
            request = self._factory.post(url, data)
            request.user = Mock(User)

            response = self._view(request)

            self.assertTrue(form.save.called)
            self.assertTrue(obj.get_absolute_url.called)
            self.assertEqual(response.status_code, 302)
            self.assertEqual(response['Location'], redirect_url)



request = RequestFactory().get('/fake-path')
view = HelloView.as_view(template_name='hello.html')
response = view(request, name='bob')
