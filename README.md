# AWS Django Guide 2021

This is a short guide on how to deploy a Django application on [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) and connect the database to an [Amazon RDS](https://aws.amazon.com/rds/) instance.

*Date: June 7th, 2021*

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
6. Log into [AWS Management Console](https://aws.amazon.com/console/) then go to **Services>Elastic Beanstalk**. 🔴
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
13. Deploy the project
```
C:\your-django-project\> eb deploy
```
14. (Optional) Open the website
```
C:\your-django-project\> eb open
```

## Creat an Amazon RDS instance and connect it to your Django application


## Set static files, make migrations, and create a superuser on AWS

Once you deploy your Django project on AWS, you can check the admin portal `admin/` and likely find out that CSS doesn't work properly. 
