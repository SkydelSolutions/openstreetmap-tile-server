# Install Docker
Full Procedure : https://docs.docker.com/engine/install/ubuntu/

# Build Container from repository :
```sh
docker build https://github.com/SkydelSolutions/openstreetmap-tile-server.git -t osm
```

# Create Volumes
```sh
docker volume create osm-data
docker volume create osm-tiles
```


# Option A: Import data for a Single Region:
You can run this command again to download additional regions.

This container will stop once it is done.
```sh
docker run -e THREADS=8 -e "OSM2PGSQL_EXTRA_ARGS=-C 8192" -e DOWNLOAD_PBF=https://download.geofabrik.de/north-america/canada/quebec-latest.osm.pbf -e DOWNLOAD_POLY=https://download.geofabrik.de/north-america/canada/quebec.poly -v osm-data:/data/database/ -v osm-tiles:/data/tiles/ osm import
```
without network : 
```sh
docker run -e THREADS=8 -e "OSM2PGSQL_EXTRA_ARGS=-C 8192" -v /home/user/Downloads/quebec-latest.osm.osm.pbf:/data/region.osm.pbf -v /home/user/Downloads/quebec.poly:/data/region.poly -v osm-data:/data/database/ -v osm-tiles:/data/tiles/ osm import
```

# Option B: Import the entire planet: 
**This will take a VERY long time to complete.**
## Requires
- 32 Thread
- 64 GB of RAM
- 1TB of SSD
This container will stop once it is done.
```sh
docker run -e THREADS=32 -e "OSM2PGSQL_EXTRA_ARGS=-C 65536" -e "FLAT_NODES=enabled" -e DOWNLOAD_PBF=https://planet.openstreetmap.org/pbf/planet-latest.osm.pbf -v osm-data:/data/database/ -v osm-tiles:/data/tiles/ osm import
```

# Run the container :
```sh
docker run -p 8080:80 -v osm-data:/data/database/ -v osm-tiles:/data/tiles/ -d osm run
```


# Modify Skydel config : 
in order for the file server to work on client devices you need to update this configuration file on the clients.

The IP should be the IP of the docker host machine.
```sh
nano /usr/lib/skydel-sdx/data/maps/earth/openstreetmap/openstreetmap.dgml
```

Windows:
```
C:\Program Files\Skydel\data\maps\earth\openstreetmap\openstreetmap.dgml
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

# Start Skydel
### Open Preferences / Marble Proxy
### Configure as follow
- Address: 192.168.1.##
- Transport Protocol : Http
- Port : 8080

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