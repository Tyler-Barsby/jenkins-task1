# Errors I Hit

## No space left on device

Kept running out of disk space on the EC2. Here's what fixed it:

```bash
# clear docker stuff
docker system prune -af

# clear trivy cache
rm -rf ~/.cache/trivy

# check how much space you have
df -h
```

If that's still not enough, containerd was secretly hoarding 1.6GB:

```bash
sudo systemctl stop docker.socket
sudo systemctl stop docker
sudo systemctl stop containerd
sudo rm -rf /var/lib/containerd/*
sudo systemctl start containerd
sudo systemctl start docker
```

Also had old kernel headers taking up 330MB for no reason:

```bash
sudo apt-get autoremove -y
sudo apt-get clean
```
