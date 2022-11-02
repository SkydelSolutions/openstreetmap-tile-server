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

# Build Container from repository :
```sh
docker build https://github.com/SkydelSolutions/openstreetmap-tile-server.git -t osm
```

# Create Volumes
```sh
docker volume create osm-data
docker volume create osm-tiles
```

# Build Tiles : 
This container will stop once it is done.
```sh
docker run -e THREADS=16 -e "OSM2PGSQL_EXTRA_ARGS=-C 8192" -v /media/skydel/OSM/planet-latest.osm.pbf:/data/region.osm.pbf -v /media/skydel/OSM/planet.poly:/data/region.poly -v osm-data:/data/database/ -v osm-tiles:/data/tiles/ osm import
```


# Modify Skydel config : 
```sh
nano /usr/lib/skydel-sdx/data/maps/earth/openstreetmap/openstreetmap.dgml
```

### Replace : 
```xml
          <downloadUrl protocol="https" host="a.tile.openstreetmap.org" path="/"/>
          <downloadUrl protocol="https" host="b.tile.openstreetmap.org" path="/"/>
          <downloadUrl protocol="https" host="c.tile.openstreetmap.org" path="/"/>
```

### With : 
```xml
          <downloadUrl protocol="http" host="172.17.0.2" path="/tile/"/>
```

# Run Container :
```sh
docker run -p 8080:80 -v osm-data:/data/database/ -e ALLOW_CORS=enabled -v osm-tiles:/data/tiles/ -d osm run
```

# Start Skydel
### Open Preferences / Marble Proxy
### Configure as follow
- Address: 172.17.0.2
- Transport Protocol : Http
- Port : 80

# Troubleshooting
- Make sure you're using root.
- Remount drive and restart docker
```sh
umount /media/skydel/OSM1
umount /media/skydel/OSM
mount /dev/sdb1 /media/skydel/OSM
service docker stop
systemctl daemon-reload
service docker start
```