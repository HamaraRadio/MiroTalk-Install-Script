# MiroTalk-Install-Script#!/bin/bash

set -e

echo "======================================"
echo "MiroTalk Auto Installer (Ubuntu 24.04)"
echo "======================================"

# Update system
echo "Updating system..."
sudo apt update -y
sudo apt upgrade -y

# Install dependencies
echo "Installing dependencies..."
sudo apt install -y git curl build-essential ufw

# Install Node.js LTS
echo "Installing Node.js LTS..."
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

echo "Node Version:"
node -v
npm -v

# Install PM2
echo "Installing PM2..."
sudo npm install -g pm2

# Install MiroTalk
echo "Installing MiroTalk..."
sudo mkdir -p /opt
cd /opt

if [ ! -d "mirotalk" ]; then
    sudo git clone https://github.com/miroslavpejic85/mirotalk.git
fi

cd mirotalk

echo "Installing MiroTalk dependencies..."
npm install

# Configure firewall
echo "Configuring firewall..."
sudo ufw allow 3000/tcp
sudo ufw allow 3478/tcp
sudo ufw allow 3478/udp
sudo ufw --force enable

# Start with PM2 using npm start (fix for server.js error)
echo "Starting MiroTalk..."
pm2 start npm --name "mirotalk" -- start

# Enable startup
pm2 save
pm2 startup systemd -u $USER --hp $HOME

IP=$(hostname -I | awk '{print $1}')

echo ""
echo "======================================"
echo "MiroTalk Installation Completed"
echo "======================================"
echo ""
echo "Open in your browser:"
echo ""
echo "http://$IP:3000"
echo ""
echo "Create meeting:"
echo "http://$IP:3000/newcall"
echo ""
echo "PM2 Commands:"
echo "pm2 status"
echo "pm2 logs mirotalk"
echo "pm2 restart mirotalk"
echo ""
