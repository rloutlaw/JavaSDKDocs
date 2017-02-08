# Azure SDK for Java

The Java SDK for Azure is a set of libraries to create and manage resources in Microsoft Azure.

For an overview of Azure and Java, visit the [Java developer center for Azure](https://azure.microsoft.com/en-us/develop/java)

## Install the complete SDK

Add the complete SDK into your projects by updating your [Maven](https://maven.apache.org) pom.xml or [Gradle](https://gradle.org) build.gradle file. 

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure</artifactId>
    <version>1.0.0-beta5</version>
</dependency>
```

```json
    dependencies {
    compile 'com.microsoft.azure:azure:1.0.0-beta5'
}
```

## Install individual libraries

Add a dependency for specific Azure services and features if you don't want to use the complete SDK package in your project.

### Azure services

Key Vault - Securely store secrets and keys.
SQL Database - Set up and configure a SQL database.
DocumentDB - Create NoSQL databases with JSON data.
Redis Cache - High throughput, in-memory key/value cache for your apps.




### Azure resource management
Content Delivery Network - Provide lower latency and high bandwidth for content no matter where your users are.


### Azure service management
App Service

