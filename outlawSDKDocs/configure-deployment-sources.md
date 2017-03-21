---
title: Configure Azure App Service deployment sources | Microsoft Docs
description: Sample Java code to configure Azure App Service deployment using FTP, Git, or continuous deployment
services: ''
documentationcenter: java
author: routlaw
manager: douge
editor: ''

ms.assetid: a90d2905-44eb-471e-a4a8-d52463cec72f
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: multiple
ms.devlang: Java
ms.topic: article
ms.date: 12/22/2016
ms.author: routlaw;asirveda

---

# Configure Azure App Service deployment sources with Java

[This sample](https://github.com/Azure-Samples/compute-java-create-virtual-machines-across-regions-in-parallel) creates four applications in a single [Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/) plan and pushes code to each of them using different deployment sources.

## Run the sample

To run the sample, create an [authentication file](https://github.com/Azure/azure-sdk-for-java/blob/master/AUTH.md) and set an environment variable `AZURE_AUTH_LOCATION` with the full path to the file on your computer. Then run:

```bash
git clone https://github.com/Azure-Samples/app-service-java-configure-deployment-sources-for-web-apps.git

cd app-service-java-configure-deployment-sources-for-web-apps

mvn clean compile exec:java
```

## Sample code

View the [complete sample code on GitHub](https://github.com/Azure-Samples/app-service-java-configure-deployment-sources-for-web-apps/blob/master/src/main/java/com/microsoft/azure/management/appservice/samples/ManageWebAppSourceControl.java)

### Authenticate with Azure

[!INCLUDE [auth-include](_shared/auth-include.md)]

### Create a App Service app with running on Tomcat

```java
// create a new S1 plan and create a single Java 8/Tomcat 8 app in the plan
WebApp app1 = azure.webApps().define(app1Name)
             .withNewResourceGroup(rgName)
             .withNewAppServicePlan(planName)
             .withRegion(Region.US_WEST)
             .withPricingTier(AppServicePricingTier.STANDARD_S1)
             .withJavaVersion(JavaVersion.JAVA_8_NEWEST)
             .withWebContainer(WebContainer.TOMCAT_8_0_NEWEST)
             .create();
```

### Deploy a Java application using FTP
```java

// pass the PublishingProfile that contains FTP credentials to a helper method that uploads a WAR file
// on the filesystem to the /site/wwwroot/webapps folder in App Service. Tomcat will automatically
// deploy WAR files uploaded to this directory.
uploadFileToFtp(app1.getPublishingProfile(), "helloworld.war", ManageWebAppSourceControl.class.getResourceAsStream("/helloworld.war"));

// this method uses the FTP classes in the Apache Commons library to connect to Azure using the username and password
// in the PublishingProfile object
private static void uploadFileToFtp(PublishingProfile profile, String fileName, InputStream file) throws Exception {
        FTPClient ftpClient = new FTPClient();
        String[] ftpUrlSegments = profile.ftpUrl().split("/", 2);
        String server = ftpUrlSegments[0];
        String path = "./site/wwwroot/webapps";
        ftpClient.connect(server);
        ftpClient.login(profile.ftpUsername(), profile.ftpPassword());
        ftpClient.setFileType(FTP.BINARY_FILE_TYPE);
        ftpClient.changeWorkingDirectory(path);
        ftpClient.storeFile(fileName, file);
        ftpClient.disconnect();
}
```

This code uploads a WAR file to the `/site/wwwroot/webapps` directory for the app in App Service. Tomcat is configured to monitor and
deploy WAR files in this directory by default in App Service.

### Deploy a Java application in a local Git repo

```java
PublishingProfile profile = app2.getPublishingProfile();
// create a new Git repo in the helloworld sample directory in src/main/resources and add all files into a new commit
Git git = Git
    .init()
    .setDirectory(new File(ManageWebAppSourceControl.class.getResource("/azure-samples-appservice-helloworld/").getPath()))
    .call();
git.add().addFilepattern(".").call();
git.commit().setMessage("Initial commit").call();
// push the commit using the Azure Git remote URL and credentials in the publishing profile
PushCommand command = git.push();
command.setRemote(profile.gitUrl()); 
command.setCredentialsProvider(new UsernamePasswordCredentialsProvider(profile.gitUsername(), profile.gitPassword()));
command.setRefSpecs(new RefSpec("master:master")); 
command.setForce(true);
command.call();
```      

This code uses the [JGit](https://eclipse.org/jgit/) libraries to create a new Git repo in the `src/main/resources/azure-samples-appservice-helloworld` directory, add all the files to an initial commit, then push the commit to Azure using the Git remote URL, username and password from the `PublishingProfile` object. 

>[!NOTE]
> The layout of the files in the repo must match exactly how you want the files deployed under the `/site/wwwroot/` directory on the Tomcat instance. For example, you cannot push a Maven project with Git deployment without committing outputs from your local build into the appropriate location in your repo first.

### Deploy an application from a public Git repo

```java
// create a new webapp in App Service and deploy a .NET sample app from a public GitHub repo
WebApp app3 = azure.webApps().define(app3Name)
                    .withNewResourceGroup(rgName)
                    .withExistingAppServicePlan(plan)
                    .defineSourceControl()
                        .withPublicGitRepository("https://github.com/Azure-Samples/app-service-web-dotnet-get-started")
                        .withBranch("master")
                        .attach()
                    .create();
 ```

### Continuous deployment from a GitHub repo

```java
// create a new webapp and deploy your code when your master branch is updated
WebApp app4 = azure.webApps()
                    .define(app4Name)
                    .withExistingResourceGroup(rgName)
                    .withExistingAppServicePlan(plan)
                    // Uncomment the following lines to turn on 4th scenario
                    //.defineSourceControl()
                    //    .withContinuouslyIntegratedGitHubRepository("username", "reponame")
                    //    .withBranch("master")
                    //    .withGitHubAccessToken("YOUR GITHUB PERSONAL TOKEN")
                    //    .attach()
                    .create();
```  

The `username` value in this code is your GitHub username. You'll need to [create a GitHub personal access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/) with read permissions and pass it to `withGitHubAccessToken`. 


## Sample explanation

The sample creates the first application using Java 8 and Tomcat 8 running in a newly created [Standard S1 plan](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview) . The `PublishingProfile` for the newly created app is retieved and used to get the hostname, username, and password for FTP deployment. The code then uploads a WAR file to the directory Tomcat is configured to monitor for new deployments. 

The second application is created in the same plan as the first and is also configured as a Java 8/Tomcat 8 application. The JGit libraries are used to create a new Git repository in a folder of the sample that contains an unpacked Java web application in a directory structure that maps to the one in App Service. The files in the folder are committed to the repo and pushed to Azure using a Git remote URL and username/password provided by the `PublishingProfile`.

The third application is created in the same plan, but is not configured for Java and Tomcat. Instead, a .NET sample is deployed directly from the source in a public GitHub repo.

The fourth application pulls your own application from your GitHub repo and will deploy the code in your master branch every time it is updated. 

| Class used in sample | Notes
|-------|-------|
| [com.microsoft.azure.management.appservice.WebApp](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.appservice._web_app) | Created from the `azure.webApps().define()....create()` fluent chain. Creates a App Service web app and any resources needed for the app. Can be queried for properties and the state of the application can be changed through methods like `restart()`.
| [com.microsoft.azure.management.appservice.WebContainer](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.appservice._web_container) | Class with static public fields used as paramters to `withWebContainer()` when defining a WebApp running a Java webcontainer. Has choices for both Jetty and Tomcat versions.
| [com.microsoft.azure.management.appservice.PublishingProfile](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.appservice._publishing_profile) | Obtained through a WebApp object using the `getPublishingProfile()` method. Contains FTP and Git deployment information, including deployment username and password (which is separate from Azure account or service principal credentials).
| [com.microsoft.azure.management.appservice.AppServicePlan](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.appservice._app_service_plan) | Returned by `azure.appServices().appServicePlans().getbyGroup()`. Methods are availble to check the capacity, tier, and number of web apps running in the plan.
| com.microsoft.azure.management.appservice.AppServicePricingTier | Class with static public fields representing App Service tiers. Used to define a plan tier in-line during app creation with `withPricingTier()` or directly when defining a plan via `azure.appServices().appServicePlans().define()`
| [com.microsoft.azure.management.appservice.JavaVersion](https://docs.microsoft.com/en-us/java/api/com.microsoft.azure.management.appservice._java_version) | Class with static public fields representing Java versions supported by App Service. Used with `withJavaVersion()` during the `define()...create()` chain when creating a new webapp.

## Next steps

[!INCLUDE [next-steps](_shared/next-steps.md)]

Additional Azure storage samples can be found in the [Azure App Service documentation](https://docs.microsoft.com/en-us/azure/app-service/).