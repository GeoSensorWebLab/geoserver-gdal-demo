# Installation Guide

This guide will hopefully get you set up with Oracle JRE 8, Tomcat 9, GeoServer 2.11.1, and GDAL 2.2 on Ubuntu 16.04 Server.

In my case I am running this tutorial in a VirtualBox VM on IP address 192.168.34.10, you may have a different IP you need to substitute.

## Install Java

```sh
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt update
$ sudo apt install oracle-java8-installer
$ sudo apt install oracle-java8-set-default
$ java -version
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
```

If there is a hash mismatch error during installation, you have to manually download Java for the installer. URL can be found on [Oracle's Java download site](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html). Manual fix below:

```sh
$ cd /var/cache/oracle-java8-installer
$ sudo rm jdk-*
$ sudo wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u144-b01/090f390dda5b47b9b721c7dfaa008135/jdk-8u144-linux-x64.tar.gz
$ sudo apt install oracle-java8-installer
$ sudo apt install oracle-java8-set-default
$ java -version
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
```

## Install Tomcat9

```sh
$ mkdir ~/src
$ cd ~/src
$ wget http://apache.mirror.iweb.ca/tomcat/tomcat-9/v9.0.0.M26/bin/apache-tomcat-9.0.0.M26.tar.gz
$ tar xzf apache-tomcat-9.0.0.M26.tar.gz
$ sudo cp -r apache-tomcat-9.0.0.M26 /opt/tomcat9
$ sudo useradd tomcat9
$ sudo usermod -s /bin/false tomcat9
$ sudo chown -R tomcat9 /opt/tomcat9
$ sudo -u tomcat9 /opt/tomcat9/bin/startup.sh
```

Tomcat should now be visible at [http://192.168.34.10:8080](http://192.168.34.10:8080).

## Install GDAL

```sh
$ sudo add-apt-repository -y ppa:ubuntugis/ubuntugis-unstable
$ sudo apt upgrade
$ sudo apt install gdal-bin libgdal-dev libgdal-java libgdal20 gdal-data
$ gdalinfo --version
GDAL 2.2.1, released 2017/06/23
```

## Install GeoServer

```sh
$ cd ~/src
$ wget http://sourceforge.net/projects/geoserver/files/GeoServer/2.11.1/geoserver-2.11.1-war.zip
$ sudo apt install unzip
$ unzip geoserver-2.11.1-war.zip -d geoserver-2.11.1
$ sudo cp geoserver-2.11.1/geoserver.war /opt/tomcat9/webapps/.
$ sudo chown -R tomcat9 /opt/tomcat9
```

Geoserver should now be visible at [http://192.168.34.10:8080/geoserver/web](http://192.168.34.10:8080/geoserver/web). The default login is `admin` with password `geoserver`.

If you go to `Data > Stores` and `Add new Store`, you can see there are no GDAL options yet (e.g. `VRT`).

## Install GeoServer GDAL Plugin

Instructions based on [GeoServer documentation](http://docs.geoserver.org/latest/en/user/data/raster/gdal.html), but this should definitely work for Ubuntu. The official guide suggests using ImageIO-Ext 1.1.16 JARs or GDAL binding JARs; we will use the latter. DO NOT install `imageio-ext-gdal-bindings-1.9.2.jar` in your GeoServer library directory, that is for an older version of GDAL!

```sh
$ cd ~/src
$ wget http://sourceforge.net/projects/geoserver/files/GeoServer/2.11.1/extensions/geoserver-2.11.1-gdal-plugin.zip
$ unzip geoserver-2.11.1-gdal-plugin.zip -d geoserver-2.11.1-gdal-plugin
$ sudo cp geoserver-2.11.1-gdal-plugin/*.jar /opt/tomcat9/webapps/geoserver/WEB-INF/lib/.
$ sudo cp /usr/share/java/gdal.jar /opt/tomcat9/webapps/geoserver/WEB-INF/lib/.
$ sudo chown -R tomcat9 /opt/tomcat9
```

At this point GeoServer will see the plugin but will not load it (See `About & Status > Server Status`, then `Modules` and look for `gs-gdal` in the list). We need to set some options for Tomcat to pass to Java and GeoServer.

```sh
$ sudo -u tomcat9 /opt/tomcat9/bin/shutdown.sh
$ sudo -u tomcat9 GDAL_DATA=/usr/share/gdal/2.2 JAVA_OPTS="-Djava.library.path=/usr/lib/jni" /opt/tomcat9/bin/startup.sh
```

Now if you go to the GeoServer main page (important — do not reload the page as it won't show the new settings) and then go to the Modules list, `gs-gdal` should now have a checkmark and be loaded. Additionally, visiting the `New data store` page will show the GDAL options.

## Author

James Badger (<jpbadger@ucalgary.ca>)

Some information based on the GeoServer documentation for GDAL raster coverage plugin installation.

## License

This tutorial is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
