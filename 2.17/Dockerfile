FROM ubuntu:trusty
MAINTAINER winsent<pipetc@gmail.com>


ENV DEBIAN_FRONTEND noninteractive
ENV GDAL_PATH /usr/share/gdal
ENV GEOSERVER_HOME /opt/geoserver
ENV JAVA_HOME /usr
ENV GDAL_DATA $GDAL_PATH/1.10
ENV PATH $GDAL_PATH:$PATH
ENV LD_LIBRARY_PATH $LD_LIBRARY_PATH:/usr/lib/jni:/usr/share/java

RUN export DEBIAN_FRONTEND=noninteractive
RUN dpkg-divert --local --rename --add /sbin/initctl

# Install packages
RUN \
  apt-get -y update --fix-missing && \
  apt-get -y install unzip software-properties-common && \
  echo openjdk-7-jdk shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get -y update && \
  apt-get install -y openjdk-7-jdk gdal-bin libgdal-java && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/oracle-jdk8-installer && \
  rm -rf /tmp/* /var/tmp/*

ENV JAVA_HOME /usr/lib/jvm/openjdk-7-jdk

# Get native JAI and ImageIO
RUN \
    cd $JAVA_HOME && \
    wget http://geotron.ntsomz.ru/media/geoserver/jai-1_1_3-lib-linux-amd64-jdk.bin && \
    echo "yes" | sh jai-1_1_3-lib-linux-amd64-jdk.bin && \
    rm jai-1_1_3-lib-linux-amd64-jdk.bin

RUN \
    cd $JAVA_HOME && \
    export _POSIX2_VERSION=199209 &&\
    wget http://geotron.ntsomz.ru/media/geoserver/jai_imageio-1_1-lib-linux-amd64-jdk.bin && \
    echo "yes" | sh jai_imageio-1_1-lib-linux-amd64-jdk.bin && \
    rm jai_imageio-1_1-lib-linux-amd64-jdk.bin

#
# GEOSERVER INSTALLATION
#
ENV GEOSERVER_VERSION 2.17.2

# Get GeoServer
RUN wget -c http://downloads.sourceforge.net/project/geoserver/GeoServer/$GEOSERVER_VERSION/geoserver-$GEOSERVER_VERSION-bin.zip -O ~/geoserver.zip &&\
    unzip ~/geoserver.zip -d /opt && mv -v /opt/geoserver* /opt/geoserver && \
    rm ~/geoserver.zip

# Get OGR plugin
RUN wget -c http://downloads.sourceforge.net/project/geoserver/GeoServer/$GEOSERVER_VERSION/extensions/geoserver-$GEOSERVER_VERSION-ogr-wfs-plugin.zip -O ~/geoserver-ogr-plugin.zip &&\
    unzip -o ~/geoserver-ogr-plugin.zip -d /opt/geoserver/webapps/geoserver/WEB-INF/lib/ && \
    rm ~/geoserver-ogr-plugin.zip
    
# Get GDAL plugin
RUN wget -c http://downloads.sourceforge.net/project/geoserver/GeoServer/$GEOSERVER_VERSION/extensions/geoserver-$GEOSERVER_VERSION-gdal-plugin.zip -O ~/geoserver-gdal-plugin.zip &&\
    unzip -o ~/geoserver-gdal-plugin.zip -d /opt/geoserver/webapps/geoserver/WEB-INF/lib/ && \
    rm ~/geoserver-gdal-plugin.zip

# Get printing plugin
RUN wget -c http://downloads.sourceforge.net/project/geoserver/GeoServer/$GEOSERVER_VERSION/extensions/geoserver-$GEOSERVER_VERSION-printing-plugin.zip -O ~/geoserver-printing-plugin.zip &&\
    unzip ~/geoserver-printing-plugin.zip -d /opt/geoserver/webapps/geoserver/WEB-INF/lib/ && \
    rm ~/geoserver-printing-plugin.zip

# Get import plugin
RUN wget -c http://downloads.sourceforge.net/project/geoserver/GeoServer/$GEOSERVER_VERSION/extensions/geoserver-$GEOSERVER_VERSION-importer-plugin.zip -O ~/geoserver-importer-plugin.zip &&\
    unzip -o ~/geoserver-importer-plugin.zip -d /opt/geoserver/webapps/geoserver/WEB-INF/lib/ && \
    rm ~/geoserver-importer-plugin.zip

# Get vector tiles plugin
RUN wget -c http://downloads.sourceforge.net/project/geoserver/GeoServer/$GEOSERVER_VERSION/extensions/geoserver-$GEOSERVER_VERSION-vectortiles-plugin.zip -O ~/geoserver-vectortiles-plugin.zip && \
    unzip -o ~/geoserver-vectortiles-plugin.zip -d $GEOSERVER_HOME/webapps/geoserver/WEB-INF/lib/ && \
    rm ~/geoserver-vectortiles-plugin.zip

# Get css styles plugin
RUN wget -c http://downloads.sourceforge.net/project/geoserver/GeoServer/$GEOSERVER_VERSION/extensions/geoserver-$GEOSERVER_VERSION-css-plugin.zip -O ~/geoserver-css-plugin.zip && \
    unzip -o ~/geoserver-css-plugin.zip -d $GEOSERVER_HOME/webapps/geoserver/WEB-INF/lib/ && \
    rm ~/geoserver-css-plugin.zip

#RUN wget -c https://build.geoserver.org/geoserver/2.17.x/ext-latest/geoserver-2.17-SNAPSHOT-css-plugin.zip -O ~/geoserver-css-plugin.zip && \
#    unzip -o ~/geoserver-css-plugin.zip -d $GEOSERVER_HOME/webapps/geoserver/WEB-INF/lib/ && \
#    rm ~/geoserver-css-plugin.zip

# Replace GDAL Java bindings
RUN rm -rf $GEOSERVER_HOME/webapps/geoserver/WEB-INF/lib/imageio-ext-gdal-bindings-1.9.2.jar
RUN cp /usr/share/java/gdal.jar $GEOSERVER_HOME/webapps/geoserver/WEB-INF/lib/gdal.jar

# Remove old JAI from geoserver
RUN rm -rf $GEOSERVER_HOME/webapps/geoserver/WEB-INF/lib/jai_codec-1.1.3.jar && \
    rm -rf $GEOSERVER_HOME/webapps/geoserver/WEB-INF/lib/jai_core-1.1.3.jar && \
    rm -rf $GEOSERVER_HOME/webapps/geoserver/WEB-INF/lib/jai_imageio-1.1.jar

# Enable JAI-EXT
RUN sed -i '/-jar start.jar/ s/$/-Dorg.geotools.coverage.jaiext.enabled=true/' /opt/geoserver/bin/startup.sh

# Set default locale
ENV LANG C.UTF-8

# Expose GeoServer's default port
EXPOSE 8080
CMD ["/opt/geoserver/bin/startup.sh"]
