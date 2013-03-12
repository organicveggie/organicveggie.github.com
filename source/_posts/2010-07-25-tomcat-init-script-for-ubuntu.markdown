---
layout: post
title: "Tomcat init script for Ubuntu"
date: 2010-07-25 13:06
comments: true
categories: 
 - Linux
 - Tomcat
 - Ubuntu
---
Recently I spent some time working on improving my init scripts for Tomcat 6.x in a production environment running Ubuntu. One of the major problems we had encountered was that occasionally Tomcat refuses to shut down completely and requires a `kill -9` to stop it. The standard init scripts I had seen didn't solve this problem at all.
<!-- more -->
[Laliluna](http://www.laliluna.de/) has a great [article](http://www.laliluna.de/articles/tomcat-startup-script-linux.html) that focuses on RedHat, CentOS and Fedora. Unfortunately, their scripts didn't work correctly under Ubuntu 8.04 LTS. As a result, I spent some time modifying their scripts to get them to work correctly under Unbuntu. Many thanks to Laliluna for doing the heavy work.

Without any further ado, here's what I put together:

{% codeblock /etc/init.dtomcat lang:bash %}
#!/bin/bash
#
# Startup script for Jakarta Tomcat
# Script should work on Ubuntu Linux.
# WARNING: The script does not allow to run Tomcat on privileged ports as non root user. 
# For this use case try : <a href="http://tomcat.apache.org/tomcat-6.0-doc/setup.html">http://tomcat.apache.org/tomcat-6.0-doc/setup.html</a> and <a href="http://commons.apache.org/daemon/jsvc.html
#">http://commons.apache.org/daemon/jsvc.html
#</a> 
# Should start normally after the databases and before http server
# chkconfig: 345 80 10
# description: Jakarta Tomcat Java Servlet/JSP Container
# processname: tomcat
# pidfile: /var/run/tomcat/tomcat.pid
 
##### In this area you can find settings which are likely to change frequently ####
 
JAVA=/opt/java/current/bin/java
# unprivileged user running Tomcat server
tomcatuser=tomcat
 
# servicename used as pidfile and lockfile name, must correspond to 'processname:' at the top of this file
# If not linux will not detect the running service during runlevel switch and will not shut it down normally
servicename=tomcat
 
# folder where Tomcat is installed
CATALINA_HOME=/opt/tomcat
 
# Options for the JVM
JAVA_OPTS="$JAVA_OPTS -Xms1024m -Xmx2048m -XX:MaxPermSize=512m -XX:PermSize=128m"
JAVA_OPTS="$JAVA_OPTS -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:ParallelGCThreads=4 JAVA_OPTS="-Djavax.servlet.request.encoding=UTF-8 -Djavax.servlet.response.encoding=UTF-8 -Dfile.encoding=UTF-8 $JAVA_OPTS"
 
##### End of frequent settings area #####
 
pidfile=/var/run/tomcat/$servicename
lockfile=/var/lock/$servicename
 
#runsecure=1 #starts tomcat with java security
runsecure=0
 
# Optional additional libs you would like to add to the classpath (= JVM Option -classpath)
CLASSPATH=""
# Optional Java Security Socket extension
# CLASSPATH="$CLASSPATH":"$JSSE_HOME"/lib/jcert.jar:"$JSSE_HOME"/lib/jnet.jar:"$JSSE_HOME"/lib/jsse.jar
 
# path to Tomcat lib
CLASSPATH="$CLASSPATH":"$CATALINA_HOME"/bin/bootstrap.jar
 
# Directory holding configuration, defaults to CATALINA_HOME
# In a Tomcat cluster you might reuse the servicename to identify the base directory
 
CATALINA_BASE="$CATALINA_HOME"
# server log during startup / shutdown
logfile=$CATALINA_BASE/logs/catalina.out
# endorsed allows to overwrite JVM libs -&gt; JVM option -Djava.endorsed.dirs 
#JAVA_ENDORSED_DIRS="$CATALINABASEDIR"/endorsed
 
# Define the java.io.tmpdir to use for Catalina
CATALINA_TMPDIR="$CATALINA_BASE"/temp
 
# Set juli LogManager if it is present
if [ -r "$CATALINA_BASE"/conf/logging.properties ]; then
  JAVA_OPTS="$JAVA_OPTS -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager"
  LOGGING_CONFIG="-Djava.util.logging.config.file=$CATALINA_BASE/conf/logging.properties"
fi
 
#### End of settings #####
 
# build java command to start Tomcat
command="$JAVA $JAVA_OPTS $LOGGING_CONFIG $CATALINA_OPTS $LOGGING_CONFIG \
      -Djava.endorsed.dirs=$JAVA_ENDORSED_DIRS -classpath $CLASSPATH \
      -Dcatalina.base=$CATALINA_BASE \
      -Dcatalina.home=$CATALINA_HOME \
      -Djava.io.tmpdir=$CATALINA_TMPDIR" 
 
if [ "$runsecure" = "1" ]; then
  command="$command -Djava.security.manager -Djava.security.policy=$CATALINA_BASE/conf/catalina.policy"
fi
 
command="$command org.apache.catalina.startup.Bootstrap"
 
 
start()
{
	echo $"Starting $servicename based at $CATALINA_BASE "
 
	daemon --user=$tomcatuser --pidfile=$pidfile --output=$logfile -- $command start
	RETVAL=$?
 
	[ "$RETVAL" = 0 ] &amp;&amp; touch $lockfile
	echo
}
 
stop()
{
	echo -n $"Stopping $prog: "
	if [ ! -r $pidfile ]; then
		echo "Pidfile $pidfile cannot be read"
		RETVAL=1
		return
	fi
	# Sends TERM signal first and kills finally after 10 seconds
	start-stop-daemon --pidfile $pidfile -R 10 --stop
	RETVAL=$?
	[ $RETVAL = 0 ] &amp;&amp; rm -f ${lockfile} ${pidfile}
	echo
}
 
version()
{
	$JAVA -classpath $CATALINA_HOME/lib/catalina.jar org.apache.catalina.util.ServerInfo
	RETVAL=$?
}
 
case "$1" in
	start)
		start
		;;
	stop)
		stop
		;;
	restart)
		stop
		start
		;;
	version)
		version
		;;
	status)
		status -p $pidfile $servicename
		RETVAL=$?
		;;
	*)
		echo $"Usage: $0 {start|stop|restart|version|status}"
		RETVAL=1
esac
exit $RETVAL
{% endcodeblock %}