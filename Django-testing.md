Table of Contents:
1. <a href="#1">Test structure overview</a>
2. <a href="#2">setUp vs setUpClass vs setUpTestData</a>
   2.1 setUp
   2.2 setUpClass
   2.3 setUpTestData
   2.4 __init__ method
3. <a href="#3">How to run the tests</a>
   3.1 Running all the tests
   3.2 Running specific tests
   3.3 Showing more test information
4. <a href="#4">TestCase testing order</a>
5. <a href="#5">Testing example</a>
   5.1 Models tests example
   5.2 Views tests example
   5.3 DRF API tests example

<br>

References:
- https://docs.djangoproject.com/en/3.2/topics/testing/
- https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/testing
- https://www.django-rest-framework.org/api-guide/testing/

<br>

# <a name="1">1. Test structure overview</a>

Django uses the unittest module’s built-in test discovery, which will discover tests under the current working directory in any file named with the pattern **test\*.py**.
[We](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Testing#test_structure_overview) recommend that you create a module for your test code, and have separate files for models, views, forms, and any other types of code you need to test. For example:

```
app_name/
  /tests/
    __init__.py
    test_models.py
    test_forms.py
    test_views.py
```

<br>

---

# <a name="2">2. setUp vs setUpClass vs setUpTestData</a>

> References:
> https://stackoverflow.com/questions/43594519/what-are-the-differences-between-setupclass-setuptestdata-and-setup-in-testcase

**Similarities:**
- **setUp()** is called before each method, **setUpClass()** and **setUpTestData()** method are called once for the class.
- **setUp()** and **setUpClass()** are Python‘s built-in methods in `unittest.TestCase`, **setUpTestData()** is a Django defined method in `django.test.testcases.TestCase`.

**Differences:**
- **setUp()** is called before every test function to set up any objects that may be modified by the test (every test function will get a "fresh" version of these objects).
- **setUpClass()** is used to perform some class-wide initialization (e.g. overriding settings, creating [database] connections, loading webdrivers).
- **setUpTestData()** is called once at the beginning of the test run for class-level setup. You'd use this to create objects that aren't going to be modified or changed in any of the test methods.

<br>

## 2.1 setUp

> References:
> https://docs.python.org/3/library/unittest.html#unittest.TestCase.setUp
> https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Testing#what_does_django_provide_for_testing

Method called to prepare the test fixture. This is called immediately before calling the test method.

`setUp` will be called before each test run, and should be used to prepare test dataset for each test run.

### For example:

```python
class YourTestClass(TestCase):

    def setUp(self):
        #Setup run before every test method.
        pass

    def tearDown(self):
        #Clean up run after every test method.
        pass

    def test_something_that_will_pass(self):
        self.assertFalse(False)

    def test_something_that_will_fail(self):
        self.assertTrue(False)
```

<br>

## 2.2 setUpClass

> References:
> https://docs.djangoproject.com/en/3.2/topics/testing/tools/#liveservertestcase
> https://docs.python.org/3/library/unittest.html#unittest.TestCase.setUpClass

A class method called before tests in an individual class are run.

`SimpleTestCase` and its subclasses (e.g. `TestCase`, …) rely on `setUpClass()` and `tearDownClass()` to perform some class-wide initialization (e.g. overriding settings). If you need to override those methods, don’t forget to call the `super` implementation:

```python
class MyTestCase(TestCase):

    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        ...

    @classmethod
    def tearDownClass(cls):
        ...
        super().tearDownClass()
```

`setUpClass` is used to perform class-wide initialization/configuration (e.g. creating connections, loading webdrivers). When using `setUpClass` for instance to open database connection/session you can use `tearDownClass` to close them.

### For example:

```python
from django.contrib.staticfiles.testing import StaticLiveServerTestCase
from selenium.webdriver.firefox.webdriver import WebDriver

class MySeleniumTests(StaticLiveServerTestCase):
    fixtures = ['user-data.json']

    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.selenium = WebDriver()
        cls.selenium.implicitly_wait(10)

    @classmethod
    def tearDownClass(cls):
        cls.selenium.quit()
        super().tearDownClass()

    def test_login(self):
        self.selenium.get('%s%s' % (self.live_server_url, '/login/'))
        username_input = self.selenium.find_element_by_name("username")
        username_input.send_keys('myuser')
        password_input = self.selenium.find_element_by_name("password")
        password_input.send_keys('secret')
        self.selenium.find_element_by_xpath('//input[@value="Log in"]').click()
```

<br>

## 2.3 setUpTestData

> References:
> https://docs.djangoproject.com/en/3.2/topics/testing/tools/#django.test.TestCase.setUpTestData

The class-level `atomic` block described above allows the creation of initial data at the class level, once for the whole `TestCase`. This technique allows for faster tests as compared to using `setUp()`.

Note that if the tests are run on a database with no transaction support (for instance, MySQL with the MyISAM engine), `setUpTestData()` will be called before each test, negating the speed benefits.

### django.test.testcases.TestCase Source code:

```python
# django/test/testcases
class TestCase(TransactionTestCase):
    ...
    @classmethod
        def setUpTestData(cls):
            """Load initial data for the TestCase."""
            pass
```

### For example:

```python
from django.test import TestCase

class MyTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Set up data for the whole TestCase
        cls.foo = Foo.objects.create(bar="Test")
        ...

    def test1(self):
        # Some test using self.foo
        ...

    def test2(self):
        # Some other test using self.foo
        ...
```

<br>

## 2.4 __init__ method

> References:
> https://stackoverflow.com/questions/17353213/init-for-unittest-testcase/17353262

`__init__()` method is not recommended for TestCase class

<br>

---

# <a name="3">3. How to run the tests</a>

## 3.1 Running all the tests

> This will discover all files named with the pattern `test*.py` under the current directory
>
> and run all tests defined using appropriate base classes:

```shell
$ python manage.py test
```

## 3.2 Running specific tests

> If you want to run a subset of your tests you can do so by specifying the full dot path to the package(s), module, `TestCase` subclass or method:

```shell
# Run the specified module
$ python3 manage.py test app_name.tests

# Run the specified module
$ python manage.py test app_name.tests.test_models

# Run the specified class
$ python manage.py test app_name.tests.test_models.ClassModelTest

# Run the specified method
$ python manage.py test app_name.tests.test_models.ClassModelTest.test_get_absolute_url
```

## 3.3 Showing more test information

If you want to get more information about the test run you can change the `verbosity`. For example, to list the test successes as well as failures (and a whole bunch of information about how the testing database is set up) you can set the verbosity to "2" as shown:

```shell
$ python manage.py test --verbosity 2
```

The allowed verbosity levels are 0, 1, 2, and 3, with the default being "1".

<br>

---

# <a name="4">4. TestCase testing order</a>

> References:
> https://stackoverflow.com/questions/2581005/django-testcase-testing-order

The order to execute is alphabetical.

For example: testTestA will be loaded first than testTestB.

```python
class Test(TestCase):
    def setUp(self):
        ...

    def testTestB(self):
        # test code

    def testTestA(self):
        # test code
```

A tenet of unit-testing is that each test should be independent of all others. If in your case the code in testTestA must come before testTestB, then you could combine both into one test:

```python
def testTestA_and_TestB(self):
    # test code from testTestA
    ...
    # test code from testTestB
```

or, perhaps better would be

```python
def TestA(self):
    # test code
def TestB(self):
    # test code
def test_A_then_B(self):
    self.TestA()
    self.TestB()
```

The `Test` class only tests those methods who name begins with a lower-case `test...`. So you can put in extra helper methods `TestA` and `TestB` which won't get run unless you explicitly call them.

<br>

---

# <a name="5">5. Testing example</a>

## 5.1 Models tests example

```python
# /catalog/models.py

class Author(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    date_of_birth = models.DateField(null=True, blank=True)
    date_of_death = models.DateField('Died', null=True, blank=True)

    def get_absolute_url(self):
        return reverse('author-detail', args=[str(self.id)])

    def __str__(self):
        return f'{self.last_name}, {self.first_name}'
```

```python
# /catalog/tests/test_models.py

from django.test import TestCase

from catalog.models import Author

class AuthorModelTest(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Set up non-modified objects used by all test methods
        Author.objects.create(first_name='Big', last_name='Bob')

    def test_first_name_label(self):
        author = Author.objects.get(id=1)
        field_label = author._meta.get_field('first_name').verbose_name
        self.assertEqual(field_label, 'first name')

    def test_first_name_content(self):
        author = Author.objects.get(id=1)
        expected_object_name = f'{author.first_name}'
        self.assertEqual(expected_object_name, 'Big')

    def test_object_name_is_last_name_comma_first_name(self):
        author = Author.objects.get(id=1)
        expected_object_name = f'{author.last_name}, {author.first_name}'
        self.assertEqual(str(author), expected_object_name)

    def test_get_absolute_url(self):
        author = Author.objects.get(id=1)
        # This will also fail if the urlconf is not defined.
        self.assertEqual(author.get_absolute_url(), '/catalog/author/1')
```

<br>

## 5.2 Views tests example

```python
# /catalog/views.py

class AuthorListView(generic.ListView):
    """
    Let's start with one of our simplest views, which provides a list of all Authors.
    This is displayed at URL '/catalog/authors/' (an URL named 'authors' in the URL configuration). """
    model = Author
    paginate_by = 10
```

```python
# /catalog/tests/test_views.py

from django.test import TestCase
from django.urls import reverse

from catalog.models import Author

class AuthorListViewTest(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Create 13 authors for pagination tests
        number_of_authors = 13

        for author_id in range(number_of_authors):
            Author.objects.create(
                first_name=f'Christian {author_id}',
                last_name=f'Surname {author_id}',
            )

    def test_view_url_exists_at_desired_location(self):
        response = self.client.get('/catalog/authors/')
        self.assertEqual(response.status_code, 200)

    def test_view_url_accessible_by_name(self):
        response = self.client.get(reverse('authors'))
        self.assertEqual(response.status_code, 200)

    # Arguably if you trust Django then the only thing you need to test is
    # that the view is accessible at the correct URL and can be accessed using its name.
    # However if you're using a test-driven development process you'll start by writing tests
    # that confirm that the view displays all Authors, paginating them in lots of 10.

    def test_view_uses_correct_template(self):
        response = self.client.get(reverse('authors'))
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'catalog/author_list.html')

    def test_pagination_is_ten(self):
        response = self.client.get(reverse('authors'))
        self.assertEqual(response.status_code, 200)
        self.assertTrue('is_paginated' in response.context)
        self.assertTrue(response.context['is_paginated'] == True)
        self.assertEqual(len(response.context['author_list']), 10)

    def test_lists_all_authors(self):
        # Get second page and confirm it has (exactly) remaining 3 items
        response = self.client.get(reverse('authors')+'?page=2')
        self.assertEqual(response.status_code, 200)
        self.assertTrue('is_paginated' in response.context)
        self.assertTrue(response.context['is_paginated'] == True)
        self.assertEqual(len(response.context['author_list']), 3)
```

<br>

## 5.3 DRF API tests example

> https://www.django-rest-framework.org/api-guide/testing/#api-test-cases

```python
from django.urls import reverse
from rest_framework import status
from rest_framework.test import APITestCase

from myproject.apps.core.models import Account


class AccountTests(APITestCase):
    def test_create_account(self):
        """ Ensure we can create a new account object. """
        url = reverse('account-list')
        data = {'name': 'DabApps'}
        # The self.client attribute will be an APIClient (instead of Django's default Client) instance.
        response = self.client.post(url, data, format='json')
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Account.objects.count(), 1)
        self.assertEqual(Account.objects.get().name, 'DabApps')
        
    def test_retrieve_account(self):
        """ Checking the response data """
        response = self.client.get('/users/4/')
        self.assertEqual(response.data, {'id': 4, 'username': 'lauren'})
```
