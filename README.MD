1------------------------DJANGO ----------------------------->
   --- Installation --------------->

      -> Python
      -> pip3 install pipenv
      -> pipenv shell // To activate the virtual environment
      -> django-admin startproject storefront . // to create a new project in the current Dir
      -> python manage.py runserver // to start the server
      -> Control C to stop the server

   NB: In this stage, you have a simple Django project running.
   
   --- Django App ----------------->
   
    - Pipfile is just a configuration of changing the django version...
    - Python manage.py startapp name -> To create a new app in Django
    - Every App created must be registered in the setting section before used.

   --- App Folder files ------------>
   
    -> Migrations for generating database tables
    -> Admin for the admin interface
    -> apps for configuring the app
    -> Models for creating app models
    -> Tests for testing the app components
    -> Views for the views request handler of the app
       They takes request and return a response

   --- Workflow ------------>
    
    Anytime we create an app by using python manage.py startapp name
    we have to:
       -> Register the app in the settings
       -> Create a function request handler in the views module
       -> Create urls.py folder in the app folder to hhandle all the URLS of the app
       -> Register the app in the central Urls modules of the base project by:
          -> adding the include to path
          -> path('playground/', include('playground.urls'))
          
   --- Templates ------------>
    
    - Create a template folder inside of your app where we can add html code
    - We can the return  return render(request, 'hello.html') in the views module

2------------------- DEBUGGING DJANGO APP WITH DEBUG TOOLBAR ----------------------------->

    - Documentation: django-debug-toolbar
    - Installation: pipenv install django-debug-toolbar
    - Add 'debug_toolbar' in the list of INSTALLLED _APP in the settings module
    - Add path('__debug__/', include(debug_toolbar.urls)) in the urls in the base project folder
       import the module: import debug_toolbar
    - Add this middleware in the settings module:
         'debug_toolbar.middleware.DebugToolbarMiddleware'
    - Final Step add the IP address: INTERNAL_IPS = [ '127.0.0.1',] somewhere in the settings module
    NB: The toolbar will only appear if we return a complete formated html files

3------------------- BUILDING THE DATA MODELS ----------------------------------------------->

    -> A relation between two entities can be:
          - One-to-One
          - One-to-many
          - Many-to-Many
    - In this section, the django docs is your friend: docs.djangoproject.com
    - All the data models might be created inside of the app models module
    - When creating a models in django, the ID field is created automatically by django
    - If we want to create our own Primary key, id = models.CharFiled(max_lemgth=10, primary_key=True

  --- Relationship One-to-One Customer -> Address ------------>

      We want to have a relationship between Customer the parent class and Address the child class

      Parent Class
      class Customer(models.Model):
          MEMBERSHIP_BRONZE = 'B'
          MEMBERSHIP_SILVER = 'S'
          MEMBERSHIP_GOLD = 'G'
          MEMBERSHIP_CHOICES = [
              (MEMBERSHIP_BRONZE, 'Bronze'),
              (MEMBERSHIP_SILVER, 'Silver'),
              (MEMBERSHIP_GOLD, 'Gold')
          ]
          first_name = models.CharField(max_length=255)
          last_name = models.CharField(max_length=255)
          email = models.EmailField(unique=True)
          phone = models.CharField(max_length=255)
          birth_date = models.DateField(null=True)
          membership = models.CharField(max_length=1, choices=MEMBERSHIP_CHOICES, default=MEMBERSHIP_BRONZE)

      Child Class
      class Address(models.Model):
          street = models.CharField(max_length=255)
          city = models.CharField(max_length=255)
          customer = models.OneToOneField(Customer, on_delete=models.CASCADE, primary_key=True)

      NB: on_delete=models.CASCADE -> Will delete the address when we delete the customer
          on_delete=models.SET_NULL -> Will put NUll in the address field when we delete the customer
          on_delete=models.SET_DEFAULT -> Will put to it default values in the address field when we delete the customer
          on_delete=models.SET_PROTECT -> Will prevent the deletion to delete the child first
      NB: Don't forget to set the primary_key= True when we use CASCADE 
          Lucky for us the reverse relationship doesn't not exist and we don't have to implement it.

  --- Relationship One-to-many Customer -> Address ------------>
   
      Child Class
      class Address(models.Model):
          street = models.CharField(max_length=255)
          city = models.CharField(max_length=255)
          customer = models.ForeignKey(Customer, on_delete=models.CASCADE)

  --- Relationship Many-to-many Promotion -> Product ------------>
 
    class Promotion(models.Model):
       description = models.CharField(max_length=255)
       discount = models.FloatField()

    class Product(models.Model):
       title = models.CharField(max_length=255)
       description = models.TextField()
       price = models.DecimalField(max_digits=6, decimal_places=2)
       inventory = models.IntegerField()
       last_update = models.DateTimeField(auto_now=True)
       collection = models.ForeignKey(Collection, on_delete=models.PROTECT)
       # If we delete the collection, the product will be there
       promotions = models.ManyToManyField(Promotion) # Many-to-Many

  --- CIRCULAR DEPENDENCY Promotion -> Product ------------>
  
    - Let say we have a reverse dependency between Collection and Product:
    - This is how to solve the problem.

       class Collection(models.Model):
       title = models.CharField(max_length=255)
       featured_Product = models.ForeignKey('Product', on_delete=models.SET_NULL, null=True, related_name='+')


       class Product(models.Model):
          title = models.CharField(max_length=255)
          description = models.TextField()
          price = models.DecimalField(max_digits=6, decimal_places=2)
          inventory = models.IntegerField()
          last_update = models.DateTimeField(auto_now=True)
          collection = models.ForeignKey(Collection, on_delete=models.PROTECT)
          # If we delete the collection, the product will be there
          promotions = models.ManyToManyField(Promotion)

  --- Generics relationship using ContentType ------------>

     - The concept of Generic relation is based on king and it successor
          For example: tag the king and tagItem the successor
     - Import it: from django.contrib.contenttypes.models import ContentType
     - Define these three relations:
         content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE) -> Get the type
         object_id = models.PositiveIntegerField() -> Get the id of the object the tag is pplied to
         content_object = GenericForeignKey() -> Get the object the tag is applied to

4------------------- DATABASES USING DJANGO MIGRATIONS -------------------------------->

  --- Databases ------------>

     -> PostgreSQL
     -> MySQL
     -> MariaDB
     -> Oracle

  --- Migrations ------------>
 
    - Run migrations: python manage.py makemigrations
       -> Django will create a migrations for each indivudual app
       -> A migration folder will be created inside of all app

  --- Run a Migrations ------------>
   
    -> python manage.py migrate -> To migrate 
    -> python manage.py migrate store 0003 -> to downgrade to this version
       But don't forget to delete all the files and code of the deleted version

  --- Installation of mysql ------------>
  
    -> brew install mysql
    -> brew info mysql -> to see the version
    -> mysql_secure_installation -> Set up a password
    -> brew services list -> to see all the services running
    -> brew services start mysql -> to start mysql server
    -> mysql -u root -> to connect
    -> mysql show databases; -> to show all the db
    -> exit to quit the database
    -> If you set up a password, mysql -u root -p and enter the pwd
  
  --- Datagrip configuration intelliJ------------>
  
    -> Create a database after installing mysql
    -> Connect to the database
  
  --- Connect the Django Project to mysql------------>
  
    -> Install: pipenv install mysqlclient 
    -> Change the database setting in settings module in the base project
        - 'ENGINE': 'django.db.backends.mysql,
        - 'NAME': name of the database,
        - 'HOST': 'localhost',
        - 'User': 'root',
        - 'PASSWORD': ''
    -> If everything is Ok, run migrate to create all the table in the database
        python manage.py migrate
        All the table should be created in the database

  --- Altering the tables ------------>
  
    -> Create an empty migration:
       pyhton manage.py makemigrations store --empty
    -> Make any changes in the operations array
    operations = [
       migrations.RunSQL("""
           INSERT INTO store_colletion (title) VALUES ('Collection1')
       """, """
            DELETE FROM store_collection WHERE title='collection1'
       """)
    ]
    -> Run python manage.py migrate 
    -> We can also downgrade to the previous one
      