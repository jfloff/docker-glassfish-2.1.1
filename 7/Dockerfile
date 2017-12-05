FROM buildpack-deps:wheezy-scm
MAINTAINER João Loff <jfloff@gsd.inesc-id.pt>

################################################################################
# Install OpenJDK 7
RUN apt-get update && \
  apt-get install -y --no-install-recommends openjdk-7-jdk && \
  rm -rf /var/lib/apt/lists/*

# Default to UTF-8 file.encoding
ENV LANG C.UTF-8
################################################################################
# Configure Glassfish 2.1.1 and Ant 1.6.5
# Based on: https://github.com/d10xa/docker-glassfish-2.1.1

# Configure ENV variables for glassfish and ant
ENV GLASSFISH_HOME /usr/lib/glassfish
ENV ANT_VERSION 1.6.5
ENV ANT_HOME $GLASSFISH_HOME/lib/ant
ENV PATH=$PATH:$GLASSFISH_HOME/bin:$ANT_HOME/bin

RUN \
  # Download and extract glassfish 2.1.1 jar
  cd /usr/lib && \
  curl -o glassfish.jar http://dlc-cdn.sun.com/javaee5/v2.1.1_branch/promoted/Linux/glassfish-installer-v2.1.1-b31g-linux.jar && \
  echo A | java -jar glassfish.jar && \
  # Remove jar to save space
  rm -f glassfish.jar && \
  cd $GLASSFISH_HOME && \
  # Remove Windows .bat and .exe files to save space
  find . -name '*.bat' -delete && \
  find . -name '*.exe' -delete && \
  # Configure executables, modify setup.xml to support Java 7 then run setup
  chmod -R +x lib/ant/bin && \
  sed -i 's/1.6/1.7/g' setup.xml && \
  lib/ant/bin/ant -f setup.xml && \
  chmod a+x bin/asadmin && \
  # Remove expired key from certificate (SEC5054 Certificate has expired error)
  # Solution from: https://stackoverflow.com/a/19591433/1700053
  keytool -storepass changeit -delete -v -alias gtecybertrustglobalca -keystore $GLASSFISH_HOME/domains/domain1/config/cacerts.jks && \
  keytool -storepass changeit -delete -v -alias gtecybertrust5ca -keystore $GLASSFISH_HOME/domains/domain1/config/cacerts.jks && \
  keytool -storepass changeit -delete -v -alias verisignserverca -keystore $GLASSFISH_HOME/domains/domain1/config/cacerts.jks
  # set corba warning level to SEVERE on domain 1 so we don't see JDK version warning
  # solution from: https://stackoverflow.com/a/19692823/1700053
  # asadmin start-domain domain1 && asadmin set server.log-service.module-log-levels.corba=SEVERE && asadmin stop-domain domain1

# Export every lib folder from Glassfish into a classpath var
ENV GLASSFISH_CLASSPATH $GLASSFISH_HOME/lib/*:$GLASSFISH_HOME/domains/domain1/lib/*:$GLASSFISH_HOME/lib/install/applications/admingui/adminGUI_war/WEB-INF/lib/*:$GLASSFISH_HOME/lib/SUNWjdmk/5.1/lib/*:$GLASSFISH_HOME/lib/ant/lib/*:$GLASSFISH_HOME/jbi/lib/*:$GLASSFISH_HOME/javadb/lib/*:$GLASSFISH_HOME/javadb/demo/programs/localcal/lib/*:$GLASSFISH_HOME/updatecenter/lib/*:$GLASSFISH_HOME/imq/lib/*

# Expose ports for admin panel and websites
# localhost:4848 -> user:admin ; pwd:adminadmin
EXPOSE 8080 8181 4848 3700 3820 3920 8686

# Start the GlassFish domain
CMD ["asadmin", "start-domain", "--verbose"]