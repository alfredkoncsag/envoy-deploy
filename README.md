# Laravel Envoy Deploy Script

This repository includes an Envoy.blade.php script that is designed to provide a basic "zero-downtime" deployment option using the open-source [Laravel Envoy](http://laravel.com/docs/7.x/envoy) tool.

## Requirements

This Envoy script is designed to be used with Laravel 7 projects and can be used within the Laravel root, or downloaded separately and included in your Laravel project.

## Installation

Your must have Envoy installed using the Composer global command:

	composer global require "laravel/envoy"


## Setup

### Config

Envoy Deploy uses [DotEnv](https://github.com/vlucas/phpdotenv) to fetch your server and repository details. If you are installing within a Laravel project, there will already be a `.env` file in the root, otherwise simply create one.

The following configuration items are required:

  - `DEPLOY_SERVER_HOST`
  - `DEPLOY_REPOSITORY`
  - `DEPLOY_PATH`

For example, deploying the standard Laravel repository on a server, we might use:

```
DEPLOY_SERVER_HOST=
DEPLOY_REPOSITORY=https://github.com/laravel/laravel.git
DEPLOY_PATH=/var/www/example.com
DEPLOY_HEALTH_CHECK=https://example.com
```

The `DEPLOY_PATH` (server path) should already be created in your server and must be a blank directory.

### Host

Envoy Deploy uses symlinks to ensure that your server is always running from the latest deployment. As such you need to setup your Apache or Nginx host to point towards the `host/current/public` directory rather than simply `host/public`.

For example:

```
server {
    listen 80;
    server_name example.com;
    root /var/www/example.com/current/public;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

## Usage

### Init

When you're happy with the config, run the init task on your local machine by running the following.

	envoy run init

You only need to run the init task once.

The init task creates a `.env` file in your root path (from your `.env.example` file), so make sure and update the environment variables appropriately. Once you have run the init task, you should proceed to run the deploy task (below).

### Deploy

Each time you want to deploy simply run the deploy task on your local machine in the repository direcory

	envoy run deploy

You can specify the Laravel environment (for artisan:migrate command) and git branch as options

	envoy run deploy --branch=develop --env=development

### Deploy with Cleanup

If you could like to deploy your repository and cleanup any old deployments at the same time, you can run

	envoy run deploy --cleanup

Or alternatively, if you need to:

This will run the deploy script and then delete any old deployments older than 48 hours, limiting the number deleted to 5.

You can also run the cleanup script independently (without deploying) using

	envoy run deployment_cleanup

### Health Check

If you would like to perform a health check (for 200 response) after deploying, simply add your site's URL in the .env file:

```
DEPLOY_HEALTH_CHECK=https://example.forge.com
```

### Rollback

To rollback to the previous deployment (e.g. when a health check fails), you can simply run 

	envoy run rollback

## Commands

#### `envoy run init`

Initialise server for deployments

    Options
        --env=ENVIRONMENT        The environment to use. (Default: "production")
        --branch=BRANCH          The git branch to use. (Default: "master")

#### `envoy run deploy`

Run new deployment

    Options
        --env=ENVIRONMENT        The environment to use. (Default: "production")
        --branch=BRANCH          The git branch to use. (Default: "master")
        --cleanup                Whether to cleanup old deployments

#### `envoy run deployment_cleanup`

Delete any old deployments, leaving just the last 4.

#### `envoy run rollback`

In case the health check does not work, you can rollback and it will use the previous deploy.

## How it Works

Your `$path` directory will look something like this after you init and then deploy.

	20170910131419/
	current -> ./20170910131419
	storage/
	.env

As you can see, the *current* directory is symlinked to the latest deployment folder

Inside one of your deployment folders looks like the following (excluded some laravel folders for space)

	app/
	artisan
	boostrap/
	composer.json
	.env -> ../.env
	storage -> ../storage
	vendor/

The deployment folder .env file and storage directory are symlinked to the parent folders in the main (parent) path.
