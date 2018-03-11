# Spring Boot multi-package demo

This project demonstrates the approach to make a Spring Boot application packageable into multiple deployment artifacts in a single build.

For example, you might have a requirement to support deployment both as an executable JAR and a traditional WAR archive that is deployable on Tomcat and WildFly. Furthermore, WAR archives might have application server specific deployment constraints which requires you to produce multiple specific WAR archives.

The application used for demo purposes is a simple Spring Boot web application built by either Gradle or Maven. To illustrate the problem of application server specific deployment constraints the demo application uses [WebJars](https://www.webjars.org/) and its `webjars-locator` module. To be able to run on WildFly, such configuration requires JBoss/WildFly specific [`webjars-locator-jboss-vfs`](https://github.com/webjars/webjars-locator-jboss-vfs) module to support the VFS protocol. This module naturally causes problem if the application is deployed to e.g. Tomcat, hence the need for multiple WAR archives.

## Solution

The proposed solution demonstrated by this project is based on splitting the project into multiple modules. The `demo-core` module is central piece which contains the actual implementation, while several other _package_ modules are responsible for packaging application into a respective deployment artifact. These modules are:

 - `demo-package-jar`: builds an executable JAR
 - `demo-package-war`: builds a traditional WAR archive for deployment in e.g. Tomcat and TomEE.
 - `demo-package-war-wildfy`: builds a WildFly specific WAR archive

The important part is that the `demo-core` module should not have Spring Boot’s Gradle or Maven plugin applied but rather only using Spring Boot for dependency management.

With Gradle, [Dependency management plugin](https://github.com/spring-gradle-plugins/dependency-management-plugin) should be used together with import of Spring Boot's BOM (this happens automatically if Boot’s Gradle Plugin is applied):

```gradle
subprojects {
	apply plugin: 'java'
	apply plugin: 'io.spring.dependency-management'

	// ...

	dependencyManagement {
		imports {
			mavenBom "org.springframework.boot:spring-boot-dependencies:$springBootVersion"
		}
	}
}
```

_Package_ modules then simply depend on `demo-core` module and apply the specific configuration (for example, to enable WAR packaging) and specific dependencies (such as `webjars-locator-jboss-vfs` from introduction).

The resulting build (executed using `./gradlew build` or `./mvnw package`) then generates 3 deployment artifacts.
