# Download the maps
## Download:
https://planet.openstreetmap.org/pbf/planet-latest.osm.pbf
## To:
```
/media/skydel/OSM/planet-latest.osm.pbf
```

# Install Docker
Full Procedure : https://docs.docker.com/engine/install/ubuntu/

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

# Build Tiles For the entire planet: 
This container will stop once it is done.

**It will take a VERY long time to complete.**
## Requires
- 32 Thread
- 64 GB of RAM
- 1TB of SSD
```sh
docker run -e THREADS=32 -e "OSM2PGSQL_EXTRA_ARGS=-C 65536" -e DOWNLOAD_PBF=https://planet.openstreetmap.org/pbf/planet-latest.osm.pbf -v osm-data:/data/database/ -v osm-tiles:/data/tiles/ osm import
```

# Build Tiles for a Single Region:
This container will stop once it is done.
You can run this command again to download additional regions.
```sh
docker run -e THREADS=8 -e "OSM2PGSQL_EXTRA_ARGS=-C 8192" -e DOWNLOAD_PBF=https://download.geofabrik.de/north-america/canada/quebec-latest.osm.pbf -e DOWNLOAD_POLY=https://download.geofabrik.de/north-america/canada/quebec.poly -v osm-data:/data/database/ -v osm-tiles:/data/tiles/ osm import
```
without network : 
```sh
docker run -e THREADS=8 -e "OSM2PGSQL_EXTRA_ARGS=-C 8192" -v /home/user/Downloads/quebec-latest.osm.osm.pbf:/data/region.osm.pbf -v /home/user/Downloads/quebec.poly:/data/region.poly -v osm-data:/data/database/ -v osm-tiles:/data/tiles/ osm import
```

# Modify Skydel config : 
in order for the file server to work on client devices you need to update this configuration file on the clients.
The IP should be the IP of the docker host machine.
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
          <downloadUrl protocol="http" host="192.168.1.##" path="/tile/"/>
```

# Run Container :
```sh
docker run -p 80:80 -e ALLOW_CORS=enabled -v osm-data:/data/database/ -v osm-tiles:/data/tiles/ -d osm run
```

# Start Skydel
### Open Preferences / Marble Proxy
### Configure as follow
- Address: 192.168.1.##
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