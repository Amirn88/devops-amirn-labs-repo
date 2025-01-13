# SonarQube Installation, Configuration, and GitLab CI Integration Guide

This guide provides a comprehensive approach to deploying, configuring, and integrating SonarQube in an on-premises environment. It covers:

1. **Deployment Methods**:
   - **Using Docker Compose** – A quick and efficient containerized setup.
   - **Manual Deployment with OpenJDK and PostgreSQL** – A more customizable approach, ideal for air-gapped or non-containerized environments.

2. **Authentication**:
   - Enabling GitLab authentication to streamline user management.

3. **CI/CD Integration**:
   - Integrating SonarQube with GitLab CI/CD pipelines for automated code quality analysis.

---

## Deployment Methods

### 1. Using Docker Compose

This method uses Docker Compose to set up and run SonarQube and PostgreSQL as containers. It is a fast and convenient approach for environments that support containerization.

### 2. Manual Deployment with OpenJDK and PostgreSQL

For environments that are air-gapped or do not support containerization, this method provides step-by-step instructions to manually install and configure SonarQube with OpenJDK and PostgreSQL.

---

---

## Prerequisites

Before proceeding with the installation, ensure the following:
- **For Docker Compose Method**:
  - Docker and Docker Compose installed.
- **For Manual Method**:
  - OpenJDK 17 installed.
  - PostgreSQL installed and configured.
- Sufficient system resources (CPU, RAM, and storage).

---

## General Notes for Air-Gapped Environments

For air-gapped environments:
1. **Prepare the required images**:
   - Download the Docker images (SonarQube and PostgreSQL) on a connected machine.
2. **Push the images to your private registry**:
   - Retag and push the images to your private registry.

Example commands:
```bash
# Pull images
docker pull sonarqube:latest
docker pull postgres:latest

# Retag images
docker tag sonarqube:latest <your-private-registry>/sonarqube:latest
docker tag postgres:latest <your-private-registry>/postgres:latest

# Push images to private registry
docker push <your-private-registry>/sonarqube:latest
docker push <your-private-registry>/postgres:latest
```
Replace <your-private-registry> with the URL of your private registry.

## Method 1: Deploy SonarQube Using Docker Compose

This method provides a quick and containerized setup for deploying SonarQube. It is ideal for environments that support Docker and Docker Compose.

### Step 1: Create a Dedicated User and Group

To isolate SonarQube processes, create a dedicated system user and group:

```bash
sudo groupadd sonarqube
sudo useradd -r -g sonarqube sonarqube
```
### Step 2: Prepare Data Directories

SonarQube requires specific directories to store its data, logs, and extensions. These directories ensure that data is persisted even if the container is restarted or recreated. Follow these steps to create and configure the directories:

1. Create the necessary directories:
   ```bash
   sudo mkdir -p /sonarqube-data/{data,extensions,logs,postgresql,postgresql_data}
   ```
2. Assign ownership of the directories to the `sonarqube` user and group to ensure proper access rights:
``` 
   sudo chown -R sonarqube:sonarqube /sonarqube-data
```
3. Set permissions to ensure that SonarQube can access and write to the directories:
```
sudo chmod -R 777 /sonarqube-data
```
### Step 3: Create a `docker-compose.yml` File

The `docker-compose.yml` file defines the configuration for the SonarQube and PostgreSQL services, including how they interact. Follow these steps to create and configure the file:

1. Create a new file named `docker-compose.yml` in your working directory:
   ```bash
   nano docker-compose.yml
   ```
Add the following configuration:
```
version: "3"
services:
  sonarqube:
    image: <your-private-registry>/sonarqube:latest
    depends_on:
      - db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: <your-database-username>
      SONAR_JDBC_PASSWORD: <your-database-password>
    volumes:
      - /sonarqube-data/data:/opt/sonarqube/data
      - /sonarqube-data/extensions:/opt/sonarqube/extensions
      - /sonarqube-data/logs:/opt/sonarqube/logs
    ports:
      - "9000:9000"

  db:
    image: <your-private-registry>/postgres:latest
    environment:
      POSTGRES_USER: <your-database-username>
      POSTGRES_PASSWORD: <your-database-password>
    volumes:
      - /sonarqube-data/postgresql:/var/lib/postgresql
      - /sonarqube-data/postgresql_data:/var/lib/postgresql/data
```
3. Save and close the file.

Replace <your-private-registry>, <your-database-username>, and <your-database-password> with the actual values for your environment.

### Step 4: Configure Virtual Memory

SonarQube relies on Elasticsearch, which requires specific virtual memory settings for proper operation. Follow these steps to configure your system:

1. Temporarily set the virtual memory limit:
   ```bash
   sudo sysctl -w vm.max_map_count=262144
   ```
2. Make the change permanent by adding it to the system configuration file:
```
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```
3. Apply the changes to take effect:
```
sudo sysctl -p
```

### Step 5: Deploy SonarQube

After completing the configuration, you can deploy the SonarQube and PostgreSQL services using Docker Compose. Follow these steps:

1. Start the services in detached mode:
   ```bash
   docker compose up -d
   ```
2. Verify that the services are running:
```
docker compose ps
```
### Step 6: Access SonarQube

Once the services are running, you can access the SonarQube interface by following these steps:

1. Open a web browser and navigate to:
```
http://<your-server-ip>:9000
```
Replace <your-server-ip> with the IP address or hostname of the server hosting SonarQube.

2. Log in using the default credentials:
   - **Username**: `admin`
   - **Password**: `admin`

3. Upon logging in for the first time, you will be prompted to change the default admin password. Choose a strong and secure password for enhanced security.

## Method 2: Deploy SonarQube Manually with OpenJDK and PostgreSQL

### Step 1: Install OpenJDK

SonarQube requires Java to run. Install OpenJDK 17 and configure the environment by following these steps:

1. Install OpenJDK 17 and its development tools:
   ```bash
   sudo dnf -y install java-17-openjdk java-17-openjdk-devel
   ```
2. Configure the Java environment by creating a profile script:
```
cat > /etc/profile.d/java.sh <<'EOF'
export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which java)))))
export PATH=$PATH:$JAVA_HOME/bin
EOF
```
3. Load the Java environment configuration:
```
source /etc/profile.d/java.sh
```
4. Verify the Java installation and version:
```
java --version
```
### Step 2: Install and Configure PostgreSQL

SonarQube requires a PostgreSQL database for storing its data. Follow these steps to install and initialize PostgreSQL:

1. Install PostgreSQL and its necessary extensions:
   ```bash
   sudo dnf install postgresql-server postgresql-contrib
   ```
2. Initialize the PostgreSQL database:
```
sudo postgresql-setup --initdb
```
3. Start the PostgreSQL service:
```
sudo systemctl start postgresql
```
4. Enable the PostgreSQL service to start automatically on system boot:
```
sudo systemctl enable postgresql
```
### Step 3: Edit the PostgreSQL Configuration File

To allow SonarQube to connect to the PostgreSQL database, update the PostgreSQL configuration file as follows:

1. Open the `pg_hba.conf` file for editing:
   ```bash
   sudo nano /var/lib/pgsql/data/pg_hba.conf
   ```
Update it to allow local and network connections:
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     trust
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
### Step 4: Create a Database and User for SonarQube

SonarQube requires a dedicated database and user. Follow these steps to set them up:

1. Switch to the PostgreSQL user and access the PostgreSQL interactive terminal:
   ```bash
   sudo -u postgres psql
   ```
2. Create a new user for SonarQube with a secure password:
```
CREATE ROLE <your-database-username> WITH LOGIN ENCRYPTED PASSWORD '<your-database-password>';
```
3. Create a database for SonarQube:
```
CREATE DATABASE sonarqube;
```
4. Grant all privileges on the SonarQube database to the newly created user:
```
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO <your-database-username>;
```
5. Exit the PostgreSQL terminal:
```
\q
```
### Step 5: Install SonarQube

Download and install SonarQube by following these steps:

1. Download the SonarQube archive:
   ```bash
   wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-<version>.zip
   ```
Replace <version> with the desired version of SonarQube.
2. Extract the downloaded archive:
```
unzip sonarqube-<version>.zip
```
3. Move the extracted directory to /opt:
```
sudo mv sonarqube-<version> /opt/sonarqube
```
4. Assign ownership of the SonarQube directory to the sonarqube user and group:
```
sudo chown -R sonarqube:sonarqube /opt/sonarqube
```
 Edit the configuration file:

```
sudo nano /opt/sonarqube/conf/sonar.properties
```
Add:
```
sonar.jdbc.username=<your-database-username>
sonar.jdbc.password=<your-database-password>
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
sonar.web.javaAdditionalOpts=-server
sonar.web.host=127.0.0.1
```
### Step 6: Configure System Settings for Elasticsearch

SonarQube relies on Elasticsearch, which requires specific system configurations for optimal performance. Follow these steps to update your system settings:

1. Open the system configuration file:
   ```bash
   sudo nano /etc/sysctl.conf
   ```
2. Add the following lines to configure virtual memory and file limits:
```
vm.max_map_count=524288
fs.file-max=131072
```
3. Apply the changes to make them effective:
```
sudo sysctl -p
```
4. Create a new limits configuration file for the sonarqube user:
```
sudo nano /etc/security/limits.d/99-sonarqube.conf
```
5. Add the following lines to set file and process limits:
```
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```
6. Reboot your system to apply all changes:
```
sudo reboot
```
### Step 7: Set Up SonarQube as a Service

To manage SonarQube efficiently, set it up as a systemd service. Follow these steps:

1. Create a new service file for SonarQube:
   ```bash
   sudo nano /etc/systemd/system/sonarqube.service
   ```
2. Add the following content to define the service:
```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonarqube
Group=sonarqube
Restart=always
LimitNOFILE=131072
LimitNPROC=8192
TimeoutStartSec=5

[Install]
WantedBy=multi-user.target
```
3. Reload systemd to recognize the new service:
```
sudo systemctl daemon-reload
```
4. Start the SonarQube service:
```
sudo systemctl start sonarqube
```
5. Enable the service to start automatically on system boot:
```
sudo systemctl enable sonarqube
```
6. Check the service status to ensure it's running:
```
sudo systemctl status sonarqube
```
### Step 8: Access SonarQube

Once the services are running, you can access the SonarQube interface by following these steps:

1. Open a web browser and navigate to:
```
http://<your-server-ip>:9000
```
Replace <your-server-ip> with the IP address or hostname of the server hosting SonarQube.

2. Log in using the default credentials:
   - **Username**: `admin`
   - **Password**: `admin`

3. Upon logging in for the first time, you will be prompted to change the default admin password. Choose a strong and secure password for enhanced security.


### Enable GitLab Authentication

To allow users to authenticate to SonarQube using their GitLab accounts, follow these steps:

---

#### Step 1: Create an OAuth Application in GitLab

1. Log in to your GitLab instance.
2. Navigate to **Settings > Applications**.
3. Click on **New Application** and configure the following:
   - **Name**: `SonarQube`
   - **Redirect URI**: `https://<your-sonarqube-url>/oauth2/callback/gitlab`
     Replace `<your-sonarqube-url>` with the actual URL of your SonarQube instance.
   - **Scopes**: Select the checkbox for `read_user`.

4. Save the application. Note the generated **Application ID** and **Secret** as they will be required in the next step.

---

#### Step 2: Configure SonarQube Authentication

1. Log in to your SonarQube instance as an admin.
2. Navigate to **Administration > Security > Authentication**.
3. Enable GitLab as an authentication provider.
4. Enter the following details:
   - **GitLab URL**: `https://<your-gitlab-url>`  
     Replace `<your-gitlab-url>` with the URL of your GitLab instance.
   - **Application ID**: Use the Application ID generated in Step 1.
   - **Secret**: Use the Secret generated in Step 1.

5. Save the configuration.

---

#### Step 3: Set the Server Base URL

1. Go to **Administration > Configuration > General**.
2. Set the **Server Base URL** to match your SonarQube instance's URL:


Replace `<your-sonarqube-url>` with the actual URL of your SonarQube deployment.

---

### Verify GitLab Authentication Integration

To ensure the integration with GitLab is working correctly:

1. Log out of your SonarQube instance.
2. On the login page, select the **Sign in with GitLab** option.
3. Confirm that GitLab users can successfully authenticate and access SonarQube.

By completing these steps, GitLab authentication will be fully integrated and functional for your SonarQube instance.

## Integration with GitLab CI for SonarQube Code Quality Checks

SonarQube can be integrated into your GitLab CI/CD pipeline to automatically analyze code quality during the build process. This ensures your code meets quality standards before deployment. Below is a general and descriptive example of configuring a SonarQube check in GitLab CI.

---

### GitLab CI Job Configuration

Add the following job to your `.gitlab-ci.yml` file:

```yaml
Sonarqube Check:
  stage: code_quality
  image: <your-container-registry>/sonar-scanner-cli:latest
  needs: []
  script:
    - curl -o sonar-project.properties "<your-gitlab-repo-url>/raw/$CI_PROJECT_NAME/scripts/sonar-project.properties"
    - sed -i "s|PROJECT_NAME|${CI_PROJECT_NAME}|g" sonar-project.properties
    - sed -i "s|BRANCH_NAME|${CI_COMMIT_REF_NAME}|g" sonar-project.properties
    - sed -i "s|CI_COMMIT_TAG|${CI_COMMIT_TAG}|g" sonar-project.properties
    - cat sonar-project.properties
    - sonar-scanner -Dsonar.login=<your-sonarqube-token> -X
  tags:
    - RUNNER_TAG

```

### Sample `sonar-project.properties`

Below is a general template for the `sonar-project.properties` file, used to configure SonarQube analysis for Java-based projects:

```properties
sonar.projectName=PROJECT_NAME
sonar.projectKey=PROJECT_NAME
sonar.projectVersion=CI_COMMIT_TAG
sonar.sources=.
sonar.java.binaries=.
sonar.host.url=https://<your-sonarqube-url>
#sonar.branch.name=BRANCH_NAME
sonar.login=YOUR_TOKEN
```

