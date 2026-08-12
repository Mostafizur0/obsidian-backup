[[Docker]]

https://kodekloud.com/blog/docker-cp/

```bash
docker cp /path/to/local/file.txt container_name:/path/in/container/file.txt

docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/usr/src/

docker cp my_container:/var/log/nginx/access.log . //Copying a File from Container to Host

docker cp my_container:/var/www/html ./html_backup // Copying Entire Directories
```
docker cp - copy a file or directory from host machine to the docker container in a defined directory and vice versa.
Validate through
```
docker exec ubuntu_latest ls -l /usr/src/
```

> [!NOTE]
> Transferring Files Between Two Containers not supported: The `docker cp` command does **not** support direct container-to-container transfers. You must route the file through your host machine using two commands:
> ```bash
> docker cp container_A:/app/data.json ./temp.json
> docker cp ./temp.json container_B:/app/data.json
> rm ./temp.json
> ```
