
Rendering websites
==================

Navigating to the main address ``http://localhost:8000/`` still displays just congratulations page. We can't have only
that!

It's good to start working on a new website after preparing a plan for URLs (addresses). We know we
will want to see a list of all the questionnaires on the site, let users vote and display the
aggregated results of the questionnaire.

Let's open the file ``urls.py`` again and add four new entries. Eventually the file should look like this::

    from django.contrib import admin
    from django.urls import path
    import polls.views

    urlpatterns = [
        path(r'polls/', polls.views.index),
        path(r'polls/<int:poll_id>/', polls.views.detail),
        path(r'polls/<int:poll_id>/results/', polls.views.results),
        path(r'polls/<int:poll_id>/vote/', polls.views.vote),
        path(r'admin/', admin.site.urls),
    ]

Let's look at this example again. Each ``urlpatterns`` list element determines one URL pattern (address).

When a user tries to enter a specific address on our website, such as: ``http://localhost:8000/polls/1/``,
Django selects the third part of the URL after the slash (in this case, ``polls/1/``) and tries to match the ``urlpatterns``. Let's look at an example of such expressions::

    r'polls/<int:poll_id>/vote/'

This is a normal string (except for the initial ``r``, which is used here only for
convenience). Now do you see that our sample phrase matches ``polls/1/?`` Why?

Once Django finds a match, it will look at the second argument of the ``path()``. It defines the view, which is
called to create the site for the user. For ``polls/1/`` it will be ``polls.views.detail``. All named
parameters will be transferred to the view as arguments of the same name.

First view
----------

Ok, let's see how it works in practice. Unfortunately, entering the address
``http://localhost:8000/polls/1/`` does not end well::

    AttributeError: module 'polls.views' has no attribute 'index'

This is because we have not defined the view yet (Django tells us that it was looking for ``polls.views.
detail``, unfortunately without any success).

Let???s open the file ``polls/views.py`` and add a few new features::

    from django.http import HttpResponse

    def index(request):
        return HttpResponse("Hello, world. You're at the poll index.")

    def detail(request, poll_id):
        return HttpResponse("You're looking at poll %s." % poll_id)

    def results(request, poll_id):
        return HttpResponse("You're looking at the results of poll %s." % poll_id)

    def vote(request, poll_id):
        return HttpResponse("You're voting on poll %s." % poll_id)

These are the simplest possible views. They do not return regular strings, like the function that
builds the Christmas tree in Python, because they have to speak in HTTP protocol, which is a bit more
complicated (Here it would be interesting to see in a browser, what's going on when you enter the
address ``http://localhost:8000/polls/1/``).


View that really does something
-------------------------------

For now our views don???t do much. Let's give them some work!

Everything Django needs from the view is a
`HttpResponse <https://docs.djangoproject.com/en/3.0/ref/request-response/#django.http.HttpResponse>`_
object or a thrown exception. All the rest is under our control. For example, we can use the functions that we learned in the interactive mode to display all the polls for the user.


At the beginning of the ``polls/views.py`` file append::

    from django.http import HttpResponse
    from polls.models import Poll

Expand function ``index`` to look as below:

.. code-block:: python

    def index(request):
        latest_poll_list = Poll.objects.all().order_by('-pub_date')[:5]
        output = ', '.join([p.question for p in latest_poll_list])
        return HttpResponse(output)

.. note::

    We do not want to display the entire content of the file as it would be too long. Only the most important changes should be reported.

It works! There is only one problem with this example: we define in the view not only what has to be
returned, but also in what format it should be returned to the site user. One of the most important
skills of a programmer is the ability to distinguish and divide those two independent things.

Django programmers thought about it and decided to create a system of templates:

At the beginning of the ``polls/views.py`` file append::

    from django.template import loader

This will let us use the template system.

In the same file, expand the function ``index`` into this::

    def index(request):
        latest_poll_list = Poll.objects.all().order_by('-pub_date')[:5]
        t = loader.get_template('polls/index.html')
        c = {
            'latest_poll_list': latest_poll_list,
        }
        return HttpResponse(t.render(c))

The functions ``get_template`` (it finds a template) and ``render`` (it changes the template into a text
to be delivered to the user) are responsible for our template handling.

The code is a bit longer, but we will see soon that everything is much clearer. First,
let???s load the page ``http://localhost:8000/polls/`` to see the result of our work::

    TemplateDoesNotExist at /polls/
    polls/index.html

Oops! Well, we still haven???t added the template. To do this, let's create a file ``polls/templates/polls/index.html`` and add to it:

.. code-block:: django

    {% if latest_poll_list %}
    <ul>
        {% for poll in latest_poll_list %}
            <li><a href="/polls/{{ poll.id }}/">{{ poll.question }}</a></li>
        {% endfor %}
    </ul>
    {% else %}
        <p>No polls are available.</p>
    {% endif %}

.. note::
    Application ``templates`` are located in the directory ``templates of applications`` and the function ``get_template`` searches templates in these directories, that is why we didn???t have to give the entire path ``polls/templates/polls/index.html``, ``polls/index.html.`` was enough.

When you reload the page in a browser you should see a list of all the polls created earlier.

.. note::

    If you refresh the page and you still see an error, you must restart the server. In the console where the server is running press ``Ctrl + C`` and execute ``python manage.py runserver`` again. It should work now.

.. note::

   HTML and CSS are the formats that define the appearance of web pages. We will use Django templates to generate the HTML code. A good description of HTML is in the book
   `Interactive Data Visualization for the Web <http://shop.oreilly.com/product/0636920026938.do>`_.
   The incredible characteristics of the Web is that HTML and CSS codes of every site are public. We recommend to look at the code of your favorite sites.

In almost every view, you will eventually need to use a template. Therefore, in Django there is a
function ``render`` which allows you to do this in a shorter way:

Please correct the beginning of the file ``polls/views.py`` to look like this::

  from django.shortcuts import render
  from polls.models import Poll

Please correct the ``index`` function to look like this::

    def index(request):
        latest_poll_list = Poll.objects.all().order_by('-pub_date')[:5]
        return render(
            request,
            'polls/index.html',
            {'latest_poll_list': latest_poll_list})


Returning 404 code
------------------

Now let???s focus on the view of the poll details ??? a site which displays questions from a defined
questionnaire.

At the beginning of the file ``polls/views.py``, append::

    from django.http import Http404

``Http404`` is an exception shared by Django. We can use this exception in case our application can???t
find the poll desired by the user (by writing ``raise Http404``). As a result, the browser will show the error
404 page.


.. note::

    You can change the page displayed by Django in case of the error 404 (the page does not exist) and 500 (unexpected server error). To do this, you need to create templates ``polls/templates/404.html`` and ``polls/templates/500.html``. Before checking if it works, change the ``settings.py`` file params: ``DEBUG`` to ``False`` and ``ALLOWED_HOSTS`` to ``['*']``. Otherwise, Django will continue to display its auxiliary yellow pages or even won't start at all.

Change the function ``detail`` as below::

    def detail(request, poll_id):
        try:
            p = Poll.objects.get(id=poll_id)
        except Poll.DoesNotExist:
            raise Http404
        return render(request, 'polls/detail.html', {'poll': p})

Then create the ``polls/templates/polls/detail.html`` file with content as below:

.. code-block:: django

    <h1>{{ poll.question }}</h1>
    <ul>
    {% for choice in poll.choice_set.all %}
        <li>{{ choice.choice_text }}</li>
    {% endfor %}
    </ul>


Form management
---------------

Let's change the template ``polls/templates/polls/details.html`` by adding a simple HTML form.

Change the file ``polls/templates/polls/details.html`` as below:

.. code-block:: django

    <h1>{{ poll.question }}</h1>

    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

    <form action="/polls/{{ poll.id }}/vote/" method="post">
    {% csrf_token %}
    {% for choice in poll.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
    {% endfor %}
    <input type="submit" value="Vote" />
    </form>

.. note::

   ``{% csrf_token %}`` is a very magical way to protect websites from new forms of attack on websites users. You can find more information in the documentation
   `Cross Site Request Forgery protection <https://docs.djangoproject.com/en/3.0/ref/csrf/>`_.

Attentive readers will note that the form is sent to the ``/polls/{{ poll.id }}/vote/`` address, which does not yet
support data from forms. Let's add support for forms.

At the beginning of the file ``polls/views.py`` append::

    from django.http import HttpResponseRedirect
    from django.shortcuts import get_object_or_404
    from django.urls import reverse
    from polls.models import Choice

Correct the function ``vote`` as below::

    def vote(request, poll_id):
        p = get_object_or_404(Poll, id=poll_id)
        try:
            selected_choice = p.choice_set.get(id=request.POST['choice'])
        except (KeyError, Choice.DoesNotExist):
            # If the user chooses the wrong option, show error
            return render(request, 'polls/detail.html', {
                'poll': p,
                'error_message': "You have to choose the correct option.",
            })

        # Save the new number of votes
        selected_choice.votes += 1
        selected_choice.save()
        # Redirect a user to the detail view of the poll on which he or she just voted.
        return HttpResponseRedirect(reverse(results, args=(p.id,)))

In the view there are a lot of new ideas we have not yet discussed.

The ``request`` object contains the data sent by the user, and ``request.POST`` contains the form data sent by the user. This lets us know which option was selected.

Here comes the important question: it may turn out that the view received a nonexistent answer. We
always have to check the data received from the user and respond to a situation when the data does not make
sense. This is what happens in the :keyword:`except` clause. In such a case, we redirect the user to the
questionnaire and display the error.

If the user selects the correct option, we can increase the number of votes and save the changes. Then,
we redirect to the view of the questionnaire details we wrote previously by using ``HttpResponseRedirect``.

Another important issue: after voting, we could just display the page, like at the end of the view of
details (by using render). Unfortunately this could lead to  resending of the questionnaire if the
user starts playing with the back and forward buttons in the browser, or just refreshes the page (by
pressing F5). To prevent this, we should always redirect with HttpResponseRedirect after the correct form is submitted (in this case, voting for a poll).

At the end we still have to develop a view of the poll results to display after voting.

Correct ``results function``, as below::

    def results(request, poll_id):
        p = get_object_or_404(Poll, id=poll_id)
        return render(request, 'polls/results.html', {'poll': p})

Create the file ``polls/templates/polls/results.html`` with the following content:

.. code-block:: django

    <h1>{{ poll.question }}</h1>

    <ul>
    {% for choice in poll.choice_set.all %}
        <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
    {% endfor %}
    </ul>

    <a href="/polls/{{ poll.id }}/">Vote again?</a>

That's it! Enter the address ``http://localhost:8000/admin/`` and create several new polls and questions.
Then play with voting on them and invite others to do the same.


.. admonition:: ``polls/views.py``
   :class: alert alert-hidden

   .. code-block:: python

        from django.http import Http404
        from django.http import HttpResponseRedirect

        from django.shortcuts import get_object_or_404
        from django.shortcuts import render
        from django.urls import reverse

        from polls.models import Choice
        from polls.models import Poll

        def index(request):
            latest_poll_list = Poll.objects.all().order_by('-pub_date')[:5]
            return render(
                request,
                'polls/index.html',
                {'latest_poll_list': latest_poll_list})

        def detail(request, poll_id):
            try:
                p = Poll.objects.get(id=poll_id)
            except Poll.DoesNotExist:
                raise Http404
            return render(request, 'polls/detail.html', {'poll': p})

        def results(request, poll_id):
            p = get_object_or_404(Poll, id=poll_id)
            return render(request, 'polls/results.html', {'poll': p})

        def vote(request, poll_id):
            p = get_object_or_404(Poll, id=poll_id)
            try:
                selected_choice = p.choice_set.get(id=request.POST['choice'])
            except (KeyError, Choice.DoesNotExist):
                # If the user chooses the wrong option, show error
                return render(request, 'polls/detail.html', {
                    'poll': p,
                    'error_message': "You have to choose the correct option.",
                })

            # Save the new number of votes
            selected_choice.votes += 1
            selected_choice.save()
            # Redirect a user to the detail view of the poll on which he or she just voted.
            return HttpResponseRedirect(reverse(results, args=(p.id,)))

.. admonition:: ``urls.py``
   :class: alert alert-hidden

   .. code-block:: python

        from django.contrib import admin
        from django.urls import path
        import polls.views

        urlpatterns = [
            path(r'polls/', polls.views.index),
            path(r'polls/<int:poll_id>/', polls.views.detail),
            path(r'polls/<int:poll_id>/results/', polls.views.results),
            path(r'polls/<int:poll_id>/vote/', polls.views.vote),
            path(r'admin/', admin.site.urls),
        ]

.. admonition:: ``polls/models.py``
   :class: alert alert-hidden

   .. code-block:: python

        import datetime

        from django.db import models

        class Poll(models.Model):
            question = models.CharField(max_length=200)
            pub_date = models.DateTimeField('date published')

            def __str__(self):
                return self.question

            def was_published_recently(self):
                return self.pub_date >= datetime.datetime.now() - datetime.timedelta(days=1)


        class Choice(models.Model):
            poll = models.ForeignKey(Poll, on_delete=models.deletion.CASCADE)
            choice_text = models.CharField(max_length=200)
            votes = models.IntegerField(default=0)

            def __str__(self):
                return self.choice_text
