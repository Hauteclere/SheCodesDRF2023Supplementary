# Deployment!
This is it, people: the moment we've all been waiting for! In this lesson, we go from having a local codebase that we can use for testing, to having a globally accessible back end that can be accessed from any computer in the world.

We are going to walk through the process of setting our app up to run in the cloud.  In the process we will no doubt run into some fun bugs, but that's the fun of coding!

> "The Cloud" is such a buzzword... really what it means is "on someone else's machine somewhere, which is being provided to us to run our code and make it available to the world". *Technically* it is possible to host a website from your own laptop, but it's totally impractical/insecure, and you'd never be able to turn the laptop off again or the website would go down. In the process we will no doubt run into some fun bugs, but that's the whole fun of coding!

For some of the code content in these notes I'll be using the `diff` file format. This is a way of expressing changes to files. Lines that are to be deleted are prepended with a `-` (minus) symbol and coloured red. Lines that are to be added are prepended with a `+` (plus) and are coloured green. You don't need to include the minuses and plusses in your code; it's just the best way to display these changes in Markdown on Github. 

```diff
-delete this line
+add this line
```

## Installation
First we have some things we need to install. Make sure you have our virtual environmet activated in the terminal and then navigate to the root folder (the one containing your `requirements.txt` file):

### Install `gunicorn`
That's "g-unicorn" not "gun-icorn". When I first read this I imagined a unicorn with a gun for a horn, but actually the G is short for "green". This library will be what actually serves the site for us, substituting in for the much less powerful server that is attached to the `runserver` command.
```Bash
python3 -m pip install gunicorn
```

### Install `whitenoise`
This is a library that will handle serving static files for us, if we ever have any. Some info on it here: https://pypi.org/project/whitenoise/

```Bash
python3 -m pip install whitenoise
```

### Install `CORS`
This is a security library to prevent people getting hacked on our site.
```Bash
python3 -m pip install django-cors-headers
```

### Add Those Three New Packages to your `requirements.txt`
```Bash
python3 -m pip freeze > requirements.txt
```

### Install Fly
This one is a little more complex - it's not a Python package, so we aren't installing it in our virtual environment. It's a terminal program that we are installing on our computers.

#### On Windows:

```cmd
iwr https://fly.io/install.ps1 -useb | iex
```

#### On Mac:
First make sure that `brew` is installed
```Bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh"
```
If this results in a successful installation you may be prompted to run two more commands - if so you should run those too!

Then use `brew` to install Fly:

```Bash
brew install flyctl
```

### Sign Up To Fly
First check that it is working by running
```Bash
fly help
```
If that works, activate an account with
```Bash
flyctl auth signup
```
At this point, you will need to provide a payment method. You **do** need to add a payment method here, even though we will be keeping our usage within their free-tier.

> Your first three projects (or VMs) are free with 3GB of storage total: https://fly.io/docs/about/pricing/#free-allowances <br><br> Over this allowance, you will be charged for the amount of data storage you use. If you accidentally go over, Fly is pretty good about refunding: https://fly.io/docs/about/credit-cards/

## Tweaking Settings
We are changing a few things. Let's talk through them before we modify the code.

### The Secret Key
Django needs a value for the `SECRET_KEY` variable in order to run, but once we've launched, we don't want to use the value that is stored in our code - this is visible on Github for anyone to steal. We'll tweak our settings so that Django tries to find a value stored on the computer it's running on first, and only uses the hardcoded value if it doesn't find anything. 

### The Debug Mode
While we are at it, we had also better make it so that Django doesn't give complex, developer-friendly error messages to the general public. Those tend to freak out the normies. We'll set it up so that it turns off the error messages by default, and only makes them available to us when we set a local variable for them.

Let's set this variable locally now before we forget. That way we will still see those useful errors when we are running the back end locally during development.

#### On Windows:
```cmd
$env:DJANGO_DEBUG = "True"
```
> If that didn’t work, follow these steps instead: https://helpdeskgeek.com/how-to/create-custom-environment-variables-in-windows/ (up to
and excluding the “Explorer” section). Then restart powershell and make sure you reactivate your virtual environment.

#### On Mac:
```Bash
export DJANGO_DEBUG=True
```

### The Static Directory
When `DJANGO_DEBUG` is turned off, Django expects us to have set up a `static/` directory. This is used to store files that need to be available for some projects, like CSS or favicons. We don't have those in this example project, but Django is a stickler. So we'll give it what it wants.

We don't want these files uploaded to our Github repo, though, so right now while we remember we should add the directory to our `.gitignore` file:

```diff
# .gitignore

venv/*
__pycache__/
*.sqlite3
*.vscode

# ...

+staticfiles
```

### The Middleware
We need to tell Django to use that `whitenoise` library we installed earlier, so we'll add it to our list of middleware.

### The Database Location
We are going to need to point Django to a different database location on our deployed app than it uses for the local version. That's because we won't be pointing it to a database on our own machine; it'll be a database on someone else's machine, and that database will be in a different spot.

### Some `CORS` Stuff
These are settings that prevent people from being tricked into taking actions on the site when malicious actors forward them there without their knowledge. It's security stuff!

### Ok Let's Make Those Tweaks!

Open your `settings.py` and make the following modifications (you can substitute the secret key string that is already in use for your project). 

```diff
+import os

# ...

-SECRET_KEY = '5*15pt5log&-bjpkqo0117!b!x4do-mgmxvg8n$3016384zz(7'
+SECRET_KEY = os.environ.get(
+   'DJANGO_SECRET_KEY',
+   '5*15pt5log&-bjpkqo0117!b!x4do-mgmxvg8n$3016384zz(7'
+)

# ...

-DEBUG = True
+DEBUG = os.environ.get(
+   'DJANGO_DEBUG', 
+   'False'
+) != 'False'

# ...

+STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

# ...

-ALLOWED_HOSTS = []
+ALLOWED_HOSTS = ['*']
+CORS_ALLOW_ALL_ORIGINS = True
+CSRF_TRUSTED_ORIGINS = ['https://*.fly.dev']

# ...

INSTALLED_APPS = [
    'projects.apps.ProjectsConfig',
    'users.apps.UsersConfig',
    'rest_framework',
    'rest_framework.authtoken',
+   'corsheaders',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'users.apps.UsersConfig',
]

# ...

MIDDLEWARE = [
+   'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
+   'whitenoise.middleware.WhiteNoiseMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# ...

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
-       'NAME': BASE_DIR / 'db.sqlite3',
+       'NAME': os.environ.get('DATABASE_DIR', BASE_DIR / 'db.sqlite3'),
    }
}

```
Since we made the change to the settings for static files, we should run a command to set up that `/static/` folder:

```Bash
python3 ./crowdfunding/manage.py collectstatic
```
> Note that we are using a slightly different way of calling `manage.py` here - that's because we are one folder up from it. You could navigate to the folder it is in and run it from there, but I'm trying to keep this simple by making it so that all our commands get run from the same location.

Now try doing a `runserver` to check for errors:
```Bash
python3 ./crowdfunding/manage.py runserver
```

## Create a Fly Project
Use the following command:
```Bash
fly launch
```
It will ask you if you want to create a Postgres database or an Upstash Redis database. **Answer no to both.**

Your project name will be something funky like `weathered-surf-123` or `hidden-thunder-456` unless you choose a custom name. Using the default name is fine - it's sometimes hard to pick a name that isn't already in use by someone. Make sure you can see the project in your dashboard: https://fly.io/apps/

After this command has finished running, you will notice that a couple of new files have been created in your project directory:
- `dockerfile`
- `fly.toml`
- `.dockerignore`

We will be customising some of these to set up our deployment. 

### Customising The `dockerfile`
```diff
ARG PYTHON_VERSION=3.10-slim-buster

FROM python:${PYTHON_VERSION}

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

RUN mkdir -p /code

WORKDIR /code

COPY requirements.txt /tmp/requirements.txt

RUN set -ex && \
    pip install --upgrade pip && \
    pip install -r /tmp/requirements.txt && \
    rm -rf /root/.cache/

-COPY . /code/
+COPY crowdfunding/ /code/

RUN python manage.py collectstatic --noinput
RUN chmod +x /code/run.sh

EXPOSE 8000

# replace demo.wsgi with <project_name>.wsgi
-CMD ["gunicorn", "--bind", ":8000", "--workers", "2", "demo.wsgi"]
+CMD ["/code/run.sh"]
```

### Customizing `fly.toml`
```diff
# fly.toml

#...

[[statics]]
    guest_path = "/app/public"
    url_prefix = "/static/"
+[mounts]
+   source = "dbvol"
+   destination = "/dbvol"

```

### Creating A File To Run The Server For Us
When we run the server in production we won't use the `manage.py runserver` command - that's just for development.

Instead, we'll be using the `gunicorn` library that we installed earlier. To set it up so that the cloud machine we are deploying to automatically starts up `gunicorn` for us, we will write a file called `run.sh`. You can see we've referenced it already up there in our `dockerfile`.

Create the file next to your `manage.py` file, and make sure you call it `run.sh`. Here's what to put in it (the comment is actually important to include!):

```Bash
#!/usr/bin/env bash
python manage.py migrate
python manage.py createsuperuser --no-input
gunicorn --bind :8000 --workers 1 crowdfunding.wsgi
```

### Setting Up Some Variables And Settings For Fly
#### Creating A Volume To Host The Database:
> **You need to substitute in the funky name that Fly generated for your app here!** For instance, mine was `quiet-tree-5089`
```Bash
fly volumes create -a <app-name> -r syd --size 1 dbvol
```

#### Noting The Location Of That Volume With Fly
```Bash
flyctl secrets set DATABASE_DIR='/dbvol/db.sqlite3'
```
#### Setting Some Superuser Credentials
> **You need to substitute in an email address and secure password here!**
```Bash
flyctl secrets set DJANGO_SUPERUSER_USERNAME='admin'
```
```Bash
flyctl secrets set DJANGO_SUPERUSER_EMAIL='<your-email-address>'
```
```Bash
flyctl secrets set DJANGO_SUPERUSER_PASSWORD='<secure-password>'
```

#### Setting That Debug Setting From Earlier
Technically, we've made it so that you don't need to set this in deployment, but it never hurts to be explicit.
```Bash
flyctl secrets set DJANGO_DEBUG=False
```

#### Setting The SECRET_KEY
This bit is pretty important. We Need to pick a secure secret key, and also not write it down or back it up anywhere. Not even we need to know this - it's only for the deployed machine.

You can generate one here: https://djecrety.ir/

> **Make sure you substitute the value in here!**

```Bash
flyctl secrets set DJANGO_SECRET_KEY='<your-secret-key>'
```

## Deploy The Fly Project!
One command:
```Bash
fly deploy
```
This takes some time, let it run and watch to see if it works... :)

If everything ran successfully, go to `https://<your_project_name>.fly.dev/projects/` to see if your site is running! 

Did it work? Mine did. That thrill never gets old. I dunno about you but I feel like this:
![In place of a Dark Lord you would have a Queen! Not dark but beautiful and terrible as the Dawn! Treacherous as the Seas! Stronger than the foundations of the Earth!](img/a_queen.jpg)

If it didn't work, grab a mentor and let's get bugshooting!

If your site is up and running, try navigating to an endpoint that doesn't exist; you should get a plain boring error message, instead of the detailed ones we are used to.

If you're stuck, have a look at my working code here and see what's different: https://github.com/SheCodesAus/she-codes-crowdfunding-api-project-Hauteclere/tree/test_deploy




