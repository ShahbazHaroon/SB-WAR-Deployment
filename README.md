# SB-WAR-Deployment
<h2>Spring Boot WAR deployment example</h2>

<h4>Step 1: Extend from SpringBootServletInitializer</h4>

<p>SpringBootServletInitializer is an abstract class implementing <a href="http://docs.spring.io/spring-framework/docs/4.3.5.RELEASE/javadoc-api/org/springframework/web/WebApplicationInitializer.html?is-external=true">WebApplicationInitializer</a> interface , the main abstraction for servlet 3.0+ environments in order to configure the ServletContext programmatically. It binds Servlet, Filter and ServletContextInitializer beans from the application context to the servlet container.</p>

<p>Create a class which extends SpringBootServletInitializer, overriding it&#8217;s configure method. You could as well let your application’s main class to extend SpringBootServletInitializer:</p>

<p><code>Main class</code></p>

<strong>Before</strong>

<pre class="brush: java; title: ; notranslate" title="">

package com.ubaidsample.WARExample;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class WarExampleApplication {

	public static void main(String[] args) {
		SpringApplication.run(WarExampleApplication.class, args);
	}

}
</pre>

<strong>After</strong>

<pre class="brush: java; title: ; notranslate" title="">

package com.ubaidsample.WARExample;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

@SpringBootApplication
public class WarExampleApplication extends SpringBootServletInitializer {

	
	@Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(WarExampleApplication.class);
    }
	
	public static void main(String[] args) {
		SpringApplication.run(WarExampleApplication.class, args);
	}

}
</pre>

<h4> Step 2: Update packaging to &#8216;war&#8217;</h4>

<strong>Before</strong>

<pre class="brush: xml; title: ; notranslate" title="">
&lt;project xmlns=&quot;http://maven.apache.org/POM/4.0.0&quot; xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
	xsi:schemaLocation=&quot;http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd&quot;&gt;
	&lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;

	&lt;groupId&gt;com.ubaidsample.WARExample&lt;/groupId&gt;
	&lt;artifactId&gt;WarExampleApplication&lt;/artifactId&gt;
	&lt;version&gt;1.0.0&lt;/version&gt;
	&lt;packaging&gt;jar&lt;/packaging&gt;

	&lt;name&gt;${project.artifactId}&lt;/name&gt;

	&lt;parent&gt;
		&lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
		&lt;artifactId&gt;spring-boot-starter-parent&lt;/artifactId&gt;
		&lt;version&gt;1.4.3.RELEASE&lt;/version&gt;
	&lt;/parent&gt;
</pre>

<strong>After</strong>
<pre class="brush: xml; title: ; notranslate" title="">
&lt;project xmlns=&quot;http://maven.apache.org/POM/4.0.0&quot; xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
	xsi:schemaLocation=&quot;http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd&quot;&gt;
	&lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;

	&lt;groupId&gt;com.ubaidsample.WARExample&lt;/groupId&gt;
	&lt;artifactId&gt;WarExampleApplication&lt;/artifactId&gt;
	&lt;version&gt;1.0.0&lt;/version&gt;
	&lt;packaging&gt;war&lt;/packaging&gt;

	&lt;name&gt;${project.artifactId}&lt;/name&gt;

	&lt;parent&gt;
		&lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
		&lt;artifactId&gt;spring-boot-starter-parent&lt;/artifactId&gt;
		&lt;version&gt;1.4.3.RELEASE&lt;/version&gt;
	&lt;/parent&gt;
</pre>

<h4> Step 3 : Exclude the embedded container from &#8216;war&#8217;</h4>

<p>As we will be deploying the WAR to external container, we don&#8217;t want the embedded container to be present in war in order to avoid any interference. Just mark the embedded container dependency as &#8216;provided&#8217;.</p>

<pre class="brush: xml; title: ; notranslate" title="">
&lt;dependency&gt;
                    &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
                    &lt;artifactId&gt;spring-boot-starter-tomcat&lt;/artifactId&gt;
                    &lt;scope&gt;provided&lt;/scope&gt;
                &lt;/dependency&gt;
</pre>

<p>Now you can build <strong>[mvn clean package]</strong> and deploy the war to your external container.</p>

<p>It will produce WAR which you can simply deploy in external container. In this post, we are deploying it in tomcat 8.0.21, by putting the resultant war file in <u>webapps</u> folder and starting the tomcat[<u>bin/startUp.bat</u>].</p>


<h2>Deploying to Docker</h2>

<p>Need tomcat image using which we will be deploying our application. Go to dockerhub and search tomcat</p>

<p>Go to the spring boot project folder and create a docker file:</p>

<pre>
From tomcat:8.0.51-jre8-alpine
CMD ["catalina.sh","run"]
</pre>

<p>Open the terminal and install the docker</p>

<pre>yum install docker</pre>

<p>Start the docker</p>

<pre>systemctl start docker<pre>

<p>Open the terminal and go to the Spring Boot project folder</p>

<p>Build an image with the name producer</p>

<pre>docker image build -t springBootApplication-producer .</pre>

<p>Run the image as a container</p>

<pre>docker container run -p 8080:8080 -d springBootApplication-producer<pre>

<p>The docker internal port 8080 to our external port 8080. Now go to localhost:8080 and we can see that tomcat has started successfully</p>

<p>Go inside the docker and investigate it using the docker exec command</p>

<pre>docker container exec -it 03 /bin/sh<pre>

<li>cd webapps/</li>

<p>Need to copy WAR file in the webapps folder of the tomcat inside the docker. This we will do it adding commands to the docker file</p>

<pre>
From tomcat:8.0.51-jre8-alpine
RUN rm -rf /usr/local/tomcat/webapps/*
COPY ./target/springBootApplication-producer-0.0.1-SNAPSHOT.war /usr/local/tomcat/webapps/ROOT.war
CMD ["catalina.sh","run"]
</pre>

<p>Stop the docker container we had previously started</p>

<pre>docker container stop 03</pre>

<p>Again build the image, and start the container</p>

<p>Go to localhost:8080/YOUR-API-END-POINT, we will see that our application is deployed successfully</p>
