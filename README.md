#  React websites on Google Cloud

Certainly! Here is a consolidated list of all the steps required to set up your GoDaddy domain to work with your Google Cloud instance, including both HTTP and HTTPS configuration without duplication:

### Step 1: Configure GoDaddy DNS Settings

1. **Log in to your GoDaddy account** and access your domain's DNS settings.
2. **Add an A Record**:
    - **Type**: A
    - **Name**: @
    - **Value**: 34.16.153.49
    - **TTL**: 600 seconds (or 1 Hour)
3. **Add/Edit the CNAME Record for `www`**:
    - **Type**: CNAME
    - **Name**: www
    - **Value**: @
    - **TTL**: 1 Hour

### Step 2: Configure Google Cloud Firewall Rules

1. **Create a firewall rule for HTTP**:
    ```sh
    gcloud compute firewall-rules create allow-http --allow tcp:80
    ```
2. **Create a firewall rule for HTTPS**:
    ```sh
    gcloud compute firewall-rules create allow-https --allow tcp:443
    ```

### Step 3: Ensure Nginx is Running

1. **SSH into your instance**:
    ```sh
    gcloud compute ssh [INSTANCE_NAME] --zone=[ZONE]
    ```
2. **Check Nginx status**:
    ```sh
    sudo systemctl status nginx
    ```
3. **If Nginx is not running, start it**:
    ```sh
    sudo systemctl start nginx
    ```

### Step 4: Configure Nginx for HTTP and HTTPS

1. **Edit the Nginx configuration file**:
    ```sh
    sudo nano /etc/nginx/sites-available/default
    ```
2. **Update the configuration to include HTTP to HTTPS redirection and HTTPS**:
    ```nginx
    server {
        listen 80;
        listen [::]:80;
        server_name yourdomain.com www.yourdomain.com;

        # Redirect all HTTP requests to HTTPS
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        server_name yourdomain.com www.yourdomain.com;

        ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

        root /var/www/html;
        index index.html index.htm;

        location / {
            try_files $uri $uri/ =404;
        }
    }
    ```
3. **Save the file and exit**.

### Step 5: Obtain and Configure SSL Certificate with Certbot

1. **Install Certbot and the Nginx plugin**:
    ```sh
    sudo apt update
    sudo apt install certbot python3-certbot-nginx
    ```
2. **Run Certbot to obtain and configure the SSL certificate**:
    ```sh
    sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
    ```

### Step 6: Verify and Test Configuration

1. **Test the Nginx configuration**:
    ```sh
    sudo nginx -t
    ```
2. **Reload Nginx**:
    ```sh
    sudo systemctl reload nginx
    ```

### Step 7: Verify Access to Your Domain

1. **Open a web browser and go to**:
    - `http://yourdomain.com`
    - `http://www.yourdomain.com`
    - `https://yourdomain.com`
    - `https://www.yourdomain.com`

You should see your default Nginx page (or your custom page) served over both HTTP and HTTPS, with HTTP requests being redirected to HTTPS.

### Step 8: Ensure Certificate Renewal

1. **Simulate certificate renewal** to ensure everything is configured correctly:
    ```sh
    sudo certbot renew --dry-run
    ```

By following these steps, your GoDaddy domain should be correctly mapped to your Google Cloud instance, and your website should be accessible over both HTTP and HTTPS. If you encounter any issues, please provide specific error messages or details about the problem.

## Debian 

Debian is way cheaper for some reason on Google cloud.
To set up five React websites on Google Cloud using Debian (which is often cheaper), each with production and development subdomains, and PostgreSQL + pgAdmin, follow these steps. This guide will cover the creation of the required infrastructure, users, and permissions.

### 1. Set Up Google Cloud Environment
1. **Create a Google Cloud Account**: If you don’t have one, sign up at [Google Cloud](https://cloud.google.com/).
2. **Create a New Project**: Go to the Google Cloud Console and create a new project for your websites.
3. **Enable Billing**: Ensure billing is enabled for your project.

### 2. Set Up Compute Engine Instances
1. **Create VM Instances**:
    - Go to the Compute Engine section and create a new VM instance.
    - Choose Debian 10 as the operating system.
    - Create five instances, naming them `www1`, `www2`, etc.
2. **Configure Firewall Rules**:
    - Allow HTTP, HTTPS, and SSH traffic.
    
### 3. Set Up PostgreSQL and pgAdmin
1. **SSH into Each Instance**:
    - Use the Google Cloud Console to SSH into each VM instance.
2. **Install PostgreSQL and pgAdmin**:
    ```sh
    sudo apt update
    sudo apt install postgresql postgresql-contrib
    sudo apt install pgadmin4
    ```
3. **Configure PostgreSQL**:
    - Set up a PostgreSQL user and database.
    ```sh
    sudo -i -u postgres
    psql
    CREATE USER yourusername WITH PASSWORD 'yourpassword';
    CREATE DATABASE yourdatabase;
    GRANT ALL PRIVILEGES ON DATABASE yourdatabase TO yourusername;
    \q
    exit
    ```

### 4. Configure React Apps
1. **Set Up Node.js and NPM**:
    ```sh
    curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```
2. **Deploy React Apps**:
    - SCP your React app files to each server.
    - SSH into each instance and navigate to your app directory.
    ```sh
    npm install
    npm run build
    ```

### 5. Set Up Nginx for HTTPS and Subdomains
1. **Install Nginx**:
    ```sh
    sudo apt install nginx
    ```
2. **Configure Nginx**:
    - Create configuration files for each subdomain in `/etc/nginx/sites-available/`.
    - Link the configurations to `/etc/nginx/sites-enabled/`.
    ```sh
    sudo ln -s /etc/nginx/sites-available/www1.conf /etc/nginx/sites-enabled/
    ```
    - Example Nginx configuration (`/etc/nginx/sites-available/www1.conf`):
    ```nginx
    server {
        listen 80;
        server_name www1.com dev.www1.com;

        location / {
            root /var/www/www1/build;
            index index.html index.htm;
            try_files $uri $uri/ /index.html;
        }
    }
    ```
3. **Enable HTTPS with Certbot**:
    ```sh
    sudo apt install certbot python3-certbot-nginx
    sudo certbot --nginx -d www1.com -d dev.www1.com
    ```
4. **Reload Nginx**:
    ```sh
    sudo systemctl reload nginx
    ```

### 6. User and Permissions Setup
1. **Add Users**:
    ```sh
    sudo adduser megha
    sudo adduser bear
    sudo adduser devs
    ```
2. **Add Users to Sudoers**:
    ```sh
    sudo usermod -aG sudo megha
    sudo usermod -aG sudo bear
    ```
3. **Set Permissions**:
    ```sh
    sudo chown -R devs:devs /var/www
    sudo chmod -R 775 /var/www
    ```

### 7. Set Up SSH and SCP Access
1. **Configure SSH**:
    - Add SSH keys for users `megha`, `bear`, and `devs`.
    - Copy the public keys to `/home/username/.ssh/authorized_keys`.
2. **Ensure SSH Service is Running**:
    ```sh
    sudo systemctl restart ssh
    ```

### Final Steps
- Test your websites: `https://www1.com` and `https://dev.www1.com`.
- Verify PostgreSQL and pgAdmin setup.
- Ensure all users can SSH and SCP into the server as expected.

This setup provides a robust environment for developing and deploying React applications with PostgreSQL databases on Google Cloud using Debian. Adjust configurations as necessary for your specific use cases and security requirements.




## Ubuntu
To set up five React websites on Google Cloud, each with production and development subdomains, and PostgreSQL + pgAdmin on Ubuntu Server 20.04, follow these steps. This guide will cover the creation of the required infrastructure, users, and permissions.

### 1. Set Up Google Cloud Environment
1. **Create a Google Cloud Account**: If you don’t have one, sign up at [Google Cloud](https://cloud.google.com/).
2. **Create a New Project**: Go to the Google Cloud Console and create a new project for your websites.
3. **Enable Billing**: Ensure billing is enabled for your project.

### 2. Set Up Compute Engine Instances
1. **Create VM Instances**:
    - Go to the Compute Engine section and create a new VM instance.
    - Choose Ubuntu Server 20.04 as the operating system.
    - Create five instances, naming them `www1`, `www2`, etc.
2. **Configure Firewall Rules**:
    - Allow HTTP, HTTPS, and SSH traffic.
    
### 3. Set Up PostgreSQL and pgAdmin
1. **SSH into Each Instance**:
    - Use the Google Cloud Console to SSH into each VM instance.
2. **Install PostgreSQL and pgAdmin**:
    ```sh
    sudo apt update
    sudo apt install postgresql postgresql-contrib
    sudo apt install pgadmin4
    ```
3. **Configure PostgreSQL**:
    - Set up a PostgreSQL user and database.
    ```sh
    sudo -i -u postgres
    psql
    CREATE USER yourusername WITH PASSWORD 'yourpassword';
    CREATE DATABASE yourdatabase;
    GRANT ALL PRIVILEGES ON DATABASE yourdatabase TO yourusername;
    \q
    exit
    ```

### 4. Configure React Apps
1. **Set Up Node.js and NPM**:
    ```sh
    curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```
2. **Deploy React Apps**:
    - SCP your React app files to each server.
    - SSH into each instance and navigate to your app directory.
    ```sh
    npm install
    npm run build
    ```

### 5. Set Up Nginx for HTTPS and Subdomains
1. **Install Nginx**:
    ```sh
    sudo apt install nginx
    ```
2. **Configure Nginx**:
    - Create configuration files for each subdomain in `/etc/nginx/sites-available/`.
    - Link the configurations to `/etc/nginx/sites-enabled/`.
    ```sh
    sudo ln -s /etc/nginx/sites-available/www1.conf /etc/nginx/sites-enabled/
    ```
    - Example Nginx configuration (`/etc/nginx/sites-available/www1.conf`):
    ```nginx
    server {
        listen 80;
        server_name www1.com dev.www1.com;

        location / {
            root /var/www/www1/build;
            index index.html index.htm;
            try_files $uri $uri/ /index.html;
        }
    }
    ```
3. **Enable HTTPS with Certbot**:
    ```sh
    sudo apt install certbot python3-certbot-nginx
    sudo certbot --nginx -d www1.com -d dev.www1.com
    ```
4. **Reload Nginx**:
    ```sh
    sudo systemctl reload nginx
    ```

### 6. User and Permissions Setup
1. **Add Users**:
    ```sh
    sudo adduser megha
    sudo adduser bear
    sudo adduser devs
    ```
2. **Add Users to Sudoers**:
    ```sh
    sudo usermod -aG sudo megha
    sudo usermod -aG sudo bear
    ```
3. **Set Permissions**:
    ```sh
    sudo chown -R devs:devs /var/www
    sudo chmod -R 775 /var/www
    ```

### 7. Set Up SSH and SCP Access
1. **Configure SSH**:
    - Add SSH keys for users `megha`, `bear`, and `devs`.
    - Copy the public keys to `/home/username/.ssh/authorized_keys`.
2. **Ensure SSH Service is Running**:
    ```sh
    sudo systemctl restart ssh
    ```

### Final Steps
- Test your websites: `https://www1.com` and `https://dev.www1.com`.
- Verify PostgreSQL and pgAdmin setup.
- Ensure all users can SSH and SCP into the server as expected.

This setup provides a robust environment for developing and deploying React applications with PostgreSQL databases on Google Cloud. Adjust configurations as necessary for your specific use cases and security requirements.



Yes, you can use `rsync` to copy local web files from your machine to a particular directory under `/var/www/html` on your server. Here’s how to set it up:

### Step 1: Install `rsync`

Ensure `rsync` is installed on both your local machine and the server. It’s usually pre-installed on most Linux distributions, but you can install it using the following command if necessary:

**On Debian-based systems (e.g., Ubuntu):**
```sh
sudo apt update
sudo apt install rsync
```

### Step 2: Set Up SSH Access

To use `rsync` over SSH, you need to have SSH access set up between your local machine and the server. Ensure you can SSH into your server without a password by setting up SSH key-based authentication.

**Generate SSH Key on Local Machine:**
```sh
ssh-keygen
```

**Copy SSH Key to Server:**
```sh
ssh-copy-id user@server_ip
```

Replace `user` with your server's username and `server_ip` with your server's IP address.

### Step 3: Use `rsync` to Copy Files

You can now use `rsync` to copy files from your local machine to the server. Here is a basic example:

```sh
rsync -avz /path/to/local/webfiles/ user@server_ip:/var/www/html/yourwebsite
```

- `-a`: Archive mode (preserves permissions, symbolic links, etc.)
- `-v`: Verbose mode (provides detailed output)
- `-z`: Compress file data during the transfer

Replace `/path/to/local/webfiles/` with the path to your local web files, `user` with your server's username, `server_ip` with your server's IP address, and `/var/www/html/yourwebsite` with the target directory on your server.

### Step 4: Automate the Process with a Script

To automate the process, you can create a shell script:

```sh
#!/bin/bash

# Variables
LOCAL_DIR="/path/to/local/webfiles/"
REMOTE_USER="user"
REMOTE_HOST="server_ip"
REMOTE_DIR="/var/www/html/yourwebsite/"

# Rsync command
rsync -avz $LOCAL_DIR $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR
```

### Step 5: Make the Script Executable

Save the script, for example, as `sync_webfiles.sh`, and make it executable:

```sh
chmod +x sync_webfiles.sh
```

### Step 6: Set Up a Cron Job for Regular Syncing

To run the script at regular intervals, set up a cron job:

1. Open the cron table for editing:
   ```sh
   crontab -e
   ```

2. Add a line to run the script at your desired interval. For example, to run it every hour:
   ```sh
   0 * * * * /path/to/sync_webfiles.sh
   ```

Replace `/path/to/sync_webfiles.sh` with the full path to your script.

### Summary

Using `rsync` along with SSH key-based authentication allows you to efficiently and securely copy local web files to a specific directory under `/var/www/html` on your server. Automating this process with a shell script and a cron job ensures your web files are kept up-to-date with minimal manual intervention.

It's a good idea to keep your scripts organized in a dedicated directory. However, it's not typical to place scripts under `/var/www`, as this directory is usually reserved for web content. Instead, it's better to place your scripts in a directory specifically meant for executable files and scripts.

A common and recommended location is `/usr/local/bin` for user-installed scripts that need to be accessible system-wide. This directory is usually included in the system's `PATH`, making it easy to run the scripts from anywhere in the terminal.

Here's how you can set it up:

### Step 1: Create a Scripts Directory

Create a directory for your useful scripts:

```sh
sudo mkdir -p /usr/local/bin/useful-scripts
```

### Step 2: Move Your Script to the Directory

Move your script to the newly created directory and make sure it is executable:

```sh
sudo mv setup_website.sh /usr/local/bin/useful-scripts/
sudo chmod +x /usr/local/bin/useful-scripts/setup_website.sh
```

### Step 3: Optional: Add the Directory to PATH

Although `/usr/local/bin` is typically in the `PATH` by default, if you want to ensure that the `useful-scripts` directory is included, you can add it to your `PATH`. Open or create a `~/.bashrc` or `~/.profile` file and add the following line:

```sh
export PATH=$PATH:/usr/local/bin/useful-scripts
```

Then, reload the profile to apply the changes:

```sh
source ~/.bashrc
```

### Accessing Your Scripts

Now, you can run your script from anywhere by simply typing:

```sh
setup_website.sh
```

or if you prefer to use the full path:

```sh
/usr/local/bin/useful-scripts/setup_website.sh
```

### Summary

Placing your scripts in `/usr/local/bin/useful-scripts` keeps them organized and accessible, without cluttering your web content directories under `/var/www`. This approach maintains a clean separation between your web files and executable scripts, adhering to best practices for system organization.

