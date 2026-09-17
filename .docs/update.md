# Update

## Introduction
This documentation is all about clean up and update the home server.

### Update Server
```bash
sudo apt update && sudo apt upgrade -y
```

### Cleanup & Update Docker
```bash
### Check Docker ###
sudo docker ps -a
sudo docker images
sudo docker volume ls

# Check images
sudo docker system df
#or
sudo docker system df -v

# remove all unused images
sudo docker image prune -a

# remove unused networks
sudo docker network prune

### Update Docker Container ###
# cd into directory and run:
sudo docker compose pull
sudo docker compose up -d --remove-orphans
# after that run one after another:
sudo docker compose ps
sudo docker compose logs --tail=100
sudo docker image prune -a
```