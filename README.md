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
