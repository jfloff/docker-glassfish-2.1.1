FROM buildpack-deps:wheezy-scm
MAINTAINER João Loff <jfloff@gsd.inesc-id.pt>

################################################################################
# Configure Java JDK 6 v6u31
# Based on: https://github.com/docker-library/openjdk/blob/master/6-jdk/Dockerfile

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    build-essential \
  && rm -rf /var/lib/apt/lists/*

# Default to UTF-8 file.encoding
ENV LANG C.UTF-8

# Due to JDK version bug we need a specific version of openjdk6
ENV JAVA_VERSION 6u31
ENV JAVA_HOME /usr/lib/jvm/java-6-jdk
ENV PATH=$PATH:$JAVA_HOME/bin

RUN \
  # Its not possible to download directly from Oracle's website (it requires login)
  # If you prefer you can download from Oracles website directly and adapt this Dockerfile
  # with ADD/COPY commands.
  # You can find them here: http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase6-419409.html
  # - http://download.oracle.com/otn/java/jdk/6u31-b04/jdk-6u31-linux-x64.bin
  # - http://download.oracle.com/otn/java/jdk/6u31-b04/jdk-6u31-linux-i586.bin
  #
  # At the moment I'm downloading from this remote FTP server that had the needed files
  # Just to make sure I'm comparing MD5 against the original Oracle's files
  # (I've manually checked that their MD5's match)
  if [ "$(uname -m)" = "x86_64" ] ; then \
    JDK_URL='ftp://193.239.45.41/by-md5/2/f/2f74dbbee4142b7366c93b115f914fff/jdk-6u31-linux-x64.bin'; \
    JDK_MD5='2f74dbbee4142b7366c93b115f914fff'; \
  else \
    JDK_URL='ftp://193.239.45.41/by-md5/9/e/9e4246fc7a6c0759b8a484ff5e820112/jdk-6u31-linux-i586.bin'; \
    JDK_MD5='9e4246fc7a6c0759b8a484ff5e820112'; \
  fi; \
  mkdir /usr/lib/jvm/ && cd /usr/lib/jvm/ && \
  # Download files and compare MD5
  curl -L -o jdk.bin $JDK_URL && \
  echo "$JDK_MD5  jdk.bin" | md5sum -c - && \
  # run bin file which unpacks folder
  chmod +x jdk.bin && ./jdk.bin && \
  # remove bin to save space
  rm -f jdk.bin && \
  # rename to correct name
  mv jdk1.6.0_31/ java-6-jdk/
  # Allows anyone to listen on un-privileged ports
  # RUN sed -i '/permission java.net.SocketPermission "localhost:0", "listen";/a \\n\tpermission java.net.SocketPermission "localhost:1024-", "listen";' $JAVA_HOME/jre/lib/security/java.policy

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
  # remove jar to save space
  rm -f glassfish.jar && \
  cd $GLASSFISH_HOME && \
  # Remove Windows .bat and .exe files to save space
  find . -name '*.bat' -delete && \
  find . -name '*.exe' -delete && \
  # configure executables and run setup
  chmod -R +x lib/ant/bin && \
  lib/ant/bin/ant -f setup.xml && \
  chmod a+x bin/asadmin && \
  # remove expired key from certificate (SEC5054 Certificate has expired error)
  # solution from: https://stackoverflow.com/a/19591433/1700053
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
