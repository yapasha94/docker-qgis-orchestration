QGIS Mapserver Demo Orchestration
=================================

Orchestration scripts for running QGIS demo server.

To use you need to have docker installed on any linux host. You
need a minimum of docker 1.0.0

There are three main scripts here:

* **build.sh**: This will build all the docker images. 
  You can optionally pass a parameter which is an alternate organisation or
  user name which will let you build against your forks of the official QGIS
  repos. e.g.

  ``./build.sh [github organisation or user name]``
  
  During the build process, these docker images will be built:
  * **kartoza/qgis-btsync**: This runs a btsync server that will
    contain the GIS. The btsync 
    peer hosted here is read only. To push data to the server, you need to 
    have the write token (ask Tim or Richard for it if needed). The 
    container run from this image will be a long running daemon. 
  * **kartoza/qgis-server**: This runs a QGIS mapserver container 
    which has apache, mod_fcgi and QGIS Mapserver installed in it.
  * **kartoza/qgis-postgis**: This runs a postgis instance in it.
  * **kartoza/qgis-desktop**: This runs a dockerised version of QGIS desktop 
    and can be used for testing and loading postgis data in a local development 
    environment.
  
* **deploy.sh**: This script will launch containers for all the long running
  daemons mentioned in the list above. Each container will be named with
  the base name of the image.
 
* **run.sh**: This script will run any short lived containerised apps.
  Currently there are no such apps in this architecture.
  

There is an additional script called `functions.sh` which contains common
functions shared by all scripts.

The orchestration scripts provided here will build against docker recipes
hosted in GitHub - there is no need to check them all out individually. So 
to use all you need to do is (on your host):


```
git clone https://github.com/kartoza/docker-qgis-orchestration.git
cd docker-qgis-orchestration
./build.sh
./deploy.sh
```

After deploy is run you should have 3 running containers e.g.:

```
CONTAINER ID        IMAGE                            COMMAND                CREATED             STATUS              PORTS                                                                       NAMES
5142b661cb4e        kartoza/qgis-server:latest       /bin/sh -c apachectl   18 minutes ago      Up 18 minutes       0.0.0.0:8198->80/tcp                                                        qgis-server                                      
a1c711ccfc70        kartoza/postgis:latest           /bin/sh -c /start-po   18 minutes ago      Up 18 minutes       5432/tcp                                                                    qgis-postgis,qgis-server/qgis-postgis            
c1ecef31a1b8        kartoza/qgis-btsync:latest       /start.sh              18 minutes ago      Up 18 minutes       0.0.0.0:55555->55555/tcp, 0.0.0.0:8888->8888/tcp                            qgis-btsync,qgis-server/qgis-btsync 
```

Lastly you will probably want to set up a reverse proxy pointing to your QGIS Mapserver container (our orchestration scripts publish on 8198 by default). Here is a sample configuration for nginx:

```
upstream maps.kartoza { server localhost:8198;}
 
server {
  listen      80;
  server_name maps.kartoza.com;
  location    / {
    proxy_pass  http://maps.kartoza;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $remote_addr;
  }
}

```


--------

Tim Sutton and Richard Duivenvoorde, August 2014

