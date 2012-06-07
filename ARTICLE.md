# Social Enterprise Java Apps on Heroku

Last week I co-presented a webinar about building Social Enterprise Java Apps on Heroku.  Here is the recording from the webinar:
<iframe width="640" height="480" src="http://www.youtube.com/embed/GRLzYwtzWqU?rel=0" frameborder="0" allowfullscreen></iframe>

During the webinar I showed how to create a Java web application that integrates with Salesforce.com / Force.com using the REST APIs.  In that example I started from a Tomcat + Hibernate + Spring MVC template application and then migrated the application from Hibernate to Force.com for persistence.  I'd like to take the end result of that migrated app and show how you can get started deploying Social Enterprise Java apps on Heroku.  Here are the steps we will follow:

1. Copy the `hello-java-spring-force_dot_com` project into Eclipse
2. Setup OAuth on Salesforce.com
3. Test the app locally
4. Deploy the app on Heroku

After we've completed those steps I will walk through the major pieces of the application:

* Dependencies
* Spring Config for OAuth
* Spring Security
* `Person` Value Object
* Data Access via the `ForceAPI`

Before you get started you will need to setup a few things:

1. Install the [Heroku Toolbelt](https://toolbelt.herokuapp.com)
2. [Signup for a Heroku Account](https://heroku.com/signup)
3. Install [Eclipse 3.7 for Java EE Developers](http://eclipse.org/downloads)
4. Install the EGit Eclipse Plugin

        1. In Eclipse select the "Help" menu
        2. Select "Eclipse Marketplace..."
        3. Search for "EGit"
        4. Select "Install" on the "EGit - Git Team Provider" item
        5. Complete the installation wizard

5. Install the Maven Eclipse Plugin

        1. In Eclipse select the "Help" menu
        2. Select "Eclipse Marketplace..."
        3. Search for "Maven"
        4. Select "Install" on the "Maven Integration for Eclipse" item
        5. Complete the installation wizard

6. Login to Heroku and setup SSH keys

        1. Open a command line or Terminal prompt and run:

                heroku login

        2. Enter the email address for your Heroku account and your Heroku password
        3. Follow the prompts to create (if needed) and setup a SSH key for git authentication

7. [Signup for a Salesforce.com Developer Edition Account](http://developer.force.com/join)


## Running the `hello-java-spring-force_dot_com` Demo Application

Now that your environment is all setup, lets grab a copy of the `hello-java-spring-force_dot_com` demo application by pulling it into Eclipse using EGit.

### Copy the `hello-java-spring-force_dot_com` project into Eclipse

1. In Eclipse select the "File" menu
2. Select "Import..."
3. Under the "Git" item, select "Projects from Git"
4. Select "Next"
5. Select "URI" and "Next"
6. Enter the following into the "URI" field:

        git://github.com/jamesward/hello-java-spring-force_dot_com.git

7. Select "Next"
8. Select "Next" again
9. Select "Import as general project" and take note of the directory Eclipse selects for the working directory
10. Select "Next"
11. Select "Finish"

You should now see the `hello-java-spring-force_dot_com` project in the Eclipse Project Explorer view:
![Eclipse Project Explorer](https://github.com/jamesward/hello-java-spring-force_dot_com/raw/master/img/project_explorer.png)

Now the project needs to be setup for use with Maven.  Maven will manage the dependencies for the project.

1. Right-click on the project in Eclipse
2. Select "Configure"
3. Select "Convert to Maven Project"

The project dependencies will be downloaded and the project will be configured as a web project using the dependencies specified by the Maven build.

### Setup OAuth on Salesforce.com

In order to run the application locally we will need to configure OAuth on Salesforce.com.  OAuth will be used to authenticate REST requests from the application based on a handshake between Salesforce.com and the application.  OAuth is the preferred approach to integrating with Salesforce.com because it does not require storage of user credentials and it allows the user to revoke access from an application.

1. Login to [Salesforce.com](http://salesforce.com)
2. Select your name in the top right and select "Setup"
3. On the left, expand "Develop" and select "Remote Access"
4. Select "New" to create a new Remote Access Application
5. In the "Application" field specify "local-spring"
6. Specify your email address in the "Contact Email" field
7. Enter `http://localhost:8080/_auth` in the "Callback URL" field
8. Select "Save"
9. Leave the "Remote Access Detail" page open because shortly you will need some information from it

### Test the app locally

Now that OAuth is configured on Salesforce.com we can run this application locally to test it.

1. In Eclipse select the "Run" menu
2. Select "Run Configurations..."
3. Double-click on "Java Application" to create a new Run Configuration
4. In the "Name" field specify `hello-java-spring-force_dot_com`
5. In the "Main Class" field specify `webapp.runner.launch.Main`
    ![New Run Configuration](https://github.com/jamesward/hello-java-spring-force_dot_com/raw/master/img/run_config.png)
6. Select the "Arguments" tab 
7. Enter "src/main/webapp" in the "Program Arguments" field
8. Select the "Environment" tab
9. Select "New"
10. Enter `OAUTH_CLIENT_KEY` in the "Name" field
11. Retrieve the "Consumer Key" value from the "Remote Access Detail" page on Salesforce.com then copy and paste it into the "Value" field
12. Select "Ok"
13. Select "New"
14. Enter `OAUTH_CLIENT_SECRET` in the "Name" field
15. Retrieve the "Consumer Secret" value from the "Remote Access Detail page then copy and paste it into the "Value" field
16. Select "Ok"
17. Select "Run" to start the application

Now that the application is up and running you can test it in your browser by visiting:  
[http://localhost:8080/](http://localhost:8080/)

The index page is unprotected and should load without having to authenticate.  Now load the "People" page by visiting:  
[http://localhost:8080/people/](http://localhost:8080/people/)

You should now be redirected to Salesforce.com's OAuth handshake page.  Select "Allow" to do the OAuth handshake.  You will then be redirected back to the "People" page which should now display a list of your Salesforce.com contacts.  (Note: Developer Edition accounts have a few contacts out-of-the-box.)  Test that creating and deleting contacts also works.

Now we can deploy the application on the cloud with Heroku.

### Deploy the app on Heroku

1. Navigate in a command line / Terminal window to the directory containing the project
2. Create a new application on Heroku's "Cedar" stack:

        heroku create -s cedar

Your application will be assigned a name that will be used for the default HTTP and Git endpoints.  The name will be something like `severe-sunset-4637` in which case the HTTP endpoint would be `http://severe-sunset-4637.herokuapp.com` and the Git endpoint would be `git@heroku.com:severe-sunset-4637.git`.  This name can be changed and other DNS names can be pointed at the application.

Now that the application has an HTTP endpoint we will need to configure a new Remote Access Application / OAuth configuration.  Like before, create a new configuration on the Salesforce.com - Setup - Develop - Remote Access page.  This time set the "Application" field to the name of your application on Heroku.  Set the "Callback URL" field to `https://your-application-1234.herokuapp.com/_auth` making sure you specify `https` and the correct domain name for your application.  Save the new configuration.

You will now need to set the OAuth environment variables on your Heroku application.  In the command line / Terminal make sure you are in the root directory of your project and run the following commands, but replace the values with the Consumer Key and Consumer Secret values from the new "Remote Access Application" page.

    heroku config:add OAUTH_CLIENT_KEY=YOUR_CONSUMER_KEY
    heroku config:add OAUTH_CLIENT_SECRET=YOUR_CONSUMER_SECRET

Your application is ready to be deployed on Heroku!  Heroku uses Git for file transfer.  When you ran the `heroku create` command the Git endpoint for your application was added to the project's Git configuration.  To upload the application to Heroku's Git endpoint do the following:

1. In Eclipse right-click on the project
2. Select "Team"
3. Select "Remote"
4. Select "Push..."
5. In the drop-down beneath "Configured remote repository" select the remote named "heroku" that contains your applications Git endpoint
6. Select "Finish"

The application will be uploaded to Heroku where the Maven build will be run to download the dependencies and compile the application.  Once the compile has successfully completed the application will be deployed onto a [Heroku Dyno](https://devcenter.heroku.com/articles/dynos) and the application will be started.  Once that process is complete you will see the output in Eclipse:
![Push to Heroku](https://github.com/jamesward/hello-java-spring-force_dot_com/raw/master/img/push_heroku.png)

Now test your application on Heroku by opening the "people" page in your browser using "https" as the protocol.  For instance:  
[https://severe-sunset-4637.herokuapp.com/people/](https://severe-sunset-4637.herokuapp.com/people/)

This should initiate the OAuth handshake again.  After allowing the application you should now see your contacts displayed like before, but this time from the application running on Heroku!

## Code Deep Dive

Now that you have everything working lets walk through how it works.

### Dependencies

First, lets look at the Maven build descriptor `pom.xml` file.  It specifies the dependencies of the application.  There are dependencies for `webapp-runner` (a Tomcat wrapper), Spring MVC for handling web requests, JSP support libraries, the Twitter Bootstrap CSS library, and the Force.com libraries.  Here are the Force.com dependencies:

    <dependency>
        <groupId>com.force.api</groupId>
        <artifactId>force-rest-api</artifactId>
        <version>0.0.18</version>
    </dependency>
    <dependency>
        <groupId>com.force.sdk</groupId>
        <artifactId>force-oauth</artifactId>
        <version>22.0.8-BETA</version>
    </dependency>
    <dependency>
        <groupId>com.force.sdk</groupId>
        <artifactId>force-springsecurity</artifactId>
        <version>22.0.8-BETA</version>
    </dependency>

The `force-rest-api` library is a Java wrapper around the Force.com REST APIs.  You can find more information about that library at:  
[https://github.com/jesperfj/force-rest-api](https://github.com/jesperfj/force-rest-api)

The `force-oauth` and `force-springsecurity` libraries make it easy to setup OAuth with Spring Security and Spring MVC.  You can find more information about those libraries at:  
[http://forcedotcom.github.com/java-sdk/](http://forcedotcom.github.com/java-sdk/)

This demo application uses Maven and Spring but you could certainly use those libraries in any type of JVM-based application.

### Spring Config for OAuth

To configure Spring Security and Spring MVC the `src/main/resources/applicationContext.xml` file contains:

    <?xml  version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:mvc="http://www.springframework.org/schema/mvc"
           xmlns:fss="http://www.salesforce.com/schema/springsecurity"
           xmlns:security="http://www.springframework.org/schema/security"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
                               http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd
                               http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd
                               http://www.salesforce.com/schema/springsecurity http://www.salesforce.com/schema/springsecurity/force-springsecurity-1.3.xsd
                               http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security-3.0.xsd">
    
        <context:annotation-config />
        <context:component-scan base-package="com.example" />
        
        <mvc:annotation-driven/>
        <mvc:resources mapping="/public/**" location="classpath:/public/"/>
    
        <bean id="jspViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
            <property name="prefix" value="/WEB-INF/jsp/" />
            <property name="suffix" value=".jsp" />
            <property name="redirectHttp10Compatible" value="false" />
        </bean>
    
        <fss:oauth>
            <fss:oauthInfo endpoint="http://login.salesforce.com"
                           oauth-key="#{systemEnvironment['OAUTH_CLIENT_KEY']}"
                           oauth-secret="#{systemEnvironment['OAUTH_CLIENT_SECRET']}"/>
        </fss:oauth>
        
        <security:http use-expressions="true">
            <security:intercept-url pattern="/people/*" access="isAuthenticated()" />
        </security:http>
    
    </beans>

Most of the Spring config is typical Spring MVC configuration.  The `<fss:oauth>` section sets up the Force.com OAuth functionality using the `OAUTH_CLIENT_KEY` and `OAUTH_CLIENT_SECRET` environment variables which you already used for running the application locally and on Heroku.  The `<security:http use-expressions="true">` section protects requests matching the `/people/*` pattern and insures the requestor has authenticated via OAuth.

### Spring Security

In order to apply Spring Security and Spring MVC to the application, the `src/main/webapp/WEB-INF/web.xml` config file must contain the Spring Servlet Filter and the Spring MVC Dispatcher Servlet:

    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns="http://java.sun.com/xml/ns/javaee"
             xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
             version="2.5">
    
        <display-name>Spring-Hibernate-Template</display-name>
        
        <filter>
            <filter-name>springSecurityFilterChain</filter-name>
            <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
            <init-param>
                <param-name>contextAttribute</param-name>
                <param-value>org.springframework.web.servlet.FrameworkServlet.CONTEXT.spring</param-value>
            </init-param>
        </filter>
        <filter-mapping>
            <filter-name>springSecurityFilterChain</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>
    
        <welcome-file-list>
            <welcome-file>index.jsp</welcome-file>
        </welcome-file-list>
    
        <servlet>
            <servlet-name>spring</servlet-name>
            <servlet-class>
                org.springframework.web.servlet.DispatcherServlet
            </servlet-class>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:applicationContext.xml</param-value>
            </init-param>
            <load-on-startup>1</load-on-startup>
        </servlet>
        
        <servlet-mapping>
            <servlet-name>spring</servlet-name>
            <url-pattern>/people/*</url-pattern>
        </servlet-mapping>
    
        <servlet-mapping>
            <servlet-name>spring</servlet-name>
            <url-pattern>/assets/*</url-pattern>
        </servlet-mapping>
    
    </web-app>

Notice that in this case only the `/people/*` and `/assets/*` paths are mapped to the `DispatcherServlet` while Spring's `DelegatingFilterProxy` is applied all paths.  This example originated from the Tomcat and Spring template on [http://heroku.com/java](http://heroku.com/java) so it follows the conventions from there.  But it's up to you how you configure then path handling.  

### `Person` Value Object 

In order to map `contact` object data from Salesforce.com to a Java object we can use a JSON mapping conventions.  This is how the `force-rest-api` library maps data to a Java value object.  This application has a `Person` class defined in `src/main/java/com/example/model/Person.java` containing:

    package com.example.model;
    
    import org.codehaus.jackson.annotate.JsonIgnoreProperties;
    import org.codehaus.jackson.annotate.JsonProperty;
    
    @JsonIgnoreProperties(ignoreUnknown=true)
    public class Person {
    
        @JsonProperty(value="Id")
        private String id;
    
        @JsonProperty(value="FirstName")
        private String firstName;
    
        @JsonProperty(value="LastName")
        private String lastName;
    
    
        public String getId() {
            return id;
        }
    
        public void setId(String id) {
            this.id = id;
        }
    
        public String getFirstName() {
            return firstName;
        }
    
        public void setFirstName(String firstName) {
            this.firstName = firstName;
        }
    
        public String getLastName() {
            return lastName;
        }
    
        public void setLastName(String lastName) {
            this.lastName = lastName;
        }
    
    }

In addition to the usual Java bean getters and setters, the `@JsonProperty` annotations define how to map from the fields on the `contact` data to the `Person` value object.  For instance, the `contact` object on Salesforce.com has a field named `FirstName` that is mapped to the `firstName` property on the `Person` object.  There is also a `@JsonIgnoreProperties(ignoreUnknown=true)` annotation on the class because we don't want to see an error if unknown properties can't be mapped.

### Data Access via the `ForceAPI`

With everything setup, it's now easy to call the Force.com REST APIs to retreive and change data.  The `PersonServiceImpl` class is essentially the data access object for the `contact` data on Salesforce.com.  Here is it's source from the `src/main/java/com/example/service/PersonServiceImpl.java` file:

    package com.example.service;
    
    import com.force.api.ApiSession;
    import com.force.api.ForceApi;
    import com.force.api.QueryResult;
    import com.force.sdk.oauth.context.ForceSecurityContextHolder;
    import com.force.sdk.oauth.context.SecurityContext;
    import org.springframework.stereotype.Service;
    
    import com.example.model.Person;
    
    import java.util.List;
    
    @Service
    public class PersonServiceImpl implements PersonService {
        
        private ForceApi getForceApi() {
            SecurityContext sc = ForceSecurityContextHolder.get();
    
            ApiSession s = new ApiSession();
            s.setAccessToken(sc.getSessionId());
            s.setApiEndpoint(sc.getEndPointHost());
    
            return new ForceApi(s);
        }
        
        public void addPerson(Person person) {
            getForceApi().createSObject("contact", person);
        }
    
        public List<Person> listPeople() {
            QueryResult<Person> res = getForceApi().query("SELECT Id, FirstName, LastName FROM contact", Person.class);
            return res.getRecords();
        }
    
        public void removePerson(String id) {
            getForceApi().deleteSObject("contact", id);
        }
        
    }

The `getForceApi` method sets up the Force.com REST connection using the OAuth information that Spring Security and the Force.com OAuth library worked together to provide.  The `addPerson` method simply creates a new `contact` object from a `Person` instance.  The `listPeople` method does a SOQL query to get all of the `contact` records and serialize them into `Person` objects.  Finally the `removePerson` method deletes the contact specified by the provided `id` value.  Salesforce.com applies the security to these operations so that a user can't access or change data that they shouldn't have access to.

The user interface for the web application is provided by the `src/main/java/com/example/controller/PersonController.java` Spring MVC Controller and the `src/main/webapp/WEB-INF/jsp/people.jsp` JSP view.  Those are just standard Spring MVC controller and view implementations.

### Further Learning

In the webinar we also demonstrated Chatter integration and Real-time Push with the Pusher add-on.  The source for those projects is at:  
[https://github.com/anandbn/spring-mvc-chatter](https://github.com/anandbn/spring-mvc-chatter)  
[https://github.com/anandbn/chatter-notify-pusher](https://github.com/anandbn/chatter-notify-pusher)

This example is just the basics of running Java applications on Heroku that integrate with Force.com.  If want to learn more about Java on Heroku, check out:  
[https://devcenter.heroku.com/articles/java-learn-more](https://devcenter.heroku.com/articles/java-learn-more)

If you want to learn more about the Force.com REST API, check out:    
[http://www.salesforce.com/us/developer/docs/api_rest/index.htm](http://www.salesforce.com/us/developer/docs/api_rest/index.htm)

Have fun and let us know how it goes integrating your Java apps on Heroku with Force.com!
