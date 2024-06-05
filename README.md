#  React websites on Google Cloud

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
