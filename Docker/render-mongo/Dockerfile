FROM ubuntu:14.04 
MAINTAINER Alex Baden / Neurodata (neurodata.io) 

RUN apt-get update
RUN apt-get -y upgrade
# dependencies for render 
RUN apt-get install -y git \
  maven \
  curl \
  supervisor 

# see https://github.com/saalfeldlab/render/blob/master/docs/src/site/markdown/render-ws.md

# install mongo
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10 
RUN echo "deb http://downloads-distro.mongodb.org/repo/debian-sysvinit dist 10gen" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list

RUN apt-get update && apt-get install -y mongodb-org 

# make a data directory for mongo
RUN mkdir -p /data/mongo 

# expose the mongo port
EXPOSE 27017 

# clone render repo 
WORKDIR /var/www/
RUN git clone https://github.com/saalfeldlab/render.git

# install JDK and Jetty
WORKDIR /var/www/render/
RUN ./render-ws/src/main/scripts/install.sh

# set java home 
RUN { echo 'JAVA_HOME="$(readlink -m ./deploy/jdk*)"'; } >> ~/.mavenrc 

# build render modules 
#RUN mvn package
RUN mvn -Dproject.build.sourceEncoding=UTF-8 package

# deploy the web service 
RUN cp render-ws/target/render-ws-*.war deploy/jetty_base/webapps/render-ws.war

# expose the render port 
EXPOSE 8080 

# setup jetty (copy of render/deploy/jetty_base/jetty_wrapper.sh) 
ENV JETTY_HOME /var/www/render/deploy/jetty-distribution-9.3.7.v20160115
ENV JETTY_BASE /var/www/render/deploy/jetty_base
ENV JETTY_RUN $JETTY_BASE/logs
ENV JETTY_PID $JETTY_RUN/jetty.pid
ENV JETTY_STATE $JETTY_RUN/jetty.state

ENV JAVA_HOME /var/www/render/deploy/jdk1.8.0_73
ENV PATH $JAVA_HOME/bin:$PATH
ENV JAVA $JAVA_HOME/bin/java

# small 4GB server: 
#ENV JAVA_OPTIONS="-Xms3g -Xmx3g -server -Djava.awt.headless=true"

# larger 16GB server
ENV JAVA_OPTIONS="-Xms15g -Xmx15g -server -Djava.awt.headless=true"

# super 500GB server
#ENV JAVA_OPTIONS="-Xms400g -Xmx400g -server -Djava.awt.headless=true"

# setup supervisor 
RUN mkdir -p /var/log/supervisor 

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf 
CMD ["/usr/bin/supervisord"]

