# Grouparoo `app-example-gcp`

_**An example Grouparoo project for deploying Grouparoo on Google Cloud Platform (`GCP`) with Node.JS**_

Goal: To create Grouparoo deployment that:

- Relies on Google's hosted databases for persistence.
- Is process-managed locally

Limitations:

- This quick configuration does not have application high-availability, as it relies on a single host. This is not a [12-factor app](https://12factor.net/) with configuration stored in the Environment
- Will not re-deploy as your code changes. You can do that manually, or use a tool like Chef or Ansible.

## Repository Configuration

1. Create a new Grouparoo project. Learn more @ https://www.grouparoo.com/docs/installation.

```
npm install -g grouparoo
grouparoo init .
```

2. Install the Grouparoo plugins you want, e.g.: `grouparoo install @grouparoo/postgres`. Learn more @ https://www.grouparoo.com/docs/installation/plugins

## Deployment Steps

1. Configure a new GCP project for Grouparoo, or choose an existing one

2. Create the Grouparoo Database

   - Choose `Postgres CLoud SQL`
   - A `standard` server should be OK, we recommend `4 vCPU, 15 GB`
   - Enable the Private IP address & an Automatically Assigned IP Range.
   - Once the database is running, create a new database like `grouparoo_production`
   - Note and build `DATABASE_URL` in a scratch document. The default username is `postgres`.

3. Create the Grouparoo Redis Server

   - Choose `Memorystore - Redis`
   - Create 1 replica
   - `1GB` or capacity should be enough
   - "Enable Auth" but disable "in transit encryption" for redis
   - Use the Private IP address & an Automatically Assigned IP Range settings you created for Postgres above.
   - Once redis is up and running, note the AUTH string. You'll use this as the "password" in `REDIS_URL`. The username should be a 1-char letter which signals it should be ignored, like `"r"`
   - Note and build `REDIS_URL` in a scratch document

4. Create a Compute Engine Server to run your Grouparoo Instances

   - Choose a machine with at least 2CPUs and 4GB of memory
   - Choose the Ubuntu 20.04 LTS Minimal Boot Disk
   - Enable the HTTP and HTTPS ports in the firewall
   - ssh into the instance from the console, and:

```bash
# Update packages
sudo apt-get update && sudo apt-get upgrade -y

# Install Tools
sudo apt-get install iptables net-tools vim git -y

# Install NVM and Node.JS
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
source ~/.bashrc
nvm install v14

# Git clone/copy your code to the server
# For example...
git clone https://github.com/grouparoo/app-example-gcp
cd app-example-gcp

# << be sure that you are in your application directory >>

# Install packages
npm install

# In your project, copy and configure the .env file to match your environment
cp .env.example .env
vim .env # Fill in your server information; enable both WEB and WORKERS

# Configure an internal forward from port 3000 (the app) to port 80
# Note: your ethernet interface may not be `ens4`, it may be something else.  Run `ifconfig -a` to see your network interfaces
sudo iptables -A INPUT -i ens4 -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -i ens4 -p tcp --dport 3000 -j ACCEPT
sudo iptables -A PREROUTING -t nat -i ens4 -p tcp --dport 80 -j REDIRECT --to-port 3000

# install iptables-persistent to save your rules
sudo apt-get install iptables-persistent -y # answer 'yes' to save your existing iptables rules

# prepare logging directory for access
sudo mkdir /var/log/grouparoo
sudo chown -R $USER /var/log/grouparoo
```

Notes on configuring your `.env` on the server:

- Keep the `PORT=3000`. We've configured `iptables` to forward all traffic on port 80 to 3000 internally. This means our unprivileged app user can doesn't need to run the app as `sudo`.
  - The `WEB_URL` will be something like `WEB_URL=http://1.2.3.4`
- For this example, we'll be making a single process which acts as both web ann background workers (`WORKERS=10` and `WEB_SERVER=true`)
- Use the values from the new Google Cloud services for `DATABASE_URL` and `REDIS_URL`

Now, we'll use `pm2` to monitor and run the Grouparoo application:

```bash
# << be sure that you are in your application directory >>

# Install PM2
npm install -g pm2

# Install PM2 into your OS's init system
pm2 startup # learn more @ https://pm2.keymetrics.io/docs/usage/startup/
# Follow the prompts to register pm2 with systemd

# start the grouparoo process
pm2 start --name grouparoo --interpreter=bash node_modules/@grouparoo/core/bin/start

# see process information
pm2 list # see running processes
pm2 logs # follow process logs (ie --tail)
pm2 monit # fancy process monitor
```

6. Configure Monitoring

7. Configure Load Balancer, SSL, and DNS

## Updating the Deployment

SSH to your server

```bash
# in the application directory
git pull
npm install
pm2 restart grouparoo
```

## Notes

In this example, we deployed a single server type that acts as both a WEB and BACKGROUND WORKER service for simplicity. For more robust deployments, you may want to split these tasks to different servers.

You may want to modify logging behavior with:

- `GROUPAROO_LOGS_STDOUT_DISABLE_TIMESTAMP=true`- Cloud Run adds timestamps to all log messages
- `GROUPAROO_LOGS_STDOUT_DISABLE_COLOR=true`- Cloud Run will not render log messages in color
- `GROUPAROO_LOGS_PATH="/var/log/grouparoo"`- Logs are stored in `/var/log/grouparoo` with this setting enabled.

---

Visit https://github.com/grouparoo/app-examples to see other Grouparoo Example Projects.
