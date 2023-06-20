Prerequisites
=============

This guide is intended for macOS users only, since Superset is only supported on Linux and macOS .

Please install the following before installing and configuring Superset

1.  Microsoft Visual Code [https://code.visualstudio.com](https://code.visualstudio.com)
2.  brew [https://brew.sh](https://brew.sh)
3.  some database tool i.e. [https://tableplus.com](https://tableplus.com)
4.  Latest Xcode Command Line Tools
    
    xcode-select --install
    
      
    

Installing Python Environment
=============================

The standard Python version installed by macOS is not recommended. Please install the newest and compatible version ( >= **3.8**) via

brew install python

We also need some more packages, inkl. **MySQL**.

brew install readline pkg-config libffi openssl mysql mysqlclient

and the newest versions of **pip** and **setuptools**

pip install --upgrade setuptools pip

And the final step for Python is to install the **Python Virtual Environment**, since Superset will run in it.

pip install virtualenv

  

Installing Superset
===================

We are starting with the most recent stable version of Superset. As of writing, this is version 1.5.1.

Getting the Sourcecode from GitHub

git clone git@gitlab.bfops.io:holodeck/tracking/superset.git

If not already in the correct directory (please check with '_pwd_'), change into it. The directory should be ./**superset**, not **./superset/superset**.

Now we install and activate a Virtual Environment for Superset.

python3 -m venv venv
. venv/bin/activate

If you ever want to deactivate the virtual environment or exit it, just type `deactivate`. Please just DO NOT DO THAT NOW!

We might need to **update pip** and install **flask.**

pip install --upgrade pip
pip install flask

install some Python packages for **MySQL** and **Image** Library

pip install PyMySQL Pillow

**For Superset 2.0, please install the following**

pip install flask==2.0.3.
pip install werkzeug==2.0.3
pip install jinja2==3.0.1

Unfortunatly some dependencies are not correct, so we need to correct the version for **markupsave**

pip install markupsafe==2.0.1

If everything went smoothly, you should be able to run the superset command.

(venv) ➜  superset git:(48f3eb427) superset

  This is a management script for the Superset application.
Options:
  --version  Show the flask version
  --help     Show this message and exit.
Commands:
  alert                     Run the alert scheduler loop
  compute-thumbnails        Compute thumbnails
  db                        Perform database migrations.
  export-dashboards         Export dashboards to JSON
  export-datasource-schema  Export datasource YAML schema to stdout
  export-datasources        Export datasources to YAML
  fab                       FAB flask group commands
  flower                    Runs a Celery Flower web server Celery Flower
                            is...
  import-dashboards         Import dashboards from JSON file
  import-datasources        Import datasources from YAML
  import-directory          Imports configs from a given directory
  init                      Inits the Superset application
  load-examples             Loads a set of Slices and Dashboards and a...
  load-test-users           Loads admin, alpha, and gamma user for testing...
  re-encrypt-secrets
  refresh-druid             Refresh druid datasources
  routes                    Show the routes for the app.
  run                       Run a development server.
  set-database-uri          Updates a database connection URI
  shell                     Run a shell in the app context.
  superset                  This is a management script for the Superset...
  sync-tags                 Rebuilds special tags (owner, type, favorited...
  update-api-docs           Regenerate the openapi.json file in docs
  update-datasources-cache  Refresh sqllab datasources cache
  version                   Prints the current version number
  worker                    Starts a Superset worker for async SQL query...

Unfortunately we have to post-install some npm packages in the **superset/superset-frontend** directory

cd superset-frontend
npm install
npm run build-dev
cd ..

Database/MySQL Setup
====================

If you have not configured your mysql service on your machine, please do so. Startup the mysql engine as a **service**

brew services start mysql

and set up your **security**

mysql\_secure\_installation

After you have successfully done that, or if you already have a functioning MySQL service on your machine, add a **superset database** to the server. Either via a database tool or interactive command line.

**This is the database/schema that holds all superset metadata.**

CREATE DATABASE superset;

Add a **superset** user via your database tools or command line

CREATE USER 'superset'@'localhost' IDENTIFIED BY 'superset';

and grant that user **all necessary privileges**

GRANT ALL PRIVILEGES ON superset.\* TO 'superset'@'localhost';

Development Environment
=======================

Please open the **Superset** Sourcecode in **VSC** via

code .

if this throws an error, please refer to [https://code.visualstudio.com/docs/setup/mac](https://code.visualstudio.com/docs/setup/mac) for setup of the command line tool.

Superset Database configuration
-------------------------------

We now need to tell the **superset app** where its **meta database** resides. In VSCode, please create a new file **superset\_config.py** in the superset root directory and add the following lines to it

SQLALCHEMY\_DATABASE\_URI = 'mysql+pymysql://superset:superset@localhost/superset'
FEATURE\_FLAGS = {
    'UX\_BETA': True,
    'ENABLE\_EXPLORE\_DRAG\_AND\_DROP' : True,
    'ENABLE\_DND\_WITH\_CLICK\_UX': True,
    'DASHBOARD\_RBAC': True,
    'DASHBOARD\_NATIVE\_FILTERS\_SET': True,
    'DASHBOARD\_NATIVE\_FILTERS': True,
    'DASHBOARD\_CROSS\_FILTERS': True,
}

and finally **build** the local superset code. Please make sure that you have not changed the directory, that you are in your superset root directory. This is important. Oh, and this could take a minute or two.

pip install .

Finishing the Superset Setup
----------------------------

After we setup und configured the mysql database and let superset know where to find it, we can now finish the rest of the installation. This only has to be done once and can take a while.

In the Terminal where you last typed

**`code .`**

we now need to **initialize** the database

superset db upgrade

The next step will create the **admin user** for the superset UI

export FLASK\_APP=superset
superset fab create-admin

If you wish, you can load some **example data** into the database.

superset load\_examples

And finally we need to create the standard **roles and permissions**

superset init

Lauch/Debug Configuration
-------------------------

We need to add a new Debug Configuration

{
      "name": "Superset Backend",
      "type": "python",
      "request": "launch",
      "module": "flask",
      "env": {
        "FLASK\_APP": "superset/app.py",
        "FLASK\_ENV": "development"
      },
      "args": \[
        "run",
        "--no-debugger",
        "-p 8088",
        "--with-threads",
        "--reload",
        "--debugger"
      \],
      "jinja": true,
      "justMyCode": true
    }

  

  

To build the code via **webpack** and copy it into the `superset/static/asset` directory run

npm run build-dev

to have your changes been build and copied "on-the-fly", run

npm run dev

and to have a lightweight dev server, which proxies the superset web ui on port 9000 instead of 8088, run

npm run dev-server

With this option, the compiles will not be copied into the assets directory, but resides just in the proxy server.