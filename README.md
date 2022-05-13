Build Docker Images
============================================================

1. base image for development and production\
docker build -t ncf-base -f Dockerfile-ncf-base .

2. image for development\
docker build -t ncf-dev -f Dockerfile-ncf-dev .

3. image for production\
docker build -t ncf-run -f Dockerfile-ncf-run .


Setup the development env
------------------------------------------------------------
* inside the host shell

1. download tsn-demo
```
git submodule init
git submodule update
```

2.
```
docker run --privileged --network host -d -v /var/run/dbus:/var/run/dbus:rw \
    -v /var/run/redis:/var/run/redis:rw -v "$PWD":/home/myapp:rw ncf-dev

```

NOTE: use host network.

3.
```
docker exec -it CONTAINER_ID bash
```

* inside the ncf-dev container
4.
```
service lldpd start

# already done in Dockerfile-ncf-dev
#cd tsn-mod
#sysrepoctl -i iana-if-type\@2017-01-19.yang
#sysrepoctl -i ieee802-dot1ab-lldp.yang -s .
#sysrepoctl -i ietf-hardware\@2018-03-13.yang -s .
#sysrepoctl -i iana-hardware\@2018-03-13.yang -s .
```

5.
```
cd tsn-demo
mkdir build
cd build
cmake ..
make
tsndemo &
```

6.
```
netopeer2-cli
connect localhost --login netconf
get-config --source running
```

Setup the production env
------------------------------------------------------------
1. modify the Dockerfile-ncf-run to include the final exeution file.

2. distribute the image to the DUT.

3. start the container on the DUT.


Note for Dbus: (TBD)
------------------------------------------------------------
1. need to install dbus on DUT.
```
apt-get install python3-dbus, dbus
```

2. modify /usr/share/dbus-1/system.conf for our service

```
  <policy context="default">
    <!-- All users can connect to system bus -->
    <allow user="*"/>
....
    <allow own="com.example.SampleService"/>
  </policy>
```

3. need to install our dbus server on DUT.

4. debug dbus
```
qdbus --system
```


Directory structure
------------------------------------------------------------


| Folder      | Brief Description | Ex:
|:---         |:---         |:---
| dbus-examples | python examples for dbus server/client | python3 example-service.py |
| deb-ub-20 | prebuilt deb packages for netopeer2/sysrepo |  |
| libssh0.9.6 | prebuilt libssh for ubuntu 20.04 |  |
| tsn-mod | yang modules used for tsn-demo |
| tsn-demo | demo example for tsn and sysrepo |



