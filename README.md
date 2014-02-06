Deploying Helios to CloudFoundry
=========================

# Introduction

So, you've decided to play with Helios and deploy your app to CloudFoundry. Good for you.
Let's grab some toys and:
*  Install & Setup PostgreSQL
*  ... and Helios
*  ... and CloudFoundry console

But, just before we start, let's check our Ruby version. In your terminal:
```
ruby -v
# ruby 1.9.3p484 (2013-11-22 revision 43786) [x86_64-linux]
```

As you can see, I'm using Ruby 1.9.3 (and newer versions should do just fine). Also, I prefer to use RVM and gemsets to play around.
Let's create an empty gemset.
```
rvm gemset use 1.9.3-p484@cf_helios --create
# Using 1.9.3-p484@cf_helios with gemset cf_helios
# ruby-1.9.3-p484 - #gemset created /home/alexander/.rvm/gems/ruby-1.9.3-p484@cf_helios
# ruby-1.9.3-p484 - #generating cf_helios wrappers
```

Now we're ready to have some fun. Let's install PostgreSQL.

# Installing PostgreSQL
PostgreSQL is one of Helios external dependencies. Helios uses PostgreSQL hstore extension, so we're going to install it too.
On my Ubuntu 12.04 LTS I did it by this:
```
sudo apt-get install postgresql-9.1 posgresql-contrib-9.1
```

Also, you're going to need a user and a proper database
```
psql postgres
CREATE DATABASE cf_helios_db;
CREATE USER cf_helios_user WITH PASSWORD 'cf_helios_password';
GRANT ALL PRIVILEGES ON DATABASE cf_helios_db TO cf_helios_user;
\c cf_helios_db
CREATE EXTENSION hstore;
\q
```
Now, add the following line to ```/etc/postgresql/9.1/main/postgresql.conf```:
```
listen_addresses = '*'
```
And this line to ```/etc/postgresql/9.1/main/pg_hba.conf```:
```
host all all 0.0.0.0/0 md5
```
This will allow PostgreSQL to receive remote connections. Note, that depending on your OS, these files might be in a different place.
Restart PostgreSQL for changes to take effect:
```
sudo service postgresql restart
```

# Installing Helios
Let's install Helios itself
```
gem install helios
# Fetching: highline-1.6.20.gem (100%)
# Successfully installed highline-1.6.20...
# ...
# Installing ri documentation for venice-0.2.0
# Installing ri documentation for zurb-foundation-4.1.2
# 52 gems installed
```
To create an application use the following code:
```
helios new cf_helios_app
#         create  Procfile
#         create  Gemfile
# ...
# Initialized empty Git repository in /home/alexander/cf_helios_app/.git/
# [master (root-commit) ad7e12a] Initial Commit
# 6 files changes, 200 insertions(+) ...
```
It will create ```ch_helios_app``` folder in your current directory.
Let's change ```cf_helios_app/.env``` file to look like this:
````
DATABASE_URL=postgres://cf_helios_user:cf_helios_password@localhost/cf_helios_db
```
Now you can start your app with ```helios start``` inside of ```cf_helios_app``` directory.
Go on, open ```http://localhost:5000/admin/``` in your browser, to see if it works.


# Installing CloudFoundry console

CloudFoundry console installation is simple as it can be:
```
gem install cf
# Fetching: addressable-2.3.5.gem (100%)
# Successfully installed addressable-2.3.5
# Fetching: multi_json-1.8.4.gem (100%)...
# ...
# Installing ri documentation for uuidtools-2.1.4
# 23 gems installed
```

# Setting installed CloudFoundry as a target
cf target http://api.cloudfoundry.yourcfdomain.com
# Setting target to http://api.cloudfoundry.yourcfdomain.com... OK

# Login to CloudFoundry. You will be definetly promted for Organization, Space and etc.
cf login
# target: http://api.cloudfoundry.altoros.com

# Email> admin

# Password> ********

# Authenticating... OK
# 1: altoros
# 2: practice-fusion
# 3: system_domain
# Organization> 1

# Switching to organization altoros... OK
# 1: dev
# 2: haspira
# 3: jenkins
# 4: test
# Space> 1

# Switching to space dev... OK


## CREATING A HELIOS APPLICATION

helios new cf_helios_app
#         create  Procfile
#         create  Gemfile
# ...
# Initialized empty Git repository in /home/alexander/cf_helios_app/.git/
# [master (root-commit) ad7e12a] Initial Commit
# 6 files changes, 200 insertions(+) ...

# You need to change cf_helios_app/.env file to look like thins
DATABASE_URL=postgres://cf_helios_user:cf_helios_password@localhost/cf_helios_db

# Now you can start it with the following command
helios start

# You can check it by opening http://localhost:5000/admin/

## DEPLOYING HELIOS TO CLOUD FOUNDRY










