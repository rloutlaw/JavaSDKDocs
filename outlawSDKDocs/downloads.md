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

```json {
    dependencies {
    compile 'com.microsoft.azure:azure:1.0.0-beta5'
}
```

## Install individual libraries

Add support for specific Azure services and features 

### Maven

Add a depdendency entry in your Maven pom.xml to use a library in your project.

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-storage</artifactId>
    <version>5.0.0</version>
</dependency>
```

### Gradle 

Add an entry for an external module in the depdencies block of your build.gradle file to use a library in your project.

```json
dependencies {
    compile group: 'com.microsoft.azure', name: 'azure-storage', version: '5.0.0'
}
```
## 

