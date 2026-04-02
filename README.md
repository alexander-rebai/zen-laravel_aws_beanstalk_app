# Task Manager-2 - Laravel CRUD App

A simple task management CRUD application built with Laravel 13, deployed on AWS Elastic Beanstalk.

## ⚠️ Aikido Zen Firewall Installation - Action Required

This application has been configured with the Aikido Zen in-app firewall for security protection. To complete the installation:

### 1. Move the middleware file

The file `AikidoMiddleware.php` in the root directory needs to be moved to the correct location:

```bash
mkdir -p app/Http/Middleware
mv AikidoMiddleware.php app/Http/Middleware/AikidoMiddleware.php
```

### 2. Set your Aikido token

Update the `.env` file (or set via AWS Elastic Beanstalk environment variables):

```bash
AIKIDO_TOKEN=your-actual-aikido-token-here
AIKIDO_BLOCK=false
```

Get your token from: https://help.aikido.dev/doc/creating-an-aikido-zen-firewall-token/doc6vRJNzC4u

For Elastic Beanstalk, set the token via:

```bash
eb setenv AIKIDO_TOKEN=your-actual-aikido-token-here AIKIDO_BLOCK=false
```

### 3. Deploy

The Aikido PHP extension will be automatically installed on the Elastic Beanstalk instance via the `.ebextensions/03_aikido_php_firewall.config` file.

```bash
eb deploy
```

### What Aikido Zen protects against

- SQL injection
- Command injection
- Path traversal
- SSRF (Server-Side Request Forgery)
- User blocking and rate limiting (via the middleware)

Logs are written to `/var/log/aikido-*/` on the EB instance for troubleshooting.

---

## Requirements

- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) configured with credentials
- [EB CLI](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html) (`pip install awsebcli`)

## Deploy to Elastic Beanstalk

### 1. Initialize the EB application

```bash
eb init -p "PHP 8.4" task-manager --region us-east-1
```

### 2. Create the environment with an RDS MySQL database

```bash
eb create task-manager-env \
  --database.engine mysql \
  --database.username admin \
  --database.password <your-db-password> \
  --envvars APP_KEY=<your-app-key>
```

Generate an `APP_KEY` locally if needed:

```bash
php artisan key:generate --show
```

### 3. Set environment variables

```bash
eb setenv \
  APP_ENV=production \
  APP_DEBUG=false \
  APP_KEY=base64:yourkey \
  LOG_CHANNEL=stderr
```

The RDS variables (`RDS_HOSTNAME`, `RDS_PORT`, `RDS_DB_NAME`, `RDS_USERNAME`, `RDS_PASSWORD`) are injected automatically when you attach an RDS instance to your environment.

### 4. Deploy updates

```bash
eb deploy
```

### 5. Open the app

```bash
eb open
```

## CI/CD with GitHub Actions

Pushing to `main` automatically runs tests and deploys to Elastic Beanstalk.

### Required GitHub Secrets

Add these in **Settings > Secrets and variables > Actions**:

| Secret                | Description                       |
|-----------------------|-----------------------------------|
| `AWS_ACCESS_KEY_ID`   | IAM user access key               |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret key             |

The IAM user needs permissions for `elasticbeanstalk:*` and `s3:*` (for uploading deployment artifacts).

### Workflow

1. **Test** -- Installs dependencies, spins up a MySQL service container, runs `php artisan test`
2. **Deploy** -- Zips the app (production deps only), deploys to EB via `beanstalk-deploy`

## Project Structure (Beanstalk)

```
.ebextensions/
  01-laravel.config          # PHP settings, document root, env vars
  02-permissions.config      # storage/cache directory permissions
.github/workflows/
  deploy.yml                 # CI/CD: test + deploy on push to main
.platform/
  hooks/postdeploy/
    01_migrate.sh            # Runs migrations + caches config after deploy
  nginx/conf.d/
    elasticbeanstalk/
      laravel.conf           # Nginx rewrite rules for Laravel
```

## Features

- Create, read, update, and delete tasks
- Task status tracking (Pending, In Progress, Completed)
- MySQL database via RDS with automatic migrations on deploy

## Useful Commands

```bash
eb status          # Check environment health
eb logs            # View application logs
eb ssh             # SSH into the instance
eb terminate       # Tear down the environment
```
