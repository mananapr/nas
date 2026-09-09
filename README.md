### nas

1. setup ZFS datasets as follows
```
home
|__docker
|____immich
|____jellyfin
|____memos
|____microbin
|____paperless
|____postgres
|____qbittorrent
|__music
|__video
|__photo
```
2. copy `docker-compose.yml` and `.env` to `/mnt/home/docker/`
3. install custom yaml in truenas
```yml
include:
  - /mnt/home/docker/docker-compose.yml
services: {}
```
4. run postgres migrations as defined in `init-db.sql` and restore dataset backups from backblaze

### vps

```bash
cp .env.example .env
docker compose up -d

# create the first Soju account
docker compose exec soju sojuctl -config /etc/soju/soju.conf user create -username your-user -password your-password -admin

# restore backblaze docker volume backups
docker run --rm -v apps_baikal-config:/target -v ~/baikal_config:/source busybox cp -a /source/. /target/
docker run --rm -v apps_baikal-data:/target -v ~/baikal_data:/source busybox cp -a /source/. /target/
docker run --rm -v apps_vw-data:/target -v ~/vw-data:/source busybox cp -a /source/. /target/
```
