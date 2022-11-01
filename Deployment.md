# Download the maps
## Download:
https://planet.openstreetmap.org/pbf/planet-latest.osm.pbf
## To:
```
/media/skydel/OSM/planet-latest.osm.pbf
```
# Install Docker
Full Procedure : https://docs.docker.com/engine/install/ubuntu/

From Script : 
```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

# Go root :
```sh
sudo -i
```

# Move Docker :
```sh
service docker stop
```
### Create/Edit config file
```sh
nano /etc/docker/daemon.json
```
File Content :
```json
{
  "data-root": "/media/skydel/OSM"
}
```
### Restart Docker
```sh
systemctl daemon-reload
service docker start
```
### Validate Configuration
```sh
docker info
```

# Pull dockerfile

# Build Containers : 
docker pull overv/openstreetmap-tile-server
docker volume create osm-data
docker volume create osm-tiles


Build Tiles : 
docker run -v /media/skydel/OSM/planet-latest.osm.pbf:/data/region.osm.pbf -v osm-data:/data/database/ overv/openstreetmap-tile-server import


Run Container :
docker run -p 8080:80 -v osm-data:/data/database/ -e ALLOW_CORS=enabled -v osm-tiles:/data/tiles/ -d overv/openstreetmap-tile-server run
#take note of the first 3 characters of the id.

### is the first char of the container id.
Check logs for IP :
docker logs ###

Modify Skydel config : 
nano /usr/lib/skydel-sdx/data/maps/earth/openstreetmap/openstreetmap.dgml

Replace : 

          <downloadUrl protocol="https" host="a.tile.openstreetmap.org" path="/"/>
          <downloadUrl protocol="https" host="b.tile.openstreetmap.org" path="/"/>
          <downloadUrl protocol="https" host="c.tile.openstreetmap.org" path="/"/>

With : 

          <downloadUrl protocol="http" host="172.17.0.2" path="/tile/"/>

Start Skydel
Set Marble ip to 127.0.0.1
Port : 80
Protocol : http