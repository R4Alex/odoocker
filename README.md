# Table of Contents

- [The Ultimate Odoo Docker Tool](#the-ultimate-odoo-docker-tool)
  - [Why Odoocker Stands Out](#why-odoocker-stands-out)
  - [Quick Setup Guide](#quick-setup-guide)
  - [The `.env` File](#the-env-file)
    - [Environment-based actions](#environment-based-actions)
      - [Fresh or Restore](#1-fresh-or-restore)
      - [Local](#2-local)
      - [Debug](#3-debug)
      - [Testing](#4-testing)
      - [Full](#5-full)
      - [Staging](#6-staging)
      - [Production](#7-production)
  - [Pro(d) Tips](#prod-tips)
    - [Define the following aliases](#1-define-the-following-aliases)
    - [NEVER run **docker-compose down -v** in Production](#2-never-run-docker-compose-down-v-in-production)
    - [Colorize your branches](#3-colorize-your-branches)
    - [Odoo Shell](#4-odoo-shell)
    - [Odoo Scaffold](#5-odoo-scaffold)
  - [DB Connection](#db-connection)
    - [PgAdmin Container](#pgadmin-container)
  - [Deployment Process](#deployment-process)
  - [Final Note](#final-note)

# The Ultimate Odoo Docker Tool

Welcome to Odoocker, a game-changer in the world of Odoo Development and Deployment. This tool is meticulously crafted to revolutionize your experience with Odoo, ensuring simplicity, efficiency, and a top-tier development journey. And while it’s rooted in the principles of the Official Odoo Docker setup, it goes several steps beyond.

**Best of all**, you don't need any prior knowledge of **Odoo** or **Docker** to start your journey with Odoocker.

### Why Odoocker Stands Out:

1. **Minimalistic Server Approach**: In an era where less is more, we keep server clutter at bay. With Odoocker, you'll witness a decluttered server environment, stripped down to the bare essentials. This minimalism ensures optimal resource utilization, leading to faster response times, reduced chances of conflicts, and a smooth-running Odoo instance.

2. **Usability at its Core**: Odoocker is user-centric. By keeping the interface intuitive and the processes straightforward, we ensure that even newcomers to the Odoo ecosystem can set up and deploy with relative ease.

3. **Dynamic Yet Simple Configuration**: Customization doesn't have to be complicated. With our `.env` setup, adapting your Odoo instance to your unique needs is a breeze, ensuring you have the flexibility without the fuss.

4. **Streamlined Development Cycle**: With our intuitive steps and tips, deploying Odoo becomes as simple as 1-2-3. The goal is clear: *to ensure developers do what they love most, code*. 

5. **Clean, Unadulterated Performance**: With a spotlight on a single Odoo instance and a decluttered server, performance optimization is a given. Expect faster load times, smoother operations, and a user experience that’s top-notch.

In essence, Odoocker isn't just another tool in the developer's arsenal. It's a philosophy. A commitment to clean, efficient, and delightful Odoo development. So, whether you’re a seasoned Odoo veteran or just starting your journey, Odoocker is here to make sure it’s smooth sailing all the way.

Note: We always recommend keeping abreast with the official documentation for the most detailed and updated insights. Knowledge is power, after all.

**Note:** While we've aimed to make things simpler, always refer to the official documentation for detailed information and updates. After all, knowledge is power!

# Quick Setup Guide:

1. **Clone and Prep**: Get your hands on this wonder-tool in seconds with a simple clone and a few copy commands:
```
git clone git@github.com:yhaelopez/odoocker.git
cp .env.example .env && cp docker-compose.override.local.yml docker-compose.override.yml
```
2. **Hosts & Domains**: To ensure everything runs smoothly, remember to add the necessary domains to your hosts file.

For *Unix*:
```
echo '127.0.0.1 erp.odoocker.test' | sudo tee -a /etc/hosts
echo '127.0.0.1 pgadmin.odoocker.test' | sudo tee -a /etc/hosts
```
For *Windows*, manually add these lines to C:\Windows\System32\drivers\etc\hosts:
```
127.0.0.1 erp.odoocker.test
127.0.0.1 pgadmin.odoocker.test
```

# The `.env` File
The environment variables located in [`.env`](https://github.com/yhaelopez/odoocker/blob/main/.env.example) provide dynamic configurations to Odoo and the project in general.
The [`odoo.conf`](https://github.com/yhaelopez/odoocker/blob/main/odoo/odoo.example.conf) file is generated by the [`odoorc.sh`](https://github.com/yhaelopez/odoocker/blob/main/odoo/odoorc.sh) script based on the environment variables.

This file is divided in sections, you most likely are going to focus on the `Main Configuration` section. This provides quick access to project and `odoo.conf` variables. The rest of section are container specific variables for different container. (Note: you may find *repeated variables* like `SUPPORT_EMAIL=${SUPPORT_EMAIL}` which interacts with different containers. This is to explicitly denote in the `.env` file that this variable is being shared through those containers.

Sample `.env` file:
```
# Odoo
ODOO_VERSION=16.0
UPDATE=custom_account_addon
INIT=
LOAD=base,web
WORKERS=2

# Enterprise (GitHub User with access to Odoo Enterprise [https://github.com/odoo/enterprise] Repo)
GITHUB_USER=yhaelopez
GITHUB_ACCESS_TOKEN=ghp_token

# Database
ADMIN_PASSWD=odoo
DB_HOST=postgres (container or external host)
DB_PORT=5432
DB_NAME=my-odoo-db
DB_USER=odoo
DB_PASSWORD=odoo
...
```

## Environment-based actions:
[`odoo/entrypoint.sh`](https://github.com/yhaelopez/odoocker/blob/main/odoo/entrypoint.sh) file is the gateway for our Odoo container. Depending on the `APP_ENV` and the rest of the environment variables, it determines how to start the Odoo service (like local, testing, production, etc.) with different configurations.

All environments receive the whole `.env` file variables. However, some of them have fixed command-line variables specific for each environment. For example, some of them may have `--limit-time-cpu=3600` because some databases are so big that may require a huge amount of seconds. Setting 1 hour ensures any DB can be imported (this can change as needed in the specific project). These values in command line overwrite the ones in the `.env` file.

To bring up most of the following environments run:
```
docker-compose down && docker-compose up -d --build && docker-compose logs odoo
```

Here are the descriptions of each of them.

### 1. Fresh or Restore
These environments (`APP_ENV=fresh` or `APP_ENV=restore`) will have no database created and are perfect for setting up a fresh database instance or restoring a production database.

### 2. Local:
This environment (`APP_ENV=local`) will strictly follow the `.env` variables with no command-line overwrites. You'll most likely be using this regularly.
Use `DEV_MODE=reload,qweb` to activate hot reload when changing `python` and `xml` files.
If you prefer to update the packages everytime you restart Odoo container, you can set `UPDATE=module1,module2,module3`.

### 3. Debug:
This environment (`APP_ENV=debug`) works same way as local, but it starts Odoo using the `debugpy` library. Thanks to our [`.vscode/launch.json`](https://github.com/yhaelopez/odoocker/blob/main/.vscode/launch.json), if you are using Visual Studio Code, you start a Debugger session and the container will be aware of your breakpoints and stop wherever you need. This is my favorite environment to work since I use the debugger a lot while developing.

### 4. Testing:
This environment (`APP_ENV=testing`) is specific for running tests (and will be included in a CI/CD pipeline in a future version). It help us test the modules we are developing to ensure a safe deployment.
A `test_DB_NAME` database is automagically created.
The `ADDONS_TO_TEST=addon_1` are installed in that fresh DB.
Use `TEST_TAGS=test_tag_1` to filter your tests.

**NOTE: Avoid running tests without tags**; otherwise, it will trigger tests in all installed addons and we don't want this. For now, let's assume Odoo Community & Enterprise tests passed and only focus on the things you need to test.

### 5. Full:
This environment (`APP_ENV=full`) will install the `INIT` modules in a new or existing `DB_NAME`. This allows us to have a fresh production database replica.

### 6. Staging:
This environment (`APP_ENV=staging`) sets `UPDATE=all`; this allows us to *update* all installed addons at once.
It also allows to install new packages before the upgrade through `INIT`.

It's highly recommended to use this command to run this environment

```
docker-compose down && git pull && docker-compose pull && docker-compose build --no-cache && docker-compose up -d && docker-compose logs -f odoo
```

This will `pull` the latest *Odoo Community, Enterprise, Extra and Custom addons*, basically, it upgrades the whole Odoo instance to the newest. Additionally, it will also pull the latest images of the other containers in this project.

**NOTE: Do not bring down & up again unless you want to perform a whole update again.**

### 7. Production:
This is the production environment (`APP_ENV=production`). It ensures no demo data is loaded and debugging is turned off. It also brings up the `Let's Encrypt` container, so you won't worry about `SSL Certificates` anymore! Some `.env` variables are overwritten in this setup.

- Take down previous setup of containers
```
docker-compose down
```
- Replace the `docker-compose.override.yml` with `docker-compose.override.production.yml` to bring `Let's Encrypt` container.
```
cp docker-compose.override.production.yml docker-compose.override.yml
```
- Rebuild the containers
```
docker-compose up -d --build && docker-compose logs odoo
```

# Pro(d) Tips
The following tips will enhance your developing and production experience.

## 1. Define the following aliases:
```
alias odoo='cd odoocker'

alias hard-deploy='docker-compose down && git pull && docker-compose pull && docker-compose build --no-cache && docker-compose up -d && docker-compose logs -f odoo'

alias deploy='docker-compose down && git pull && docker-compose up -d --build && docker-compose logs -f --tail 2000 odoo'

alias logs='docker-compose logs -f --tail 2000 odoo'
```

## 2. NEVER run `docker-compose down -v` in Production
...without having a `tested backed up` database

Have in mind that dropping volumes will destroy DB data, Odoo Conf & Filestore, *Let's Encrypt certificates, and more!*. If you execute this command several times in `prod` in a short period of time, you may reach the `Let's Encrypt` certificates limit and Odoocker won't be able to generate new ones after **several hours**.

## 3. Odoo Shell
1. Log into the odoo container
```
docker-compose exec odoo bash
```
2. Start Odoo shell running:
```
odoo shell --http-port=8071
```

## 4. Odoo Scaffold
1. Log into the odoo container
```
docker-compose exec -u root odoo
```
2. Navigate to custom addons folder inside the container
```
cd /usr/lib/python3/dist-packages/odoo/custom-addons
```
3. Create new addons running:
```
odoo scaffold <addon_name>
```
- The new addon will be available in the `odoo/custom_addons` folder in this project.

## 5. Colorize your branches
Add the following to `~/.bashrc`
```
# Color git branches
function parse_git_branch () {
  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1)/'
}

if [ "$color_prompt" = yes ]; then
    #PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
    # Color git branches
    PS1="${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w \[\033[01;31m\]\$(parse_git_branch)\[\033[00m\]\$ "
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt
```

# DB Connection
- Any other Postgres Database Manager can connect to the DB using the `.env` credentials.

## PgAdmin Container
- This project comes with a PgAdmin container which is loaded only in `docker-compose.override.pgadmin.yml`.
In order to manage DB we provide a pgAdmin container.
In order to bring this up, simply run:
```
docker-compose -f docker-compose.yml -f docker-compose.override.yml -f docker-compose.pgadmin.yml up -d --build
```
And to turn down
```
docker-compose -f docker-compose.yml -f docker-compose.override.yml -f docker-compose.pgadmin.yml down
```

If your instance has pgAdmin, make sure you adapt this to your aliases.

# Deployment Process
Note: the deployment process is easier & faster with aliases.

1. Backup the production Databases from `/web/database/manager`.
2. Run
```
sudo apt update && sudo apt upgrade -y
```
- If packages are kept, install them
```
sudo apt install <kept packages>
```
3. Restart the server
```
sudo reboot
```
- Make sure there are no more upgrades or possible kept packages
```
sudo apt update && sudo apt upgrade -y
```
4. Go to the project folder in /home/ubuntu or (~)
```
cd ~/odoocker
```
or with alias:
```
odoo
```
5. Pull the latest `main` branch changes.
```
git pull origin main
```
6. Set `Staging` environment
7. Set `Production` environment

# Footnote
This project is based on the [Official Odoo Docker](https://hub.docker.com/_/odoo/) image. We've strived to ensure a seamless integration with the original Docker setup while making necessary customizations to suit our requirements. We encourage contributors and users to frequently refer to the official documentation for foundational concepts and updates. Thank you for your continued support and trust in our project."