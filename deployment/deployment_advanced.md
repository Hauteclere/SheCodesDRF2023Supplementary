# Deployment!

- [Deployment!](#deployment)
  - [Phase 0: Commit!](#phase-0-commit)
  - [Phase 1: Installation](#phase-1-installation)
    - [1.1: Install `gunicorn`](#11-install-gunicorn)
    - [1.2: Install `whitenoise`](#12-install-whitenoise)
    - [1.3: Install `CORS`](#13-install-cors)
    - [1.4: Add Those Three New Packages to your `requirements.txt`](#14-add-those-three-new-packages-to-your-requirementstxt)
    - [1.5: Install Fly](#15-install-fly)
      - [1.5.a - On Windows:](#15a---on-windows)
      - [1.5.b - On Mac:](#15b---on-mac)
    - [1.6: Sign Up To Fly](#16-sign-up-to-fly)
  - [Phase 2: Tweaking Settings](#phase-2-tweaking-settings)
    - [2.1: The Secret Key](#21-the-secret-key)
    - [2.2: The Debug Mode](#22-the-debug-mode)
      - [2.2.a - On Windows:](#22a---on-windows)
      - [2.2.b - On Mac:](#22b---on-mac)
    - [2.3: The Static Directory](#23-the-static-directory)
    - [2.4: The Middleware](#24-the-middleware)
    - [2.5: The Database Location](#25-the-database-location)
    - [2.6: Some `CORS` Stuff](#26-some-cors-stuff)
    - [2.7: Ok Let's Make Those Tweaks!](#27-ok-lets-make-those-tweaks)
  - [Phase 3: Create a Fly Project](#phase-3-create-a-fly-project)
    - [3.1: Launch!](#31-launch)
    - [3.2: Customising The `dockerfile`](#32-customising-the-dockerfile)
    - [3.3: Customizing `fly.toml`](#33-customizing-flytoml)
    - [3.4: Creating A File To Run The Server For Us](#34-creating-a-file-to-run-the-server-for-us)
    - [3.5: Setting Up Some Variables And Settings For Fly](#35-setting-up-some-variables-and-settings-for-fly)
      - [3.5.1 - Creating A Volume To Host The Database](#351---creating-a-volume-to-host-the-database)
      - [3.5.2 - Noting The Location Of That Volume With Fly](#352---noting-the-location-of-that-volume-with-fly)
      - [3.5.3: Setting Some Superuser Credentials](#353-setting-some-superuser-credentials)
      - [3.5.4: Setting That Debug Setting From Earlier](#354-setting-that-debug-setting-from-earlier)
      - [3.5.5: Setting The SECRET\_KEY](#355-setting-the-secret_key)
  - [Phase 4: Set Up A Github Action For Continuous Deployment](#phase-4-set-up-a-github-action-for-continuous-deployment)
    - [Continuous Integration??](#continuous-integration)
    - [Continuous Deployment??!?](#continuous-deployment)
    - [Step 1: Creating A FLY API token](#step-1-creating-a-fly-api-token)
    - [Step 2: Saving the token as a Github Secret](#step-2-saving-the-token-as-a-github-secret)
    - [Step 3: Check your .gitignore](#step-3-check-your-gitignore)
    - [Step 4: Create the action.](#step-4-create-the-action)
  - [Phase 5: Commit! Again!](#phase-5-commit-again)


This is it, people: the moment we've all been waiting for! In this lesson, we go from having a local codebase that we can use for testing, to having a globally accessible back end that can be accessed from any computer in the world.

We are going to walk through the process of setting our app up to run in the cloud.  In the process we will no doubt run into some fun bugs, but that's the fun of coding!

> "The Cloud" is such a buzzword... really what it means is "on someone else's machine somewhere, which is being provided to us to run our code and make it available to the world". *Technically* it is possible to host a website from your own laptop, but it's totally impractical/insecure, and you'd never be able to turn the laptop off again or the website would go down. In the process we will no doubt run into some fun bugs, but that's the whole fun of coding!

For some of the code content in these notes I'll be using the `diff` file format. This is a way of expressing changes to files. Lines that are to be deleted are prepended with a `-` (minus) symbol and coloured red. Lines that are to be added are prepended with a `+` (plus) and are coloured green. You don't need to include the minuses and plusses in your code; it's just the best way to display these changes in Markdown on Github. 

```diff
-delete this line
+add this line
```

## Phase 0: Commit!
Before we get started, let's make sure everything is backed up. You should stage your changes, make a commit, and push that commit. 

```Bash
git add .
```

```Bash
git commit -m "pre-deployment commit"
```

```Bash
git push origin main
```
Feel free to use github desktop if that's what you've been using until now. Also feel free to make a new branch to work on if you are so inclined.

## Phase 1: Installation
First we have some things we need to install. Make sure you have our virtual environmet activated in the terminal and then navigate to the root folder (the one containing your `requirements.txt` file):

### 1.1: Install `gunicorn`
That's "g-unicorn" not "gun-icorn". When I first read this I imagined a unicorn with a gun for a horn, but actually the G is short for "green". This library will be what actually serves the site for us, substituting in for the much less powerful server that is attached to the `runserver` command.
```Bash
python3 -m pip install gunicorn
```

### 1.2: Install `whitenoise`
This is a library that will handle serving static files for us, if we ever have any. Some info on it here: https://pypi.org/project/whitenoise/

```Bash
python3 -m pip install whitenoise
```

### 1.3: Install `CORS`
This is a security library to prevent people getting hacked on our site.
```Bash
python3 -m pip install django-cors-headers
```

### 1.4: Add Those Three New Packages to your `requirements.txt`
```Bash
python3 -m pip freeze > requirements.txt
```

### 1.5: Install Fly
This one is a little more complex - it's not a Python package, so we aren't installing it in our virtual environment. It's a terminal program that we are installing on our computers.

#### 1.5.a - On Windows:

```cmd
iwr https://fly.io/install.ps1 -useb | iex
```

#### 1.5.b - On Mac:
First make sure that `brew` is installed
```Bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh"
```
If this results in a successful installation you may be prompted to run two more commands - if so you should run those too!

Then use `brew` to install Fly:

```Bash
brew install flyctl
```

### 1.6: Sign Up To Fly
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

## Phase 2: Tweaking Settings
We are changing a few things. Let's talk through them before we modify the code.

### 2.1: The Secret Key
Django needs a value for the `SECRET_KEY` variable in order to run, but once we've launched, we don't want to use the value that is stored in our code - this is visible on Github for anyone to steal. We'll tweak our settings so that Django tries to find a value stored on the computer it's running on first, and only uses the hardcoded value if it doesn't find anything. 

### 2.2: The Debug Mode
While we are at it, we had also better make it so that Django doesn't give complex, developer-friendly error messages to the general public. Those tend to freak out the normies. We'll set it up so that it turns off the error messages by default, and only makes them available to us when we set a local variable for them.

Let's set this variable locally now before we forget. That way we will still see those useful errors when we are running the back end locally during development.

#### 2.2.a - On Windows:
```cmd
$env:DJANGO_DEBUG = "True"
```
> If that didn’t work, follow these steps instead: https://helpdeskgeek.com/how-to/create-custom-environment-variables-in-windows/ (up to
and excluding the “Explorer” section). Then restart powershell and make sure you reactivate your virtual environment.

#### 2.2.b - On Mac:
```Bash
export DJANGO_DEBUG=True
```

### 2.3: The Static Directory
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

### 2.4: The Middleware
We need to tell Django to use that `whitenoise` library we installed earlier, so we'll add it to our list of middleware.

### 2.5: The Database Location
We are going to need to point Django to a different database location on our deployed app than it uses for the local version. That's because we won't be pointing it to a database on our own machine; it'll be a database on someone else's machine, and that database will be in a different spot.

### 2.6: Some `CORS` Stuff
These are settings that prevent people from being tricked into taking actions on the site when malicious actors forward them there without their knowledge. It's security stuff!

### 2.7: Ok Let's Make Those Tweaks!

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

## Phase 3: Create a Fly Project
### 3.1: Launch!
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

### 3.2: Customising The `dockerfile`

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
+RUN chmod +x /code/run.sh

EXPOSE 8000

# replace demo.wsgi with <project_name>.wsgi
-CMD ["gunicorn", "--bind", ":8000", "--workers", "2", "demo.wsgi"]
+CMD ["/code/run.sh"]
```

### 3.3: Customizing `fly.toml`

1. First, check to see if a `procfile` has been automatically generated by Fly. If it has, please delete this file! It is unnecessary and may cause bugs.
2. Some Windows users have reported that the `.toml` file autogenerated by Fly has some variations, or does not exist. Rather than listing tweaks to make, the entire file is set out below. We recommend copying over everything **and modifying line 1 to include your app's unique name**: 

```toml
app = "YOUR-APP-NAME"
kill_signal = "SIGINT"
kill_timeout = 5
processes = []

[env]
  PORT = "8000"

[experimental]
  auto_rollback = true

[[services]]
  http_checks = []
  internal_port = 8000
  processes = ["app"]
  protocol = "tcp"
  script_checks = []
  [services.concurrency]
    hard_limit = 25
    soft_limit = 20
    type = "connections"

  [[services.ports]]
    force_https = true
    handlers = ["http"]
    port = 80

  [[services.ports]]
    handlers = ["tls", "http"]
    port = 443

  [[services.tcp_checks]]
    grace_period = "1s"
    interval = "15s"
    restart_limit = 0
    timeout = "2s"

[[statics]]
  guest_path = "/app/public"
  url_prefix = "/static/"

[mounts]
  source = "dbvol"
  destination = "/dbvol"
```

### 3.4: Creating A File To Run The Server For Us
When we run the server in production we won't use the `manage.py runserver` command - that's just for development.

Instead, we'll be using the `gunicorn` library that we installed earlier. To set it up so that the cloud machine we are deploying to automatically starts up `gunicorn` for us, we will write a file called `run.sh`. You can see we've referenced it already up there in our `dockerfile`.

Create the file next to your `manage.py` file, and make sure you call it `run.sh`. Here's what to put in it (the comment is actually important to include!):

```Bash
#!/usr/bin/env bash
python manage.py migrate
python manage.py createsuperuser --no-input
gunicorn --bind :8000 --workers 1 crowdfunding.wsgi
```

### 3.5: Setting Up Some Variables And Settings For Fly
#### 3.5.1 - Creating A Volume To Host The Database
> **You need to substitute in the funky name that Fly generated for your app here!** For instance, mine was `quiet-tree-5089`
```Bash
fly volumes create -a <app-name> -r syd --size 1 dbvol
```

#### 3.5.2 - Noting The Location Of That Volume With Fly
```Bash
flyctl secrets set DATABASE_DIR='/dbvol/db.sqlite3'
```
#### 3.5.3: Setting Some Superuser Credentials
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

#### 3.5.4: Setting That Debug Setting From Earlier
Technically, we've made it so that you don't need to set this in deployment, but it never hurts to be explicit.
```Bash
flyctl secrets set DJANGO_DEBUG=False
```

#### 3.5.5: Setting The SECRET_KEY
This bit is pretty important. We Need to pick a secure secret key, and also not write it down or back it up anywhere. Not even we need to know this - it's only for the deployed machine.

You can generate one here: https://djecrety.ir/

> **Make sure you substitute the value in here!**

```Bash
flyctl secrets set DJANGO_SECRET_KEY='<your-secret-key>'
```

## Phase 4: Set Up A Github Action For Continuous Deployment

Right now, your app is configured and ready to deploy. If we were doing things in a quick-and-dirty way, you could just run a deployment command, and the website would go live. However, there's a couple of problems with the quick-and-dirty method:

- The website that got deployed would be based on the current state of your local code, rather than the main branch of your Github remote. This is a Dodgy with a capital D. We only want committed, merged-to-main code to be deployed.
- The physical machine you deployed from (and the specific Fly login details that were active on it) would be tangled up into your deployment process. This is a problem if you have two computers, and it's a *much bigger problem* if you have multiple members of a team who all want to be able to update the deployment. What happens if the group member who did the deployment gets sick, and you want to add a new feature to the site? You'd have to wait on them to be available before you could redeploy!

Instead, we are going to use a technique called CI/CD, which stands for **Continuous Integration, Continuous Deployment**.

### Continuous Integration??

The great news is that if you've been following good Git technique up until now, you're already using the simplest form of CI. Large organisations include automated testing, branch protection, and pull request approval processes to make their CI more powerful, but at its most basic, Continuous Integration is just the practise of making frequent, small merges whenever you know that the change you just made is working.

### Continuous Deployment??!?

CD means having your remote repository set up so that when you merge code into the main branch on the remote repository, that code is automatically deployed. You can do this by merging locally and then pushing, or by pushing your feature branch to the remote and then making a formal pull request. The second option is best when you're working in a team.

Large companies do this so that they can make sure that every deployment happens from a uniform environment, and every person who needs to take code to deployment has equal access to that environment. Instead of executing deployment from your local machine, we'll set up our Github repo so that a Github-hosted cloud computer spins up every time we push code to main, and executes the deployment for us!

### Step 1: Creating A FLY API token
In your terminal, run the following command:
```bash
flyctl tokens create deploy -x 999999h
```

The terminal will spit out a long sequence of numbers and letters. Copy the whole thing (it may contain spaces) and store it in a text document for the moment.

### Step 2: Saving the token as a Github Secret
Navigate to the Github.com repo for your project. Go to `Settings` > `Secrets`, and select the option to create a repository secret.

Call your secret `FLY_API_TOKEN`, and paste the whole text of the token you got from step 1 in as its value.

### Step 3: Check your .gitignore
Make sure that it does **not** contain an entry for `dockerfile` or anything ending with `.toml`. We need these files to be pushed to Github.

### Step 4: Create the action.
- Create a folder next to your `requirements.txt` called `.github`. It needs to have that fullstop at the start of the name!
- Inside of that folder create another folder called `workflows`. Again, the exact name is important here.
- Inside THAT folder, create a file called `fly.yml`

Here are the contents you need in the `fly.yml` file:

```yml
name: Fly Deploy
on:
push:
    branches:
    - main
jobs:
deploy:
    name: Deploy app
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: superfly/flyctl-actions/setup-flyctl@master
    - run: flyctl deploy --remote-only
        env:
        FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```
Commit your changes, and merge them into main on the remote branch. 
- If you're working on a solo project, you can do this by just merging to main locally, and then pushing main to the remote. 
- If you're working in a group, you should push your *feature branch* to the remote, open a pull request to merge it into main, and then get someone to approve that pull request.

> Sometimes at this point you'll encounter an error saying that the method you're using to push to Github doesn't have permission to create a Github Action. If this happens to you, you can work with your mentors to increase the permission assigned to your authentication method.

Once these changes are in the main branch on Github, an action will run to deploy your site.

If everything ran successfully, go to `https://<your_project_name>.fly.dev/projects/` to see if your site is running! 

Did it work? Mine did. That thrill never gets old. I dunno about you but I feel like this:
![Galadriel looks at the camera with a powerful expression and shouts: "In place of a Dark Lord you would have a Queen! Not dark but beautiful and terrible as the Dawn! Treacherous as the Seas! Stronger than the foundations of the Earth! All shall love me and despair!"](img/a_queen.jpg)

If it didn't work, grab a mentor and let's get bugshooting!

If your site is up and running, try navigating to an endpoint that doesn't exist; you should get a plain boring error message, instead of the detailed ones we are used to.

If you're stuck, have a look at my working code here and see what's different: https://github.com/SheCodesAus/she-codes-crowdfunding-api-project-Hauteclere/tree/test_deploy

## Phase 5: Commit! Again!

For land's sake, if you got it working lock those changes in!
```Bash
git add .
```

```Bash
git commit -m "post-deployment commit"
```

```Bash
git push origin main
```