#!/bin/bash
set -e

echo "=== Updating system ==="
sudo apt update && sudo apt upgrade -y

echo "=== Installing Java and curl ==="
sudo apt install -y default-jdk curl

echo "=== Preparing Tomcat installation ==="
TOMCAT_VERSION=10.1.30
TOMCAT_TAR=apache-tomcat-${TOMCAT_VERSION}.tar.gz
TOMCAT_URL=https://archive.apache.org/dist/tomcat/tomcat-10/v${TOMCAT_VERSION}/bin/${TOMCAT_TAR}

# Create Tomcat user if not exists
if ! id "tomcat" &>/dev/null; then
    sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat
fi

echo "=== Downloading Tomcat ==="
cd /tmp
curl -fO ${TOMCAT_URL}

echo "=== Installing Tomcat ==="
sudo mkdir -p /opt/tomcat
sudo tar xzf ${TOMCAT_TAR} -C /opt/tomcat --strip-components=1
sudo chown -R tomcat: /opt/tomcat

echo "=== Creating systemd service for Tomcat ==="
sudo tee /etc/systemd/system/tomcat.service > /dev/null <<EOL
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

Restart=on-failure

[Install]
WantedBy=multi-user.target
EOL

echo "=== Starting Tomcat ==="
sudo systemctl daemon-reload
sudo systemctl enable tomcat
sudo systemctl start tomcat

echo "=== Tomcat Installed and Running ==="
echo "Access it at: http://<your-ec2-public-ip>:8080"
