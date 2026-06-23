# Lightpack Deployment Suite

> *From your local dev-machine to production in minutes, not hours.*

**Lightpack PHP** ships with a complete deployment suite that makes it painless to provision fresh Ubuntu servers and deploy your applications right from your local machine. This makes deploying and maintaining your applications a breeze. The deployment suite includes commands to:

- Provision custom Ubuntu VPS
- Deploy applications per environment
- Manage domains and SSL certificates
- Manage database backups and restores
- Manage scheduled cron jobs and commands
- Manage backround queue processes and workers
- View and stream processes and applications logs
- Run arbitrary commands on the server

These tools require a Unix-like environment because they use SSH and other Unix commands. If you are on `macOS/Linux`, you are already set up. If you are on `Windows`, you can use *WSL2*, or a VM to run these commands. Before getting started, you'll need:

- A fresh Ubuntu 22.04 or 24.04 LTS server
- and SSH access to the server with root user privileges

So go ahead and provision a new Ubuntu machine and make sure you have SSH access to it with root user privileges. You can use any VPS provider including AWS, DigitalOcean, Linode, Vultr, Google Cloud, Azure, etc.

---

## Quick Start

### 1. Create Deploy Config

Run this command to publish the deploy configuration file:

```bash
php console create:config --support=deploy
```

This creates `config/deploy.php` with a sample **production** environment.

Each environment supports these options which you need to configure as per your setup:

| Option | Description |
|---|---|
| `host` | Ubuntu server IP |
| `key` | Local SSH private key path |
| `path` | App deployment path on the server |
| `repo` | Git repository URL |
| `branch` | Git branch to deploy |

You can define multiple environments (e.g., `staging`, `production`) in the same file. Every command documented below takes an environment as an argument, and defaults to `production` if you leave it out.

### 2. Provision the Server

Provisioning is a one-time setup that configures the server with all necessary dependencies and configurations like Nginx, PHP, MySQL, etc. To provision the `production` server, run:

```bash
php console server:provision production
```

This will prompt you with required default information like root username, PHP version, database, timezone, etc which you can accept or modify as per your setup. Once you confirm, it will start provisioning the server which usually takes a few minutes. When done, root SSH is disabled and a new `deploy` user is created to access the server.

It will also print the security credentials like the deploy user's password, SSH public key, created database username and password, etc. which you should save securely.

### 3. Prepare Environment File

Create `.env.<env>` in your project root (e.g. `.env.production` for the `production` environment). You can simply copy your existing `.env` file and rename it to `.env.production` and update the values as per your setup.

### 4. SSH Deploy Key

After provisioning, the deploy user's SSH public key is printed in the terminal. Copy it in your Git repository's deploy keys settings. For GitHub, go to: **Settings > Deploy keys > Add deploy key**. Do **not** allow write access.

### 5. Deploy Application

To deploy your application to the default `production` server, run:

```bash
php console app:deploy
```

Copies your `.env.production` to the server, pulls code, installs dependencies, symlinks storage, runs migrations, and reloads PHP-FPM.

### 6. Add Domain

Point your DNS A record to the server IP, then:

```bash
php console server:site:add
```

### 7. Enable HTTPS

After adding the domain, enable a secure HTTPS connection using Let's Encrypt certificate:

```bash
php console server:site:ssl
```

**⭐️ Congratulations! Your application is now live with HTTPS ⭐️**

If everything goes well, you should be able to access your application at the domain you added. Usually DNS propagation takes a few minutes. So if your browser doesn't load the domain immediately, try refreshing the page after a few minutes.

---

## Rolling Back

```bash
php console app:rollback
```

Go back further with `--steps=3`.

**Important:** Rollback reverts code only. It does not revert database migrations.

---

## Queue Workers

### Setup (Once)

```bash
php console server:queue:setup production
```

This creates a supervised worker group. By default it is named after the environment (`production`). You can give it a custom name:

```bash
php console server:queue:setup production --name=emails --queue=emails --workers=2
```

**What `--name` means:** It is the label for the worker group in Supervisor, not a queue name. Each `--name` creates a separate group of processes. The `--queue` flag tells those processes which job queues to process.

| Flag | What it does |
|---|---|
| `--name` | Supervisor group label (default: environment name) |
| `--queue` | Comma-separated job queue names to process (default: `default`) |
| `--workers` | Number of parallel processes (default: `1`) |
| `--cooldown` | Seconds before voluntary restart to prevent memory leaks (default: `3600`) |
| `--stop-wait` | Seconds to wait before force-kill on shutdown (default: `60`) |

### Multiple Worker Groups

You can run multiple worker groups on the same server, each processing different queues:

```bash
php console server:queue:setup production --name=default --queue=default  --workers=4
php console server:queue:setup production --name=emails --queue=emails   --workers=2
```

Each group is independent. `default` runs 4 workers processing the `default` queue. `emails` runs 2 workers processing the `emails` queue.

### Managing Workers

```bash
php console server:queue:start   production          # start default group
php console server:queue:stop    production          # stop default group
php console server:queue:restart production          # restart default group
php console server:queue:status  production          # check default group
```

For a named group:

```bash
php console server:queue:restart production --name=emails
```

### Viewing Worker Logs

```bash
php console server:queue:logs:view production          # last 50 lines
php console server:queue:logs:view production --lines=200
php console server:queue:logs:tail production         # live stream
```

### Restart Workers After Deploy

Queue workers are **not** restarted automatically. After each deploy, run:

```bash
php console server:queue:restart production
```

---

## Scheduled Tasks

```bash
php console server:schedule:setup production   # install cron job
php console server:schedule:status production    # check if installed
php console server:schedule:remove production    # remove cron job
```

On the server, `php console schedule:events` runs every minute to execute due tasks.

---

## Remote .env Files

When you deploy, your local `.env.production` is automatically copied to the server as `.env`.

To inspect the remote `.env` without overwriting your local copy:

```bash
php console server:env:pull production
```

Saved to `storage/env/production.env`.

---

## Database

```bash
php console db:backup production     # download timestamped dump
php console db:restore production    # upload and restore
php console db:create production     # new DB + user
```

Use `db:create` when deploying a second application to the same server.

---

## Logs

```bash
php console server:logs:view production   # last N lines
php console server:logs:tail production     # live stream
```

---

## Security Checklist

After provisioning:

- [ ] Root SSH login is disabled
- [ ] Password authentication is disabled
- [ ] Firewall is active (only 22, 80, 443 open)
- [ ] Fail2Ban is running
- [ ] Automatic security updates are enabled
- [ ] `.env` is not in Git
- [ ] `.env.*` files are in `.gitignore`
- [ ] GitHub deploy key does not have write access

---

## Multiple Apps on One Server

Provision once, then add separate environments in `config/deploy.php`:

```php
'deploy' => [
    'blog' => [
        'host' => '1.2.3.4',
        'key'    => '~/.ssh/id_rsa',
        'path' => '/var/www/blog',
        'repo' => 'git@github.com:you/blog.git',
        'branch' => 'main',
    ],
    'shop' => [
        'host' => '1.2.3.4',
        'key'    => '~/.ssh/id_rsa',
        'path' => '/var/www/shop',
        'repo' => 'git@github.com:you/shop.git',
        'branch' => 'main',
    ],
],
```

```bash
php console app:deploy blog
php console app:deploy shop
php console server:site:add blog --domain=blog.example.com
php console server:site:add shop --domain=shop.example.com
```

Queue workers must use unique `--name` values:

```bash
php console server:queue:setup blog --name=blog-worker
php console server:queue:setup shop --name=shop-worker
```

---

## Command Reference

### Core

| Command | Description |
|---|---|
| `php console server:provision <env>` | Provision a fresh server |
| `php console app:deploy <env>` | Deploy code |
| `php console app:rollback <env>` | Roll back to previous commit |

### Schedule

| Command | Description |
|---|---|
| `php console server:schedule:setup <env>` | Install cron job |
| `php console server:schedule:remove <env>` | Remove cron job |
| `php console server:schedule:status <env>` | Check cron status |
| `php console schedule:events` | Run due scheduled events (on server) |

### Queue

| Command | Description |
|---|---|
| `php console jobs:run` | Run worker locally |
| `php console jobs:retry` | Retry failed jobs |
| `php console server:queue:setup <env> [options]` | Install worker (once) |
| `php console server:queue:start <env> [--name]` | Start worker |
| `php console server:queue:stop <env> [--name]` | Stop worker |
| `php console server:queue:restart <env> [--name]` | Restart worker |
| `php console server:queue:status <env> [--name]` | Worker status |
| `php console server:queue:list <env>` | List all worker groups |
| `php console server:queue:logs:view <env> [--name] [--lines=50]` | View worker logs |
| `php console server:queue:logs:tail <env> [--name]` | Stream worker logs |

### Database

| Command | Description |
|---|---|
| `php console db:backup <env>` | Backup database |
| `php console db:restore <env>` | Restore from backup |
| `php console db:create <env> [--db=] [--user=]` | Create new database + user |

### Logs

| Command | Description |
|---|---|
| `php console server:logs:view <env> [--lines=50] [--file]` | View recent logs |
| `php console server:logs:tail <env>` | Stream logs live |

### Sites

| Command | Description |
|---|---|
| `php console server:site:add <env> [--domain=]` | Add Nginx virtual host |
| `php console server:site:remove <env> --domain=` | Remove Nginx virtual host |
| `php console server:site:ssl <env> [--domain=] [--email=]` | Install SSL certificate |

### .env Files

| Command | Description |
|---|---|
| `php console server:env:pull <env>` | Download remote .env |

### Server

| Command | Description |
|---|---|
| `php console server:run <env> --cmd="..."` | Run any command on the server |

---
