FROM tomcat:8.0.51-jre8-alpine
#FROM tomcat:8-jdk8-corretto
EXPOSE 8080
RUN rm -rf /usr/local/tomcat/webapps/*
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war
CMD ["catalina.sh","run"]