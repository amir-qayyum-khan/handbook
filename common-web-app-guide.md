# ODL Docker/Django/Webpack Web App Guide

**SECTIONS**
1. [Initial Setup](#initial-setup)
2. [Running and Accessing the App](#running-and-accessing-the-app)
3. [Troubleshooting](#troubleshooting)


# Initial Setup

### Major Dependencies
- Docker
  - _[OSX only]_ Recommended OSX install method: [Download from Docker website](https://docs.docker.com/mac/)
- docker-compose
  - Recommended install: pip (`pip install docker-compose`)
- _[OSX only]_
  - Virtualbox (https://www.virtualbox.org/wiki/Downloads)
  - Node/NPM 
    - Recommended install method: [Installer on Node website](https://nodejs.org/en/download/)
    - *NOTE: No specific versions have been chosen yet for NPM.*
  - Yarn
    - Some of our projects ship with a helpful yarn installation script: `sudo node ./scripts/install_yarn.js`
    - If the the install script doesn't exist, install by running: `npm install -g yarn@0.27.0` (or whatever version is specified for yarn in `package.json`; use `sudo` as necessary)

All commands in this guide should be run from the root directory of your project's repository (unless specified otherwise).

### _[OSX only]_ Create a Docker VM for this app

Replace `MACHINENAME` with a name that indicates the project (e.g.: `mm` or `micromasters` for the MicroMasters project). Allowed characters for the machine name: alphanumeric characters, periods, and dashes.

    docker-machine create --driver virtualbox MACHINENAME
    docker-machine start MACHINENAME
    # 'docker-machine env MACHINENAME' prints export commands for important environment variables
    eval "$(docker-machine env MACHINENAME)"

### Docker Container Setup

#### 1) Create your ``.env`` file
 
    cp .env.example .env

#### 2) Build the containers

    docker-compose build

*NOTE: You will also need to run this command whenever requirements files change (``requirements.txt``, ``test_requirements.txt``, etc.)*

#### 3) Create database tables from the Django models

    docker-compose run web ./manage.py migrate

*NOTE: You will also need to run this command whenever there are new migrations (i.e.: database models have been changed/added/removed).*

#### (Optional) Create a superuser
Some of our apps include user creation as part of their specific setup steps. If the given app does not
include steps to create a user, you can create one easily via Django's `createsuperuser` command.
It will prompt you for the username and some other details for this user.

    docker-compose run web ./manage.py createsuperuser


# Running and Accessing the App

#### 1) _[OSX only]_ Make sure the right environment variables are set before running the container or executing any commands: 

```
eval "$(docker-machine env MACHINENAME)"
```

#### 2) Run the containers

Start all the services that are required to run the app:

    docker-compose up
    
#### 3) _[OSX only]_ Set up and run the webpack dev server on your host machine

In a separate terminal tab, use the webpack helper script to install npm modules and run the dev server:

    ./webpack_dev_server.sh --install

*NOTE: The ``--install`` flag is only needed if you're starting up for the first time, or if updates have been made to the packages in ``./package.json``.*

#### 4) Navigate to the running app in your browser

Your app should now be accessible via browser:

1. The URL of the locally-running site needs to include the port number that the `nginx` service is running on.
    - One-line command to figure out that port number: `docker-compose ps nginx | perl -nle '/[0-9\.]*:(\d+)/ && print "$1";'`
    - This port number is specified for the `nginx` service in `docker-compose.yml`
1. _[Linux only]:_ Navigate to `localhost:PORT` (e.g.: `localhost:8079`)
1. _[OSX only]:_ Run ``docker-machine ip`` to see the IP that has been assigned for your docker VM, then navigate to `YOUR_MACHINE_IP:PORT` (e.g.: `192.168.99.100:8079`)

# Testing

There are a few different commands for running tests/linters:

```bash
# In most of our projects we include a shell script that runs 
# the entire test suite.
./test_suite.sh

### PYTHON TESTS/LINTING
# Run Python tests
docker-compose run web tox
# Run Python tests in a single file
docker-compose run web tox /path/to/test.py
# Run Python test cases in a single file that match some function/class name
docker-compose run web tox /path/to/test.py -- -k test_some_logic
# Run Python linter
docker-compose run web pylint

### JS/CSS TESTS/LINTING
# We also include a helper script to execute JS tests in most of our projects 
# (this file may exist at the project root or in ./scripts/test)
./js_test.sh
# Run JS tests in specific file
./js_test.sh path/to/file.js
# Run JS tests in specific file with a description that matches some text
./js_test.sh path/to/file.js "should test basic arithmetic"
### OSX USERS:
### Run the commands below WITHOUT `docker-compose run watch`
# Run the JS linter
docker-compose run watch npm run lint
# Run SCSS linter
docker-compose run watch npm run scss_lint
# Run prettier-eslint, fixes style issues
docker-compose run watch npm run fmt
```

# Running Commands

You can run a Django shell with the following command:

    docker-compose run web ./manage.py shell

# Troubleshooting

#### `node-sass` error when running the webpack_dev_server.sh script or `yarn install`

Try running `npm rebuild node-sass`, then running the previous command again.

#### Error message indicating that the `auth_user` table doesn't exist

Try running `docker-compose run web ./manage.py migrate auth`, then run `docker-compose run web ./manage.py migrate`.

#### _[OSX only]_ `docker-compose up` or `docker-machine start MACHINENAME` hanging on "Waiting for an IP..." or "Waiting for SSH to be available..."

These errors can often be solved by powering off the VM in VirtualBox and trying again (docker-machine will automatically restart the VM). Open the VirtualBox app, right-click on the machine in the left column, and click Close > Power Off. If Close > Power Off is not available, the error may be related to the saved machine state. Try clicking 'Discard Saved State'. After either powering off the machine or discarding the saved state, run `docker-machine restart MACHINENAME`.

#### _[OSX only]_ "Error checking TLS connection ...  You can attempt to regenerate them using `docker-machine regenerate-certs [name]`."

This sometimes happens after the machine has been sleeping. The best known way to address it is to run the suggested command - `docker-machine regenerate-certs MACHINENAME` - and confirm with 'y' when prompted. This has the unfortunate effect of changing the IP address of the machine, so you'll need to navigate to a different URL in your browser to see the running site. For example, your machine URL might have been `http://192.168.99.100:8079/`. After running `regenerate-certs`, the IP might change to `http://192.168.99.101:8079/`. To check the IP of your machine, run `docker-machine ip MACHINENAME`.

#### _[OSX only]_ Error indicating that Docker for Mac is not running (even though it is running)

Open a new terminal tab/window, navigate to the same directory, and re-run the same command (e.g.: `docker-compose up`). For whatever reason, a terminal window can sometimes fail to recognize that Docker for Mac is running, and a fresh terminal window will fix that.

#### _[OSX only]_ Everything is broken and I need a fresh start

If your machine state seems absolutely beyond repair, you can run `docker-machine rm -f MACHINENAME` to destroy the VM and start over with the normal [initial setup steps](#initial-setup).
