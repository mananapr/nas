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

### postgresql migration

[immich postgres major upgrade](https://github.com/immich-app/base-images/tree/main/postgres#major-upgrade)

1. Stop the entire custom app in TrueNAS.
2. Configure permissions on the **dedicated Postgres dataset**:
   ```bash
   sudo zfs get acltype,aclmode,aclinherit home/docker/postgres
   sudo zfs set aclmode=discard home/docker/postgres
   sudo zfs set aclinherit=discard home/docker/postgres
   ```
3. Run the upgrade container:
   ```bash
   sudo docker run -d \
     --name postgres-upgrade \
     --env-file /mnt/home/docker/.env \
     -e POSTGRES_DB=immich \
     -e DB_STORAGE_TYPE=HDD \
     --mount type=bind,source=/mnt/home/docker/postgres,target=/var/lib/postgresql/data \
     ghcr.io/immich-app/postgres:<version>-upgrade

   sudo docker logs -f postgres-upgrade

   # if successful
   sudo docker stop postgres-upgrade
   sudo docker rm postgres-upgrade
   ```
4. Update the Postgres image in docker compose.
