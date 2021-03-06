Deploying Helios and LoopBack to CloudFoundry
=========================

# Introduction

So, you've decided to play with Helios and LoopBack. And even more: you want to deploy your app to CloudFoundry.
Lucky for you, I've got just the ticket.

# Helios
Let's grab some toys and:
*  Install & setup PostgreSQL
*  ... and Helios
*  ... and CloudFoundry Command Line Interface

But, just before we start, let's check our Ruby version. In your terminal:
```
ruby -v
# ruby 1.9.3p484 (2013-11-22 revision 43786) [x86_64-linux]
```

As you can see, I'm using Ruby 1.9.3 (and newer versions should do just fine). Also, I prefer to use RVM and gemsets to play around.
```
rvm gemset use 1.9.3-p484@cf_helios --create --default
# Using 1.9.3-p484@cf_helios with gemset cf_helios
# ruby-1.9.3-p484 - #gemset created /home/alexander/.rvm/gems/ruby-1.9.3-p484@cf_helios
# ruby-1.9.3-p484 - #generating cf_helios wrappers
```

Now we're ready to have some fun.

## Installing PostgreSQL
PostgreSQL is one of Helios external dependencies. Helios uses PostgreSQL hstore extension, so we're going to install it too. There's another thing though.
Since we're going to deploy our app to CloudFoundry we need to provide PostgreSQL service. There is a couple of ways to do this.
If your existing CloudFoundry installation already has PostgreSQL service (or way to create one), you can just bind it to your app.
However, I had no other way but to provide my own PostgreSQL service. Let's begin with local setup. 

Simplest way to do it on Ubuntu 12.04 LTS:
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

If you're willing to provide PostgreSQL service from your machine, you can simply continue with instructions below.
Otherwise, you need to install PostgreSQL on a machine which can be accessed by your app from CloudFoundry (in my case - it was a machine with a public IP).
These are additional steps required for PostgreSQL to receive remote connections:
Add the following line to ```/etc/postgresql/9.1/main/postgresql.conf```:
```
listen_addresses = '*'
```

And this line to ```/etc/postgresql/9.1/main/pg_hba.conf```:
```
host all all 0.0.0.0/0 md5
```

Note, that depending on operating system, these files might be in a different place. Also, don't forget about firewalls.
PostgreSQL is running on 5432 port by default.
Restart PostgreSQL for changes to take effect:
```
sudo service postgresql restart
```

## Installing Helios

Let's install Helios itself:
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

## Installing CloudFoundry CLI

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

Now we need to connect to our CloudFoundry installation. Console will ask you to select Organiation and Space.
Just follow the steps below:
```
cf target http://api.cloudfoundry.yourcfdomain.com
# Setting target to http://api.cloudfoundry.yourcfdomain.com... OK

cf login
# target: http://api.cloudfoundry.yourcfdomain.com

# Email> your_cf_login

# Password> ********

# Authenticating... OK
# 1: your_organization
# Organization> 1

# Switching to organization your_organization... OK
# 1: dev
# Space> 1

# Switching to space dev... OK
```

We're almost ready to deploy our application to CloudFoundry.

## Just before deployment

As I said before, in order to deploy our app, we need to provide it PostgreSQL service. Here's how to create
so called 'user provided' service in CloudFoundry. In your terminal:
```
cf create-service
# 1: mongodb , via 
# 2: user-provided , via 
# What kind?> 2

# Name?> postgresql-d15b3   

# What credential parameters should applications use to connect to this service instance?
# (e.g. hostname, port, password)> uri

# uri> postgres://cf_helios_user:cf_helios_password@your_postgresql_host/cf_helios_db

# Creating service postgresql-d15b3... OK
```

Let's create ```.cfignore``` file inside of ```cf_helios_app``` directory and fill it with these lines:
```
.sass-cache
.env
```
That's it!

## Deploying to CloudFoundry

After all we've done in this guide, it's going to be pretty easy. Just follow the directions of CloudFoundry console.
Inside of ```cf_helios_app``` directory:
```
cf push
# Name> helios

# Instances> 1

# 1: 128M
# 2: 256M
# 3: 512M
# 4: 1G
# Memory Limit> 3   

# Creating helios... OK

# 1: helios
# 2: none
# Subdomain> 1     

# 1: cloudfoundry.yourcfdomain.com
# 2: none
# Domain> 1                       

# Binding helios.cloudfoundry.yourcfdomain.com to helios... OK

# Create services for application?> n

# Bind other services to application?> y

# ...
# 7: postgresql-d15b3
# Which service?> 7

# Binding postgresql-d15b3 to helios... OK
# Bind another service?> n

# Save configuration?> y

# Saving to manifest.yml... OK
# Uploading helios... OK
# Preparing to start helios... OK
# -----> Downloaded app package (8.0K)
# -----> Using Ruby version: ruby-1.9.3
# -----> Installing dependencies using Bundler version 1.3.2
#        Running: bundle install --without development:test...
#        Fetching gem metadata from https://rubygems.org/........
#        Fetching gem metadata from https://rubygems.org/..
#        Installing i18n (0.6.9)
# ...
# Checking status of app 'helios'...
#   0 of 1 instances running (1 starting)
#   0 of 1 instances running (1 starting)
#   0 of 1 instances running (1 starting)
#   0 of 1 instances running (1 starting)
#   0 of 1 instances running (1 starting)
#   1 of 1 instances running (1 running)
# Push successful! App 'helios' available at helios.cloudfoundry.yourcfdomain.com
```

Now you can access your Helios app by http://helios.cloudfoundry.yourcfdomain.com/admin. Note, that
CloudFoundry replaced application port (from 5000 to 80).

Haven't tired yet? We still have LoopBack (StrongLoop) to play with. I hope, he doesn't mind.

# LoopBack
There're some prerequisites. And again, this is going to be much, much easier. Here's all we need to do:
*  Install Node.js
*  ... and Node Package Manager
*  ... and StrongLoop CLI


## Installing Node.js and npm

Node.js and npm in Ubuntu repo are pretty old. And as you've seen before - I like to play with version managers.
Let's install NVM:
```
curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```

Reopen your terminal and type this:
```
nvm install 0.10
nvm alias default 0.10
```

## Installing StrongLoop CLI
You would never believe how I did it:
```
npm install -g strong-cli
```
And we're ready to create our LoopBack app.
Here's how:
```
slc lb project cf_loopback_app
# ...
# Create a model in your app:
#   $ cd cf_loopback_app
#   $ slc lb model product -i
# Run your Project:
#   $ slc run .

```
It will create ```cf_loopback_app``` folder in your current directory.
We can create a simple model to play with. Inside of ```cf_loopback_app``` do the following:
```
slc lb model company
# Created company model.
```

Check out ```http://localhost:3000/explorer/``` for a bunch of new toys. 
Only one line of code left before...

## ...Deploying to CloudFoundry (again)
Almost like
> One ring to rule them all, one ring to find them,  
> One ring to bring them all and in the darkness bind them.  

```
cf push
# Name> loopback

# Instances> 1

# 1: 128M
# 2: 256M
# 3: 512M
# 4: 1G
# Memory Limit> 3   

# Creating loopback... OK

# 1: loopback
# 2: none
# Subdomain> 1       

# 1: cloudfoundry.yourcfdomain.com
# 2: none
# Domain> 1                       

# Creating route loopback.cloudfoundry.yourcfdomain.com... OK
# Binding loopback.cloudfoundry.yourcfdomain.com to loopback... OK

# Create services for application?> n

# Bind other services to application?> n

# Save configuration?> y

# Saving to manifest.yml... OK
# Uploading loopback... OK
# Preparing to start loopback... OK
# -----> Downloaded app package (8.7M)
# ...
# -----> Installing dependencies
#        npm WARN package.json cf_loopback_app@0.0.0 No description
#        npm WARN package.json cf_loopback_app@0.0.0 No repository field.
#        npm WARN package.json cf_loopback_app@0.0.0 No README data
# -----> Caching node_modules directory for future builds
# -----> Cleaning up node-gyp and npm artifacts
# -----> No Procfile found; Adding npm start to new Procfile
# -----> Building runtime environment
# -----> Uploading droplet (14M)
# Checking status of app 'loopback'...
#   0 of 1 instances running (1 starting)
#   1 of 1 instances running (1 running)
# Push successful! App 'loopback' available at loopback.cloudfoundry.yourcfdomain.com
```

Don't forget that CloudFoundry has its own preferences about ports:
```
http://loopback.cloudfoundry.yourcfdomain.com/explorer/
```

# Useful links

In case of trouble or unbearable interest, here's a list of some useful links on the topic.
*  [CloudFoundry CLI reference][2]
*  [Key facts application deploying application to CloudFoundry][3]
*  [Helios Github page][1]
*  [PostgreSQL documentation: pg_hba.conf][4]
*  [PostgreSQL documentation: Connections and Authentication][5]
*  [NVM Github page][6]
*  [Loopback Github page][7]


[1]: https://github.com/helios-framework/helios
[2]: http://docs.cloudfoundry.com/docs/using/managing-apps/cf/
[3]: http://docs.cloudfoundry.com/docs/using/deploying-apps/
[4]: http://www.postgresql.org/docs/9.1/static/auth-pg-hba-conf.html
[5]: http://www.postgresql.org/docs/9.1/static/runtime-config-connection.html
[6]: https://github.com/creationix/nvm
[7]: https://github.com/strongloop/loopback