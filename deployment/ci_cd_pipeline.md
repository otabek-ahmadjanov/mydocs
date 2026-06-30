# ⚙️ CI/CD Pipeline Setup for Spring Boot Project

## 🔐 Step 1: Configure SSH Connection Without Password

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ci_cd_key

# Copy public key to server
ssh-copy-id -i ~/.ssh/ci_cd_key.pub user@ip_address

# Configure user rights on the server
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

# Test connection
ssh -i ~/.ssh/ci_cd_key user@ip_address
```

📌 **Note:** Copy the **private key**:

```bash
cat ~/.ssh/ci_cd_key
```

Add it to your **GitHub repository secrets** as:

```
SSH_PRIVATE_KEY
```

---

## 🚀 Step 2: Create GitHub Actions Workflow

Create the file: `.github/workflows/cicd.yml`

### ✨ Example `cicd.yml`

```yaml
name: Build & Deploy

on:
  push:
    branches:
      - master

jobs:
  build-deploy:
    name: Build and Deploy project
    runs-on: ubuntu-22.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup JDK 21
        uses: actions/setup-java@v3
        with:
          distribution: 'corretto'
          java-version: 21

      - name: Build the app
        run: |
          mvn clean
          mvn -B package -DskipTests --file pom.xml

      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/ci_cd_key
          chmod 600 ~/.ssh/ci_cd_key
          ssh-keyscan -H ${{ secrets.SERVER_IP }} >> ~/.ssh/known_hosts

      - name: Copy JAR to server
        run: |
          scp -i ~/.ssh/ci_cd_key target/maydongo.jar ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_IP }}:~/maydon/maydongo/

      - name: Deploy to server
        run: |
          ssh -i ~/.ssh/ci_cd_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_IP }} << 'EOF'
            cd ~/maydon/maydongo

            # Create .env file
            echo "SPRING_DATASOURCE_URL=${{ secrets.SPRING_DATASOURCE_URL }}" > .env
            echo "SPRING_DATASOURCE_USERNAME=${{ secrets.SPRING_DATASOURCE_USERNAME }}" >> .env
            echo "SPRING_DATASOURCE_PASSWORD=${{ secrets.SPRING_DATASOURCE_PASSWORD }}" >> .env
            echo "JWT_SECRET_KEY=${{ secrets.JWT_SECRET_KEY }}" >> .env
            echo "AWS_SECRET_KEY=${{ secrets.AWS_SECRET_KEY }}" >> .env
            echo "AWS_SECRET_ACCESS_KEY=${{ secrets.AWS_SECRET_ACCESS_KEY }}" >> .env
            echo "AWS_REGION=${{ secrets.AWS_REGION }}" >> .env
            echo "AWS_S3_BUCKET_NAME=${{ secrets.AWS_S3_BUCKET_NAME }}" >> .env

            chmod u+x maydongo.jar
            sudo systemctl restart maydongo.service || echo "Failed to restart service"
          EOF
```

---

## 🛠 Step 3: Configure `systemd` Service on Server

Create the service file:

```bash
sudo nano /etc/systemd/system/maydongo.service
```

### 🧾 Example `maydongo.service`

```ini
[Unit]
Description=Maydon Spring Boot App
After=network.target

[Service]
User=riat
WorkingDirectory=/home/riat/maydon/maydongo
EnvironmentFile=/home/riat/maydon/maydongo/.env
ExecStart=/usr/bin/java -Xms512m -Xmx1024m -jar /home/riat/maydon/maydongo/maydongo.jar
SuccessExitStatus=143
Restart=always
RestartSec=10
StandardOutput=append:/home/riat/maydon/maydongo/maydongo.log
StandardError=append:/home/riat/maydon/maydongo/maydongo.log

[Install]
WantedBy=multi-user.target
```

---

### 🔄 Enable & Manage the Service

```bash
# Reload systemd
sudo systemctl daemon-reload
```

Grant permissions using `visudo`:

```bash
sudo visudo
```

Add the line:

```
riat ALL=(ALL) NOPASSWD: /bin/systemctl restart maydongo.service, /bin/systemctl status maydongo.service
```

Enable the service at boot:

```bash
sudo systemctl enable maydongo.service
```

---

## 🧪 Useful Commands

```bash
# View application logs
cat /home/riat/maydon/maydongo/maydongo.log

# View service logs
journalctl -u maydongo.service -b

# Check service status
sudo systemctl status maydongo.service

# Restart service manually
sudo systemctl restart maydongo.service

# Start/Stop service
sudo systemctl start maydongo.service
sudo systemctl stop maydongo.service
```

---

✅ **CI/CD pipeline is ready!**  
Push to the `master` branch — and GitHub Actions will handle build & deployment automatically! 🚀
