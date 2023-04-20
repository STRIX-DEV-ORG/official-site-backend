# 🚀 Getting started with Strapi

Strapi comes with a full featured [Command Line Interface](https://docs.strapi.io/developer-docs/latest/developer-resources/cli/CLI.html) (CLI) which lets you scaffold and manage your project in seconds.

### `develop`

Start your Strapi application with autoReload enabled. [Learn more](https://docs.strapi.io/developer-docs/latest/developer-resources/cli/CLI.html#strapi-develop)

```
npm run develop
# or
yarn develop
```

### `start`

Start your Strapi application with autoReload disabled. [Learn more](https://docs.strapi.io/developer-docs/latest/developer-resources/cli/CLI.html#strapi-start)

```
npm run start
# or
yarn start
```

### `build`

Build your admin panel. [Learn more](https://docs.strapi.io/developer-docs/latest/developer-resources/cli/CLI.html#strapi-build)

```
npm run build
# or
yarn build
```

## ⚙️ Deployment

Strapi gives you many possible deployment options for your project. Find the one that suits you on the [deployment section of the documentation](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/deployment.html).

### Have you read the previous sentence? well, those bastards are lying, this is not easy, it is a pain in the...

#### Configure the basics in your droplet, hosting, ec2, whatever tool you're using

- Go the the to your service provider panel, create an ssh access door and access it by using `ssh [username]@ip-address` and your password/ssh key
- Don't forget to create a non-root user, for that just execute the next few bash lines

```bash
adduser username # Create the non-root user
# After executing the previous command you will be asked
# for some information, rememb you will need to keep track
# of username and password, write them down...

usermod -aG sudo username # Give sudo privileges to the non-root user
```

- Login with your new non-root user

- Add some protection rules with basic firewall configuration

```bash
sudo ufw allow OpenSSH # Allow SSH secure communication
sudo ufw enable # Make the previous rule take effect
sudo ufw status # Make sure OpenSSH is listed and Allowed
```

- Install nodejs and make sure it has execution privileges for all your users

```bash
sudo apt update # Grant you are using most recent packages
sudo apt install nodejs # Install the latest/stable release
sudo apt install npm # Make sure npm gets installed

# If you need any other node version...
sudo npm cache clean -f # Force to clean any npm cache
sudo npm install -g n # Install node helper
sudo n version # sudo n 18.15.0 as example
node --version # Check your node version

cd ~ # Go to root folder
mkdir ~/.npm-global # Create node global directory
npm config set prefix '~/.npm-global' # node_modules will be installed here
sudo nano ~/.profile # Create profile file
--------------------
# Add these lines
# set PATH so global node modules install without permission issues
export PATH=~/.npm-global/bin:$PATH
--------------------
source ~/.profile # Use previous file as system variables
```

- Make sure you have git installed, otherwise, install it

#### Database creation

> As several users recommend using posgresql, we are going to stick to that configuration

We are going to install postgresql, modify the default user's password and create a database, don't forget to keep the database port, name, user and user password

```bash
sudo apt update # Grant you are using most recent packages
sudo apt install postgresql postgresql-contrib # Install postgresql packages

sudo systemctl start postgresql.service # Start postgresql service

sudo -i -u postgres # Log into postgres user (was created along postgres installation)

psql # Execute postgresql

# Change postgres password
postgres=# ALTER USER postgres PASSWORD 'new password';

# Create the database
postgres=# createdb dbName
```

#### Local/server repo configurations

Make sure you read everything here carfully, as some misconfigurations may result in several hours spent.

- Create the next folter structure inside the `config` folder: `config/env/production`

- Inside `production` create a file named `database.js`, copy and paste the next snippet:

```js
// path: ./config/env/production/database.js

const { parse } = require("pg-connection-string");

module.exports = ({ env }) => {
  const { host, port, database, user, password } = parse(env("DATABASE_URL"));

  return {
    connection: {
      client: "postgres",
      connection: {
        host,
        port,
        database,
        user,
        password,
        ssl: { rejectUnauthorized: false },
      },
      debug: false,
    },
  };
};
```

- Make sure you commit and push your changes to the repo

- Make sure you are loged into your server with the non-root user, navigate to `/var/www` and clone your repo by giving it `your domain name` as the name of the folder

```bash
cd /var/www # Move to folder
git clone repo-url yourdomainname # clone your repository

# It is very important you clone
# your repo with your domain name
```

- Move to your repo folder, install node dependencies and install posgresql dependencies too.

```bash
cd yourdomainname # Move to folder
npm install # Install project dependencies
npm install pg --save # Install posgresql dependencies
```

- Create `.env` file and add the required variables

```bash
sudo nano .env # Create and open .env file

# Make sure your file has the next variables
------------
DATABASE_PORT=value
DATABASE_NAME=value
DATABASE_USERNAME=value
DATABASE_PASSWORD=value
HOST=value
PORT=value
APP_KEYS=value
API_TOKEN_SALT=value
ADMIN_JWT_SECRET=value
JWT_SECRET=value
DATABASE_URL=value
------------

# This file will only be use for the npm run build command
```

- Build the project with `sudo NODE_ENV=production node run build`

#### Configuring pm2, this bastard...

- Make sure you are in the root folder with `cd ~`

- Install pm2 globally with `npm install pm2@latest -g`

- Make the required configurations by executing the next command lines:

```bash
pm2 init # Create default ecosystem.config.js file
sudo nano ecosystem.config.js # Modify the created file

# Replace the file content wit the next
------------
module.exports = {
  apps: [
    {
      name: 'app_or_service_name',
      cwd: '/var/www/yourdomainname',
      script: 'npm',
      args: 'start',
      env: {
        HOST: 'yourdomainname',
        PORT: '1337',
        NODE_ENV: 'production',
        DATABASE_HOST: 'localhost', // database endpoint
        DATABASE_PORT: '5432',
        DATABASE_NAME: 'dbName', // DB name
        DATABASE_USERNAME: 'postgres', // your username for psql
        DATABASE_PASSWORD: 'postgres_user_password', // your password for psql
        DATABASE_URL: 'postgresql://username:password@host:port/dbName',
      },
    },
  ],
};
------------

# Remember you need to modify values with yours
# DATABASE_URL might be the only database related
# variable being used, provide other ones as a backup
```

- Execute the project by writing `pm2 start ecosystem.config.js`

- Save your pm2 configurations for whenever server needs a reboot

```bash
pm2 startup systemd # Generate the startup command
# Copy and paste the command provided by the previous execution

pm2 save # save the current pm2 configuration and state
```

- Make sure you have no errors by checking the `pm2 logs` command, the service should be up and running

#### Final, hard steps

- Install nginx

```bash
sudo apt update # Grant most recent packages are being used
sudo apt install nginx
```

- Adjust firewall to allow http and https, in other words, allow `Nginx Full`

```bash
sudo ufw allow 'Nginx Full' # Allo http and https
sudo ufw enable # Apply previous changes
sudo ufw status # Check that OpenSSH and Nginx Full are allowed
```

- Make sure nginx is up and running by executing:

```bash
sudo systemctl status nginx # The active status shoud be active and running

# This will show some IP addresses
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'

# Copy and paste the IP addresses into your browser, you should se an nginx message
```

- Make sure your folder under `/var/www` path has the right permissions

```bash
sudho chown -R $USER:$USER /var/www/yourdomainname # Assign ownership
sudo chmod -R 755 /var/www/yourdomainname # Change folder permissions
```

- Create the configuration file for nginx under `/etc/nginx/sites-available`

```bash
sudo nano /etc/nginx/sites-available/yourdomainname # Create and open the file

# Paste the next template, we will modify this one later
------------
server {
        listen 80;
        listen [::]:80;

        root /var/www/yourdomainname;
        index index.html index.htm index.nginx-debian.html;

        server_name yourdomainname;

        location / {
                try_files $uri $uri/ =404;
        }
}
------------
```

- Create a symbolic link to enable the previous config file

```bash
sudo ln -s /etc/nginx/sites-available/yourdomainname /etc/nginx/sites-enabled/
```

- Check that all the configurations are ok with `sudo nginx -t` and restart the nginx service with `sudo systemctl restart nginx`

- Install certbot to add SSL and create the certification

```bash
sudo apt install certbot python3-certbot-nginx # Install certbot and nginx plugin for certbot

sudo certbot -d yourdomainname # Create the certification

sudo certbot renew --dry-run # Make sure the renewal is set
```

- Modify once more the nginx config file

```bash
sudo nano /etc/nginx/sites-available/yourdomainname

# In the first 'server' entry you see,
# you will find some certbot related configs
# right under 'server_name' line and above
# 'listen [::]443 ssl... you will paste...
------------
 location / {
    proxy_pass http://yourdomainname:1337;
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Host $http_host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_pass_request_headers on;
}
------------

# Everything else should remain the same but
# make sure every server_name and $host mention
# has yourdomainname as a value
# and the root remains /var/www/yourdomainname
```

- Check that all the configurations are ok with `sudo nginx -t` and restart the nginx service with `sudo systemctl restart nginx`

#### If you have finished all these steps and still have troubles, there will be no one to help you as you may want ❌

## 📚 Learn more

- [Resource center](https://strapi.io/resource-center) - Strapi resource center.
- [Strapi documentation](https://docs.strapi.io) - Official Strapi documentation.
- [Strapi tutorials](https://strapi.io/tutorials) - List of tutorials made by the core team and the community.
- [Strapi blog](https://docs.strapi.io) - Official Strapi blog containing articles made by the Strapi team and the community.
- [Changelog](https://strapi.io/changelog) - Find out about the Strapi product updates, new features and general improvements.

Feel free to check out the [Strapi GitHub repository](https://github.com/strapi/strapi). Your feedback and contributions are welcome!

## ✨ Community

- [Discord](https://discord.strapi.io) - Come chat with the Strapi community including the core team.
- [Forum](https://forum.strapi.io/) - Place to discuss, ask questions and find answers, show your Strapi project and get feedback or just talk with other Community members.
- [Awesome Strapi](https://github.com/strapi/awesome-strapi) - A curated list of awesome things related to Strapi.

---

<sub>🤫 Psst! [Strapi is hiring](https://strapi.io/careers).</sub>
