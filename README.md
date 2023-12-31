# Guacamole Container Deployment with Nginx Reverse Proxy with 2FA protection

- This repository contains Docker configuration files for setting up Apache Guacamole, a clientless remote desktop gateway, using Docker. It includes an Nginx reverse proxy to make Guacamole accessible directly at the server's root URL (`http://IP` or `http://fqdn` or `http://127.0.0.1`). 
- The configuration uses the latest versions of Guacamole and MySQL.

## Components
- **Guacamole Server (`guacd`)**: The core server component of Guacamole.
- **Guacamole Client**: The web application users interact with, served via a custom-built Docker image.
- **MySQL Database (`db`)**: Stores user accounts and connection data.
- **Nginx**: Configured as a reverse proxy to allow access to Guacamole directly from the root URL.

## Initial Setup and Configuration

### Prerequisites
- Docker and Docker Compose installed on your system.
- Basic understanding of Docker and Docker Compose.

### Steps for First-Time Setup
1. **Clone the Repository**:
    ```
    git clone [Your Repository URL]
    cd [Your Repository Name]
    ```

2. **Initial Run**:

   After running.
    ```
    docker-compose up
    ```

   You might encounter an error while accessing to url. Due to the lack of the necessary database schema and user.

   To resolve this, proceed with the following steps.

4. **Download Guacamole SQL Files**:
- Visit the [Guacamole release page](https://guacamole.apache.org/releases/) and download the `guacamole-auth-jdbc-x.x.x.tar.gz` file for the latest version.
- Extract the files:
  ```
  tar -xvf guacamole-auth-jdbc-x.x.x.tar.gz
  ```
- Navigate to the `mysql/schema` directory in the extracted folder.

4. **Copy SQL Scripts to MySQL Container**:

   Replace `<db-container-id>` with the actual container ID of your MySQL container.
    ```  
    docker cp 001-create-schema.sql <db-container-id>:/home/
    docker cp 002-create-admin-user.sql <db-container-id>:/home/
    ```

6. **Execute SQL Scripts**:
- Access the MySQL container:
    ```
    docker exec -it <db-container-id> bash
    ```
- Inside the container, run:
    ```
    dmysql -u root -p$rootpass -D$guacamole_db < /home/001-create-schema.sql
    mysql -u root -p$rootpass -D$guacamole_db < /home/002-create-admin-user.sql
    ```
- Replace `$rootpass` and `$guacamole_db` with your MySQL root password and database name.

6. **Restart MySQL Container**:
After creating the schema and user, restart the MySQL container for the changes to take effect.

## Docker Compose Configuration

The `docker-compose.yml` file defines the services, networks, and volumes for the Guacamole deployment. Here's a brief overview of its contents:

- `guacd`: The Guacamole server daemon.
- `guacamole`: The Guacamole client, accessible at the root URL via Nginx.
- `db`: The MySQL database for Guacamole data.
- `nginx`: Config
## 2FA Authentication

- Two-factor authentication is enabled by default for added security. Users will be prompted to set up 2FA upon their first login.
- If you want to disable it fell free to disable it from the docker-compose.yml file. Remove the below line.
    ```
    TOTP_ENABLED: "true"
    ```

## Nginx Configuration

The `nginx.conf` file sets up Nginx as a reverse proxy. It redirects traffic from the server's root URL to the Guacamole client, allowing direct access without specifying a path.

## Notes

- Ensure all the $vaules (`$rootpass`, `$guacamole_db`, `$user`, `$pass`) in `docker-compose.yml` is chaged according to your environment.
- The default login id and password is `guacadmin`
- For security purposes, avoid using default passwords and usernames.

## Contributions

Contributions to this project are welcome! Please fork the repository and submit a pull request with your improvements.
