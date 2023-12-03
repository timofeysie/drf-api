![CI logo](https://codeinstitute.s3.amazonaws.com/fullstack/ci_logo_small.png)

# Django Rest Framework API

The DRF-API project is based on the Code Institute Django REST Framework module.

## Workflow

```shell
python manage.py runserver
python manage.py migrate
python manage.py makemigrations xyz
python manage.py shell # Use quit() or Ctrl-Z plus Return to exit
python manage.py createsuperuser
python manage.py test zyx # run the sample test
pip freeze > requirements.txt # update dependencies file
```

## Starting the project

Install the specific version of Django needed for the course.

Start the project and install the dependencies.

```shell
> pip3 install 'django<4'
> django-admin startproject drf_api .
> pip install django-cloudinary-storage
> pip install pillow
Requirement already satisfied: Pillow in c:\users\timof\appdata\local\programs\python\python310\lib\site-packages (10.1.0)
```

The Django Cloudinary storage library connects Django to a service that will host the images for the API.

The Pillow library adds image processing capabilities that needed for working with Cloudinary.

Add the apps to the drf_api\settings.py file.  The app names need to be in this particular order

with django.contrib.staticfiles between  cloudinary_storage and Cloudinary.

drf_api\settings.py

```py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'cloudinary_storage',
    'django.contrib.staticfiles',
    'cloudinary',
]
```

Next create the env.py file in the top directory to use the Cloudinary API key:

```py title="env.py"
import os
os.environ['CLOUDINARY_URL'] = 'CLOUDINARY_URL=cloudinary://...'
```

In settings.py again load the environment variable needed to set a variable called CLOUDINARY_STORAGE.
Use the environment variable set in the env.py file to declare that value.
Define a setting called MEDIA_URL, which is the standard Django folder to store media so the settings know where to put image files.
Also set a DEFAULT_FILE_STORAGE variable.

```py title="drf_api\settings.py"
from pathlib import Path
import os

if os.path.exists('env.py'):
    import env

CLOUDINARY_STORAGE = {
    'CLOUDINARY_URL': os.environ.get('CLOUDINARY_URL')
}

MEDIA_URL = '/media/'
DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'
```

## Create the profile app

```shell
python manage.py startapp profiles
Traceback (most recent call last):
  File "C:\Users\timof\repos\django\drf-api\manage.py", line 22, in <module>
    main()
  File "C:\Users\timof\repos\django\drf-api\manage.py", line 18, in main
    execute_from_command_line(sys.argv)
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\django\core\management\__init__.py", line 419, in execute_from_command_line
    utility.execute()
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\django\core\management\__init__.py", line 395, in execute
    django.setup()
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\django\__init__.py", line 24, in setup
    apps.populate(settings.INSTALLED_APPS)
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\django\apps\registry.py", line 91, in populate
    app_config = AppConfig.create(entry)
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\django\apps\config.py", line 224, in create
    import_module(entry)
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\importlib\__init__.py", line 126, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
  File "<frozen importlib._bootstrap>", line 1050, in _gcd_import
  File "<frozen importlib._bootstrap>", line 1027, in _find_and_load
  File "<frozen importlib._bootstrap>", line 1006, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 688, in _load_unlocked
  File "<frozen importlib._bootstrap_external>", line 883, in exec_module
  File "<frozen importlib._bootstrap>", line 241, in _call_with_frames_removed        
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\cloudinary\__init__.py", line 219, in <module>
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\cloudinary\__init__.py", line 216, in _load_config_from_env
    self._load_from_url(os.environ.get("CLOUDINARY_URL"))
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\cloudinary\__init__.py", line 166, in _load_from_url
    return self._setup_from_parsed_url(parsed_url)
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\cloudinary\__init__.py", line 154, in _setup_from_parsed_url
    config_from_parsed_url = self._config_from_parsed_url(parsed_url)
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\cloudinary\__init__.py", line 189, in _config_from_parsed_url
    raise ValueError("Invalid CLOUDINARY_URL scheme. Expecting to start with 'cloudinary://'")
ValueError: Invalid CLOUDINARY_URL scheme. Expecting to start with 'cloudinary://'
```

The above error is because the copied key from cloudinary contained the key name.

Change this:

```py
os.environ['CLOUDINARY_URL'] = 'CLOUDINARY_URL=cloudinary:/...```
```

to this:

```py
os.environ['CLOUDINARY_URL'] = 'cloudinary:/
```

Then the command works as expected.

### The Profile Model

Create our Profile model with a one-to-one field pointing to a User instance and store the images in the database.

Create a Meta class that will return a Profile instances with most recently created is first.  

In the dunder string method return  information about who the profile owner is.

To ensure that a  profile is created every time a user is created use signals notifications that get triggered when a  
user is created.

```py title=models.py
from django.db import models
from django.db.models.signals import post_save
from django.contrib.auth.models import User

class Profile(models.Model):
    owner = models.OneToOneField(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    name = models.CharField(max_length=255, blank=True)
    content = models.TextField(blank=True)
    image = models.ImageField(
        upload_to='images/', default='../cld-sample-5'
    )

    class Meta:
        ordering = ['-created_at']

    def __str__(self):
        return f"{self.owner}'s profile"

def create_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(owner=instance)

post_save.connect(create_profile, sender=User)
```

Note the original file is [located in the moments repo](https://github.com/Code-Institute-Solutions/drf-api/blob/master/profiles/models.py).

Register the Profile model in admin.py:2

```py
from django.contrib import admin
from .models import Profile

admin.site.register(Profile)
```

Then run make migrations whihc you have to do after updating a model:

```shell
python manage.py makemigrations
```

Create a admin user and provide a password:

```shell
python manage.py createsuperuser

You have 19 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, profiles, sessions.
Run 'python manage.py migrate' to apply them.
Traceback (most recent call last):
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\django\db\backends\utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\django\db\backends\sqlite3\base.py", line 423, in execute
    return Database.Cursor.execute(self, query, params)
sqlite3.OperationalError: no such table: auth_user

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "C:\Users\timof\repos\django\drf-api\manage.py", line 22, in <module>
    main()
  File "C:\Users\timof\repos\django\drf-api\manage.py", line 18, in main
    execute_from_command_line(sys.argv)
  ...
    return self.cursor.execute(sql, params)
  File "C:\Users\timof\AppData\Local\Programs\Python\Python310\lib\site-packages\django\db\backends\sqlite3\base.py", line 423, in execute
    return Database.Cursor.execute(self, query, params)
django.db.utils.OperationalError: no such table: auth_user
```

This error was becuase I missed the second command:

```shell
python manage.py makemigrations
python manage.py migrate
```

Run the server:

```shell
python manage.py runserver
```

Goto the admin url: http://127.0.0.1:8000/admin

Create a file with the dependencies:

```shell
pip freeze > requirements.txt
```

## Rest Framework Serializers

Currently when running the server, there are no REST API endpoints:

```shell
PS C:\Users\timof\repos\django\drf-api> python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
December 02, 2023 - 11:19:46
Django version 3.2, using settings 'drf_api.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
Not Found: /profiles
[02/Dec/2023 11:20:00] "GET /profiles HTTP/1.1" 404 1987
Not Found: /favicon.ico
[02/Dec/2023 11:20:00] "GET /favicon.ico HTTP/1.1" 404 1996
Not Found: /profiles/
[02/Dec/2023 11:21:30] "GET /profiles/ HTTP/1.1" 404 1990
```

### Install the the Django REST Framework

```shell
pip install djangorestframework
Collecting djangorestframework
  Downloading djangorestframework-3.14.0-py3-none-any.whl (1.1 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.1/1.1 MB 11.2 MB/s eta 0:00:00
Requirement already satisfied: pytz in c:\users\timof\appdata\local\programs\python\python310\lib\site-packages (from djangorestframework) (2023.3.post1)
Requirement already satisfied: django>=3.0 in c:\users\timof\appdata\local\programs\python\python310\lib\site-packages (from djangorestframework) (3.2)
Requirement already satisfied: sqlparse>=0.2.2 in c:\users\timof\appdata\local\programs\python\python310\lib\site-packages (from django>=3.0->djangorestframework) (0.4.4)
Requirement already satisfied: asgiref<4,>=3.3.2 in c:\users\timof\appdata\local\programs\python\python310\lib\site-packages (from django>=3.0->djangorestframework) (3.7.2)
Requirement already satisfied: typing-extensions>=4 in c:\users\timof\appdata\local\programs\python\python310\lib\site-packages (from asgiref<4,>=3.3.2->django>=3.0->djangorestframework) (4.8.0)
Installing collected packages: djangorestframework
Successfully installed djangorestframework-3.14.0
[notice] A new release of pip is available: 23.0.1 -> 23.3.1
[notice] To update, run: python.exe -m pip install --upgrade pip
```

Add it after cloudinary at the bottom of the installed apps array in settings.py:

```py
INSTALLED_APPS = [
    ...
    'cloudinary',
    'rest_framework',

    'profiles',
]
```

### Import the APIView and Response classes in views.py

APIView is very similar to Django’s View  class. It also provides a few bits of extra  
functionality such as making sure you  receive Request instances in your view,  
handling parsing errors, and adding  context to Response objects.

Create the ProfileList view and define the get method.

profiles\views.py

```py
from rest_framework.views import APIView
from rest_framework.response import Response
from .models import Profile

class ProfileList(APIView):
    def get(self, request):
        profiles = Profile.objects.all()
        return Response(profiles)
```

### Create profile urls

create a urls.py file. Inside, we’ll  import Django’s ‘path’ and views from profiles.
We have just one view, hence just one url pattern.  
ProfileList is a class view, so  remember to call as_view on it.

profiles\urls.py

```py
from django.urls import path
from profiles import views

urlpatterns = [
    path('profiles/', views.ProfileList.as_view()),
]
```

include profile urls in  our main app

drf_api\urls.py

```py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('profiles.urls')),
]
```

Currently if you start the server and go to ‘profiles/’ we will see the error: Object of type Profile is not JSON serializable.

When a user posts data to an API, the following has to happen:

- data is deserialized (converted from a data format like JSON or  XML to Python native data types)
- validated
- the model instance is saved in the database
- a queryset or model instance  is returned from the database
- it is converted again, or serialized to a JSON

The error indicates we need a serializer to convert Django model instances to JSON

### Create the serializer

Creating serializers.py, import serializers from  rest framework and our Profile model.

Specify ‘owner’ as a ReadOnlyField and populate it with the owner's username.

In the Meta class, we’ll point to our Profile model and specify the fields we’d like to  
include in the response.

profiles\serializers.py

```py
from rest_framework import serializers
from .models import Profile

class ProfileSerializer(serializers.ModelSerializer):
    owner = serializers.ReadOnlyField(source='owner.username')

    class Meta:
        model = Profile
        fields = [
            'id', 'owner', 'created_at', 'updated_at', 'name',
            'content', 'image',
        ]
```

When extending Django's model class using models.models, the id field is created automatically

### Add the serializer to our views.py file

Import the ProfileSerializer, create a ProfileSerializer instance and pass in profiles and many equals True to specify we’re serializing multiple Profile instances.
In the Response send data returned from our serializer.

profiles\views.py

```py
...
from .serializers import ProfileSerializer

class ProfileList(APIView):
    def get(self, request):
        profiles = Profile.objects.all()
        serializer = ProfileSerializer(profiles, many=True)
        return Response(serializer.data)
```

Now with the server running refresh the preview window and the JSON user list is returned.

Time to update dependencies, git add, commit and push all the changes to GitHub.

## Populating Serializer ReadOnly Field using dot notation

This section begins with a discussion of the ‘source’ attribute in the serializer.

In the ProfileSerializer, dot notation is used to populate the owner ReadOnlyField:

```py
class ProfileSerializer(serializers.ModelSerializer):
    owner = serializers.ReadOnlyField(source='owner.username')
```

The User and Profile tables are connected through the owner ```OneToOne``` field in the models.py:

```py
class Profile(models.Model):
    owner = models.OneToOneField(User, on_delete=models.CASCADE)
```

By default, the owner field always returns the user’s id value.

To make this clear we can overwrite the default behavior to return username instead using ```owner.username```

How you would access the profile name field if you were working in the Post serializer?

```py
profile_name = serializers.ReadOnlyField(source='owner.profile.name')
```

To add the profile image field to each post, we need to access a sub-attribute, so it would look like this:

```py
profile_image = serializers.ReadOnlyField(source='owner.profile.image.url')
```

## Profile Details View CRUD: GET and PUT

The REST endpoints for the profiles API look like this:

### List View

- /profiles GET list profiles
- /profiles POST create profile

### Detail View

- /profiles/:id GET get profile by id
- /profiles/:id PUT update profile by id
- /profiles/:id DELETE profile by id

The delete endpoints will not be covered in this module, so now we will implement the GET & PUT endpoints.

I notice that the ProfileDetail class in [the repo](https://github.com/Code-Institute-Solutions/drf-api/blob/master/profiles/views.py) is a bit different from what is shown in the tutorial.

In this project, we use ```APIView```.  Note pk = Primary Key.

```py
class ProfileDetail(APIView):
    def get_object(self, pk):
        try:
            profile = Profile.objects.get(pk=pk)
            return profile
        except Profile.DoesNotExist:
            raise Http404
```

In the repo, it has ```generics.RetrieveUpdateAPIView```

```py
class ProfileDetail(generics.RetrieveUpdateAPIView):
    permission_classes = [IsOwnerOrReadOnly]
    queryset = Profile.objects.annotate(
        posts_count=Count('owner__post', distinct=True),
        followers_count=Count('owner__followed', distinct=True),
        following_count=Count('owner__following', distinct=True)
    ).order_by('-created_at')
    serializer_class = ProfileSerializer
```

I guess that will come when permissions are introduced later.

Not that it says "raise Http404".  I mistakenly used 'return' at first and it would return a 200 instead of a 404.

The GET looks like this:

```py
    def get(self, request, pk):
        profile = self.get_object(pk)
        serializer = ProfileSerializer(profile)
        return Response(serializer.data)
```

I'm not exactly sure why the pk is used differently in both methods:

- profile = Profile.objects.get(pk=pk)
- profile = self.get_object(pk)

Next, add the details url in profiles/urls.py to use the new class:

```py
urlpatterns = [
    path('profiles/', views.ProfileList.as_view()),
    path('profile/<int:pk>/', views.ProfileDetail.as_view()),
]
```

After this, running the server again, the profiles/ returns this:

```json
[
    {
        "id": 1,
        "owner": "timof",
        "created_at": "2023-12-02T01:18:52.483047Z",
        "updated_at": "2023-12-02T01:18:52.483047Z",
        "name": "",
        "content": "",
        "image": "https://res.cloudinary.com/dr3am91m4/image/upload/v1/media/../cld-sample-5"
    }
]
```

The profile/1 returns:

```json
{
    "id": 1,
    "owner": "timof",
    "created_at": "2023-12-02T01:18:52.483047Z",
    "updated_at": "2023-12-02T01:18:52.483047Z",
    "name": "",
    "content": "",
    "image": "https://res.cloudinary.com/dr3am91m4/image/upload/v1/media/../cld-sample-5"
}
```

## Useful links

- The official docs for [Django REST framework](https://www.django-rest-framework.org/)
- The [quick-start tutorial](https://www.django-rest-framework.org/tutorial/quickstart/): a simple API to allow admin users to view and edit the users and groups
- The [Tutorial 1: Serialization](https://www.django-rest-framework.org/tutorial/1-serialization/):  a simple pastebin code highlighting Web API

## Original Readme

Welcome USER_NAME,

This is the Code Institute student template for Gitpod. We have preinstalled all of the tools you need to get started. It's perfectly ok to use this template as the basis for your project submissions.

You can safely delete this README.md file, or change it for your own project. Please do read it at least once, though! It contains some important information about Gitpod and the extensions we use. Some of this information has been updated since the video content was created. The last update to this file was: **September 1, 2021**

## Gitpod Reminders

To run a frontend (HTML, CSS, Javascript only) application in Gitpod, in the terminal, type:

`python3 -m http.server`

A blue button should appear to click: _Make Public_,

Another blue button should appear to click: _Open Browser_.

To run a backend Python file, type `python3 app.py`, if your Python file is named `app.py` of course.

A blue button should appear to click: _Make Public_,

Another blue button should appear to click: _Open Browser_.

In Gitpod you have superuser security privileges by default. Therefore you do not need to use the `sudo` (superuser do) command in the bash terminal in any of the lessons.

To log into the Heroku toolbelt CLI:

1. Log in to your Heroku account and go to *Account Settings* in the menu under your avatar.
2. Scroll down to the *API Key* and click *Reveal*
3. Copy the key
4. In Gitpod, from the terminal, run `heroku_config`
5. Paste in your API key when asked

You can now use the `heroku` CLI program - try running `heroku apps` to confirm it works. This API key is unique and private to you so do not share it. If you accidentally make it public then you can create a new one with _Regenerate API Key_.

------

## Release History

We continually tweak and adjust this template to help give you the best experience. Here is the version history:

**September 20 2023:** Update Python version to 3.9.17.

**September 1 2021:** Remove `PGHOSTADDR` environment variable.

**July 19 2021:** Remove `font_fix` script now that the terminal font issue is fixed.

**July 2 2021:** Remove extensions that are not available in Open VSX.

**June 30 2021:** Combined the P4 and P5 templates into one file, added the uptime script. See the FAQ at the end of this file.

**June 10 2021:** Added: `font_fix` script and alias to fix the Terminal font issue

**May 10 2021:** Added `heroku_config` script to allow Heroku API key to be stored as an environment variable.

**April 7 2021:** Upgraded the template for VS Code instead of Theia.

**October 21 2020:** Versions of the HTMLHint, Prettier, Bootstrap4 CDN and Auto Close extensions updated. The Python extension needs to stay the same version for now.

**October 08 2020:** Additional large Gitpod files (`core.mongo*` and `core.python*`) are now hidden in the Explorer, and have been added to the `.gitignore` by default.

**September 22 2020:** Gitpod occasionally creates large `core.Microsoft` files. These are now hidden in the Explorer. A `.gitignore` file has been created to make sure these files will not be committed, along with other common files.

**April 16 2020:** The template now automatically installs MySQL instead of relying on the Gitpod MySQL image. The message about a Python linter not being installed has been dealt with, and the set-up files are now hidden in the Gitpod file explorer.

**April 13 2020:** Added the _Prettier_ code beautifier extension instead of the code formatter built-in to Gitpod.

**February 2020:** The initialisation files now _do not_ auto-delete. They will remain in your project. You can safely ignore them. They just make sure that your workspace is configured correctly each time you open it. It will also prevent the Gitpod configuration popup from appearing.

**December 2019:** Added Eventyret's Bootstrap 4 extension. Type `!bscdn` in a HTML file to add the Bootstrap boilerplate. Check out the <a href="https://github.com/Eventyret/vscode-bcdn" target="_blank">README.md file at the official repo</a> for more options.

------

## FAQ about the uptime script

**Why have you added this script?**

It will help us to calculate how many running workspaces there are at any one time, which greatly helps us with cost and capacity planning. It will help us decide on the future direction of our cloud-based IDE strategy.

**How will this affect me?**

For everyday usage of Gitpod, it doesn’t have any effect at all. The script only captures the following data:

- An ID that is randomly generated each time the workspace is started.
- The current date and time
- The workspace status of “started” or “running”, which is sent every 5 minutes.

It is not possible for us or anyone else to trace the random ID back to an individual, and no personal data is being captured. It will not slow down the workspace or affect your work.

**So….?**

We want to tell you this so that we are being completely transparent about the data we collect and what we do with it.

**Can I opt out?**

Yes, you can. Since no personally identifiable information is being captured, we'd appreciate it if you let the script run; however if you are unhappy with the idea, simply run the following commands from the terminal window after creating the workspace, and this will remove the uptime script:

```
pkill uptime.sh
rm .vscode/uptime.sh
```

**Anything more?**

Yes! We'd strongly encourage you to look at the source code of the `uptime.sh` file so that you know what it's doing. As future software developers, it will be great practice to see how these shell scripts work.

---

Happy coding!
