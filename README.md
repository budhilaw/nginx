# Portfolio Nginx Reverse Proxy

This repository contains the nginx configuration for the portfolio website infrastructure. It acts as a reverse proxy and load balancer for the frontend and backend services.

## Architecture

```
Internet → Nginx (Port 80/443) → {
  /api/* → Backend Service (portfolio-backend:8000)
  /*     → Frontend Service (portfolio-frontend:80)
}
```

## Features

- **Reverse Proxy**: Routes API requests to backend and static requests to frontend
- **Rate Limiting**: Protects against abuse with configurable limits
- **Security Headers**: Implements security best practices
- **Cloudflare Integration**: Configured for Cloudflare CDN
- **Health Checks**: Built-in health monitoring
- **Gzip Compression**: Optimizes response sizes

## Configuration Files

- `nginx.conf` - Main nginx configuration
- `default.conf` - Server block configuration with routing rules
- `Dockerfile` - Container build configuration
- `docker-compose.yml` - Local development setup

## Rate Limiting

- **General**: 30 requests/second with burst of 50
- **API**: 10 requests/second with burst of 20  
- **Login**: 5 requests/minute with burst of 5

## Security Features

- **Host Header Validation**: Blocks requests with invalid hosts
- **Cloudflare IP Whitelisting**: Only accepts traffic from Cloudflare
- **Security Headers**: X-Frame-Options, X-XSS-Protection, etc.
- **Hidden File Protection**: Blocks access to dotfiles and backups

## Local Development

```bash
# Build and run locally
docker-compose up --build

# Or build manually
docker build -t nginx .
docker run -d --name nginx -p 80:80 --network portfolio-network nginx
```

## Production Deployment

The repository includes GitHub Actions for automated deployment:

1. **Build**: Creates Docker image and pushes to GitHub Container Registry
2. **Deploy**: SSH to production server and updates running container

### Required Secrets

Set these in GitHub repository secrets:

- `HOST` - Production server IP/hostname
- `USERNAME` - SSH username
- `SSH_KEY` - Private SSH key for deployment

## Network Requirements

The nginx container must be on the `portfolio-network` Docker network to communicate with:

- `portfolio-frontend` container (port 80)
- `portfolio-backend` container (port 8000)

## Health Check

The nginx container includes a health check endpoint:

```bash
curl http://localhost/health
# Returns: healthy
```

## Logs

Nginx logs are stored in a Docker volume:

```bash
# View access logs
docker exec nginx tail -f /var/log/nginx/access.log

# View error logs  
docker exec nginx tail -f /var/log/nginx/error.log
```

## Configuration Updates

To update the configuration:

1. Modify `nginx.conf` or `default.conf`
2. Commit and push to trigger CI/CD
3. The new container will be automatically deployed

## Troubleshooting

### Common Issues

1. **502 Bad Gateway**: Backend/frontend containers not reachable
   ```bash
   # Check if services are running
   docker ps
   # Check network connectivity
   docker exec nginx nslookup portfolio-backend
   ```

2. **Rate Limiting**: Requests being blocked
   ```bash
   # Check nginx error logs
   docker logs nginx
   ```

3. **SSL/TLS Issues**: Certificate problems
   ```bash
   # Verify certificate files
   docker exec nginx ls -la /etc/ssl/certs/
   ```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes and test locally
4. Submit a pull request

## License

This configuration is part of the portfolio infrastructure. 