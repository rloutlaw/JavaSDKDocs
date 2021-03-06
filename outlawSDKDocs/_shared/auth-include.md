Create an [authentication file](https://github.com/Azure/azure-sdk-for-java/blob/master/AUTH.md) and export an environment variable `AZURE_AUTH_LOCATION` on the command line with the full path to the file.

```bash
export AZURE_AUTH_LOCATION=/Users/raisa/azure.auth
```

The authentication file is used to configure the entry point `Azure` object used by the management libraries to define, create, and configure Azure resources.

```java
// pull in the location of the security file from the environment 
final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));

Azure azure = Azure
        .configure()
        .withLogLevel(LogLevel.NONE)
        .authenticate(credFile)
        .withDefaultSubscription();
```

[Learn more](ceoncepts.md#authentication) about authentication options  when using the Azure management libraries for Java.