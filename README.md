# AWS Django Guide 2021

This is a short guide on how to deploy a Django application on [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) and connect the database to an [Amazon RDS](https://aws.amazon.com/rds/) instance.

*Date: June, 2021*

This guide includes

1. Creating and deploying a Django application on AWS Elastic Beanstalk
2. Adding an Amazon RDS instance to the Django application
3. Making migrations and creating a superuser on AWS

Most steps are taken from [Deploying a Django application to Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-django.html), [Adding an Amazon RDS DB instance to your Python application environment](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-rds.html) for the first and second parts, and many stackoverflow posts for the third part.

Steps that are **different** from the official AWS guides are marked with 🔴.



## Prerequisites

* Python 3.8 or later
* pip
* virtualenv
* awsebcli

## Create a Django project

1. Create a virtual environment `your-virt-name`
```
C:\> virtualenv %HOMEPATH%\eb-virt
```
2. Activate the virtual environment
```
C:\> %HOMEPATH%\your-virt-name\Scripts\activate
(your-virt-name) C:\>
```
3. Install Django 3.1 🔴
```
(your-virt-name) C:\> pip install django==3.1
```
4. Creat a Django project
```
(your-virt-name) C:\> django-admin startproject your-django-project
```
5. (Optional) Run your Django site locally to make sure it works
```
(your-virt-name) C:\> cd your-django-project
(your-virt-name) C:\your-django-project\> python manage.py runserver
```

## Deploy your Django project to AWS Elastic Beanstalk

1. Activate your virtual environment.
```
C:\your-django-project\>%HOMEPATH%\your-virt-name\Scripts\activate
```
2. Save required packages to `requirements.txt`
```
(your-virt-name) C:\your-django-project\> pip freeze > requirements.txt
```
> Make sure your Django version is [compatible](https://docs.djangoproject.com/en/3.2/faq/install/#what-python-version-can-i-use-with-django) with Python 3.8. Note that even Django 3.2 is supported by Python 3.8, it might **not** work on AWS. 🔴
3. Create a directory named `.ebextensions`
```
(your-virt-name) C:\your-django-project\> mkdir .ebextensions
```
4. In `.ebextensions\`, add a file named `django.config` with the following content
```
option_settings:
  aws:elasticbeanstalk:container:python:
    WSGIPath: your-django-project.wsgi:application
```
5. (Optional) Deactivate your virtual environment
```
(your-virt-name) C:\your-django-project\> deactivate
```
6. Log into [AWS Management Console](https://aws.amazon.com/console/) then go to **Services > Elastic Beanstalk**. 🔴
> I found it much easier to create an EB application through AWS console. If you wish to use `eb` commands, try `eb init -p "Python 3.8 running on 64bit Amazon Linux 2" your-app-name` then `eb create your-env-name`
7. Go to **Environments**, then **Create a new environment** 
8. Choose **Web server environment**, enter your Application/Environment/Domain names. Choose platform **Python/Python 3.8 running on 64bit Amazon Linux 2/Recommended version**
> It might take a while to for AWS to initialize your environment, make sure it is completed before you continue next step
9. Choose the application you just created
```
C:\your-django-project\> eb init
```
> If you don't see your application, double-check you chose the right **region**.
10. Set your environment as the default environment
```
C:\your-django-project\> eb use your-env-name
```
11. Check the status and find the domain name (you probably already have it if you created it though AWS management console)
```
C:\your-django-project\> eb status
Environment details for: your-env-name
  Application name: your-app-name
  ...
  CNAME: <your-domain-name>
```
12. Open `settings.py` in the directory `your-django-project\` and add `<your-domain-name>` to `ALLOWED_HOSTS`
```python
ALLOWED_HOSTS = ["<your-domain-name>"]
```
> If you have `.gitignore` in your directory, then `eb deploy` you should add a (blank) file named `.ebignore`
13. Deploy the project
```
C:\your-django-project\> eb deploy
```
> If you have `.gitignore` in your directory, then `eb deploy` will not upload files in there. You can add a file named `.ebignore` with files that you don't want to upload to AWS EB. 🔴
14. (Optional) Open the website
```
C:\your-django-project\> eb open
```

## Creat an Amazon RDS instance and connect it to your Django application

1. Open the [Elastic Beanstalk console](https://console.aws.amazon.com/elasticbeanstalk), go to **Environments** then choose the enviroment you have created
2. Go to **Configuration > Database > Edit**
3. Choose a DB engine, and enter a user name and password. Then click **Apply**

You may choose any egine, here I am using MySQL as an example.

4. Activate the virtual environment and install `mysqlclient`
```
C:\your-django-project\>%HOMEPATH%\your-virt-name\Scripts\activate
(your-virt-name) C:\your-django-project\> pip install mysqlclient
```
5. Update your `requirements.txt`
```
(your-virt-name) C:\your-django-project\> pip freeze > requirements.txt
```
6. (Optional) Remove the version of `mysqlclient` from `requirements.txt` 🔴
```
...
mysqlclient
...
```
> If you encounter any errors later, I highly recommend you to try this.
7. In directory `.ebextensions\`, create a file `packages.config` with the following content 🔴
```
packages:
  yum:
    python3-devel: []
    mariadb-devel: []
```
8. In `settings.py`, replace the Database section with the following (Make sure you have `import os`)
```python3
if 'RDS_HOSTNAME' in os.environ:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': os.environ['RDS_DB_NAME'],
            'USER': os.environ['RDS_USERNAME'],
            'PASSWORD': os.environ['RDS_PASSWORD'],
            'HOST': os.environ['RDS_HOSTNAME'],
            'PORT': os.environ['RDS_PORT'],
        }
    }
```
> You may wish to keep the default SQLite setting so you can test on your local host
9. Deploy the project, check everything works
```
C:\your-django-project\> eb deploy
```

## Set static files, make migrations, and create a superuser on AWS

For simplicity, I will do everything at once. (There might be some redundant steps.)

1. If you haven't done so, create an app for your django project
```
C:\your-django-project\>%HOMEPATH%\your-virt-name\Scripts\activate
(your-virt-name) C:\your-django-project\> python manage.py startapp your-app-name
```
2. Add your app to `settings.py`
```python3
INSTALLED_APPS = [
    ...
    'your-app-name.apps.YourAppNameConfig',
]
```
3. Make migrations
```
(your-virt-name) C:\your-django-project\> python manage.py makemigrations your-app-name
(your-virt-name) C:\your-django-project\> python manage.py migrate
```
4. In `your-app-name/`, make a new directory `management` and add a (blank) file named `__init__.py` inside
5. In `management/`, make a new directory `commands` and add a (blank) file named `__init__.py` inside
6. In `commands/`, add a new file named `create_my_superuser.py` with the following content
```python3
import os
from django.core.management.base import BaseCommand
from django.contrib.auth.models import User

class Command(BaseCommand):
    def handle(self, *args, **options):
        if not User.objects.filter(username='<your-superuser-name>').exists():
            User.objects.create_superuser('<your-superuser-name>',
                                          '<your-email-adress>',
                                          '<your-password>')
```
7. In `.ebextensions/`, add a new file named `django-migrate.config` with the following content
```
container_commands:
  01_migrate:
     command: "source $PYTHONPATH/activate && python manage.py migrate --noinput"
     leader_only: true
  02_collectstatic:
    command: "source $PYTHONPATH/activate && python manage.py collectstatic --noinput"
  03_create_superuser_for_django_admin:
    command: "source $PYTHONPATH/activate && python manage.py create_my_superuser"
    leader_only: true
option_settings:
  aws:elasticbeanstalk:container:python:
    WSGIPath: appname.wsgi:application
  aws:elasticbeanstalk:environment:proxy:staticfiles:
    /static: static
```

Now, you should've the following directories and files
```
your-django-project/
├── .ebextensions/
│   ├── django-migrate.config
│   ├── django.config
│   └── packages.config
├── your-app-name/
│   └── management/
│       ├── command/
│       │   ├── __init__.py
│       │   └── create_my_superuser.py
│       └── __init__.py
├── your-django-project/
│   └── settings.py
├── manage.py
└── requirements.txt
```
8. Add the following to `settings.py`
```python3
STATIC_URL = '/static/'
STATIC_ROOT = 'static'
```
9. Collect static files then deploy 
```
(your-virt-name) C:\your-django-project\> python manage.py collectstatic 
(your-virt-name) C:\your-django-project\> eb deploy
```
> You should to the admin dashboard `admin/` and log in to make sure everything works. If CSS doesn't work, then something is wrong with the static files.
