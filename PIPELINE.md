#### Building Pipeline

- Overview
```
- Compile Tomcat WAR Test package
- Deploy Staging for QE testing
```

- Follow TOMCAT.md to install tomcat service on staging node.

- Building Artifacts WAR file.
```
Post-build actions -> Archive the artifacts -> File to archive "**/*.war".
Run job now.
Job should have "Last Successful Artifacts" -> "webapp.war" on project page.
Log shows where the location of WAR file.

[INFO] Building war: /var/lib/jenkins/workspace/maven-project/webapp/target/webapp.war
```

- Install "Copy artifacts" + "deploy to container" plugin (install without restart)

- Create a new job "deploy-to-staging"
```
- Build -> Copy artifacts from another project -> Project name "maven-project"
- Artifact to copy -> "**/*.war"

- Post-build actions -> Deploy war/ear to a container -> WAR files "**/*.war".
- Add Containter -> Select Credentials -> Jenkins -> Add "tomcat/tomcat".
- Select tomcat/**** at choice selector.
```

- Setup "maven-project" to trigger "deploy-to-staging" job.
```
- Build other project -> Project to build -> "deploy-to-staging".
- Run "maven-project" upstream job to trigger to "deploy-to-staging" downstream.
```

- Troubleshooting: when deploys to staging.
```
org.codehaus.cargo.container.tomcat.internal.TomcatManagerException: The username you provided is not allowed to use the text-based Tomcat Manager (error 403)
```

- Disable xml statement under /opt/tomcat/webapps/manager/META-INF/context.xml and restart tomcat

https://stackoverflow.com/questions/41675813/the-username-you-provided-is-not-allowed-to-use-the-text-based-tomcat-manager-e
```
<Context antiResourceLocking="false" privileged="true" >
  <!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
  -->
```

- Deployment show up at http://[Staging IP]:8080/webapp/

#### Pipeline Visualization 

- Apply "Build Pipeline Plugin"

- Setup Pipeline dashboard
```
- Click "+" sign 
- Select Layut -> Select Initial Job -> "maven-project"
- Click "run"
```

