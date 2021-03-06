# Fetch image based on Tomcat 8.5.34 and Java 11
# More infos about this image: https://github.com/Alfresco/alfresco-docker-base-tomcat
FROM alfresco/alfresco-base-tomcat:8.5.34-java-11-openjdk-centos-7

# Set default docker_context. Will / Can be overriden with maven.
ARG resource_path=target

# Create prerequisite to store tools and properties
RUN mkdir -p /usr/local/tomcat/shared/classes/alfresco/extension && \
    mkdir /usr/local/tomcat/alfresco-mmt
RUN touch /usr/local/tomcat/shared/classes/alfresco-global.properties

# You need to run `mvn clean install` in the root of this project to update the following dependencies
# Copy the WAR files to the appropriate location for your application server
# Copy the JDBC drivers for the database you are using to the lib/ directory.
# Copy the alfresco-mmt.jar
COPY ${resource_path}/war /usr/local/tomcat/webapps
COPY ${resource_path}/connector/* /usr/local/tomcat/lib/
COPY ${resource_path}/alfresco-mmt/* /usr/local/tomcat/alfresco-mmt/
RUN rm -rf /usr/local/tomcat/webapps/ROOT
COPY ${resource_path}/server-root/* /usr/local/tomcat/webapps
COPY ${resource_path}/wti-bin/* /usr/local/tomcat/webapps

# Change the value of the shared.loader= property to the following:
# shared.loader=${catalina.base}/shared/classes
RUN sed -i "s/shared.loader=/shared.loader=\${catalina.base}\/shared\/classes/" /usr/local/tomcat/conf/catalina.properties

# Add here configurations for alfresco-global.properties
RUN echo -e '\n\
jodconverter.enabled=true\n\
' >> /usr/local/tomcat/shared/classes/alfresco-global.properties

# Add debug for testing
# RUN echo -e '\n\
# log4j.logger.org.alfresco.repo.content.transform.TransformerDebug=debug\n\
# ' >> /usr/local/tomcat/shared/classes/alfresco/extension/custom-log4j.properties

RUN mkdir -p /usr/local/tomcat/amps

# Copy the amps from build context to the appropriate location for your application server
COPY ${resource_path}/amps /usr/local/tomcat/amps

# Install amps on alfresco.war
RUN java -jar /usr/local/tomcat/alfresco-mmt/alfresco-mmt*.jar install \
              /usr/local/tomcat/amps /usr/local/tomcat/webapps/alfresco -directory -nobackup -force

# Docker CMD from parent image starts the server

# Unpack ROOT.war and make webapps folder read-only.
RUN mkdir /usr/local/tomcat/webapps/ROOT && cd /usr/local/tomcat/webapps/ROOT && jar -xvf /usr/local/tomcat/webapps/ROOT.war && rm -f /usr/local/tomcat/webapps/ROOT.war && chmod -R =r /usr/local/tomcat/webapps && \
# Add catalina.policy to ROOT.war and alfresco.war
# Grant all security permissions to alfresco webapp because of numerous permissions required in order to work properly.
# Grant only deployXmlPermission to ROOT webapp.
    sed -i -e "\$a\grant\ codeBase\ \"file:\$\{catalina.base\}\/webapps\/alfresco\/-\" \{\n\    permission\ java.security.AllPermission\;\n\};\ngrant\ codeBase\ \"file:\$\{catalina.base\}\/webapps\/ROOT\/-\" \{\n\    permission org.apache.catalina.security.DeployXmlPermission \"ROOT\";\n\};" /usr/local/tomcat/conf/catalina.policy

# Unpack wti-bin.war
RUN mkdir /usr/local/tomcat/webapps/wti-bin && cd /usr/local/tomcat/webapps/wti-bin && jar -xvf /usr/local/tomcat/webapps/wti-bin.war && rm -f /usr/local/tomcat/webapps/wti-bin.war

# To remote debug into this image add: EXPOSE 8000
# Changes are also required to the docker-compose/docker-compose.yml file.
# EXPOSE 8000
