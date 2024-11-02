
# Simple docker app without DNS/https for prototyping 
1. First, make sure your server's firewall allows incoming traffic on port 5000:

```bash
# Check if ufw is installed
sudo apt-get install ufw
# Allow SSH (important to do this first to not lock yourself out)
sudo ufw allow ssh
# Allow port 5000
sudo ufw allow 5000
# Enable the firewall
sudo ufw enable
# Check status
sudo ufw status
```

2. Then you can run your Docker container:
```bash
# Build the image
docker build -t docker-app .
# Run the container with port mapping
docker run -d -p 5000:5000 docker-app
```

3. You should now be able to access your app using:
```
http://<IP:5000
```

# With DNS and For better security and production deployment, I recommend:

1. Using HTTPS with a domain name:
```bash
# Install Nginx
sudo apt-get install nginx
# Install Certbot for SSL
sudo apt-get install certbot python3-certbot-nginx
```

2. Create an Nginx configuration file:

```nginx
# /etc/nginx/sites-available/docker-app

server {
    server_name your_domain.com;  # Replace with your domain

    location / {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 80;
}

```

3. Enable the site and get SSL certificate:
```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/docker-app /etc/nginx/sites-enabled/
# Test nginx config
sudo nginx -t
# If test is successful, reload nginx
sudo systemctl reload nginx
# Get SSL certificate (if you have a domain)
sudo certbot --nginx -d your_domain.com
```

4. Update your Docker run command for production:

```yaml
version: '3.8'

services:
  docker-app:
    build: .
    restart: always
    ports:
      - "127.0.0.1:5000:5000"  # Only allow local access, Nginx will handle external
    environment:
      - FLASK_ENV=production
      - FLASK_APP=app.py

```

Security recommendations:
1. Don't expose port 5000 directly to the internet; use Nginx as a reverse proxy
2. Set up SSL/TLS encryption
3. Consider adding authentication
4. Use environment variables for sensitive configurations
5. Regularly update your system and dependencies

Basic setup steps summary:
1. SSH into your Hetzner server
2. Install Docker if not already installed:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

3. Clone your repository:
```bash
git clone your-repo-url
cd your-repo-directory
```

4. Build and run:
```bash
docker build -t docker-app .
docker run -d -p 5000:5000 docker-app
```

5. Access using:
```
http://YOUR_HETZNER_IP:5000
```

Quick test to make sure it's working:
```bash
# Test locally on the server
curl localhost:5000

# Test from your local machine
curl http://YOUR_HETZNER_IP:5000
```