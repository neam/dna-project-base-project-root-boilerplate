_PROJECT_
========

## Local development

### Shortcut script

Here is a script that will ensure docker-machine is up, start the stack, the db and initiate angular frontend local development (grunt etc):

    stack/start-local-dev.sh

### Full procedure

Ensure docker is running (see _PROJECT_-product/docs/21-local-dev-first-time-set-up.md)

    docker-machine start default
    eval "$(docker-machine env default)"

First, open up a terminal window and step into _PROJECT_-product:

    cd _PROJECT_-product

### Backend

#### Local development

Start a local "_PROJECT_-product" server:

    stack/start.sh

First time a DATA profile is used locally, it's database needs to be created:

    export DATA=example
    bin/ensure-db.sh

Now you can open up a browser displaying the cms app included in the project:

    stack/open-browser.sh /

See [docs/13-overview-urls.md](./docs/13-overview-urls.md) for more examples.

#### Build & Deploy

Follow the instructions in `_PROJECT_-product/docs/52-deploy-tutum.md`

### Frontend

Instructions necessary to work actively with the frontend ui.

#### First-time only

Install s3cmd:

    brew install s3cmd

Make sure to have a .deploy-secrets file:

    cp ui/angular-frontend/.deploy-secrets.dist ui/angular-frontend/.deploy-secrets

Populate it with your deploy credentials before continuing.

#### Install deps + build

This needs to be done on first-time setup and sometimes when the campaign manager dependencies have been updated (i.e. after a git pull):

    bin/angular-frontend-full-build.sh

#### Local development

To work actively with the frontend ui, fire up grunt in angular-frontend (Note: will give error msg if run in more than one terminal window):

    bin/angular-frontend-develop.sh

#### Build

This needs to be done before deployment:

    bin/angular-frontend-build.sh

#### Deploy

To production:

    bin/angular-frontend-deploy.sh live

To stage:

    bin/angular-frontend-deploy.sh develop

## Advanced

### Local development offline

In order to work locally without and active internet connection, you need to use a mock auth0 api server.

First time only:

    npm install -g api-mock
    git clone https://github.com/neam/auth0-mock-api.git ../auth0-mock-api
    openssl genrsa -des3 -out ../auth0-mock-api/server.key 1024 # any passphrase will do
    openssl req -new -key ../auth0-mock-api/server.key -out ../auth0-mock-api/server.csr
    openssl x509 -req -days 365 -in ../auth0-mock-api/server.csr -signkey ../auth0-mock-api/server.key -out ../auth0-mock-api/server.crt

Every time, so that the server is actively running:

    api-mock ../auth0-mock-api/auth0-mock-api-blueprint.md --port 2999 --ssl-port 3000 --ssl-enable -ssl-host 127.0.0.1 --ssl-key ../auth0-mock-api/server.key --ssl-cert ../auth0-mock-api/server.crt --cors-disable false

To prevent having to generate and maintain mock authentication tokens, enable a data profile to be used offline by uncommented one of the LOCAL_OFFLINE_DATA lines in `.env`.

Then run the develop-script with the "offline" argument:

    bin/angular-frontend-develop.sh offline
