Recently, I needed to have a custom self-hosted maven repository for .jar libraries that I had and are not in the Maven Repository or any other public repository. I had to manually install the library to my local repo using the following maven command:

mvn install:install-file -Dfile=<Name of jar>.jar -DgroupId=my.group.id -DartifactId=the-artifact -Dversion=1.0.1 -Dpackaging=jar
The issue with this is I had to run this command on every server that needed to compile the application. Also for a project with multiple developers, each developer has to run the command to include the dependency in their local repository.

Enter Reposilite
So I searched for a solution but the solutions were either expensive or too complicated. I just needed a simple one that I can host on my server and have full control over.

I stumbled upon Reposilite, a lightweight repository manager for maven artifacts, as described in their site. And truly, it is lightweight! The jar file’s size is around 23MB and that’s it. That’s all you need!

This tutorial will take you step by step from installation to deployment of your artifacts. Let’s begin, shall we.

Installation
Download the .jar file from their Github and place it in a folder in your server. Ensure you place it inside some empty folder since Resposilite will generate files and folders in the location where the .jar is placed.

Run the jar file. By default the reposilite application runs on port 80. Since port 80 is common and there is a high probability of it bound to another application, it’s best if you change the default port to something else in the reposilite.cdn config file. The config file is generated on first run. You can also specify a port using the following command:

java -Xmx32M -Dreposilite.port=8080 -jar reposilite.jar
When the applications starts, it provides a Command Line Interface (CLI) to interact with the application. Just type help to view a list of commands available.

Authorization
The first step is to create an access token. The elaborate guide provided in their site shows how to manage different users but for simplicity we will create a single user: admin.

To create the user run the following command in the CLI:

keygen <path> <alias> [<permissions>]
e.g.
keygen / admin m
This will create a new access token for admin with full access permissions.

Use these credentials to login to the application via a web browser on http://localhost:8080. You should see something like this:


Reposilite Default Page
Click on the feather icon on the right and that should take you to the login page. Enter the alias and token (shown on the command line interface after generation).

Deployment
For the deployment phase, I will show you in two ways:

Deploy a jar file
Deploy a project using mvn deploy command
1. Deploy a jar file
This one is relatively easy.


Select the repository from the drop down (releases or snapshot). Enter the groupId, artifactId and version. Drop the jar file in the file drop area and click Generate stub pom file for this artifact if you don’t have the .pom (yes I said .pom and not pom.xml) file and click upload. That’s it.

2. Deploy a project using mvn deploy command
Say you have your own library that you are creating and you need to add as a dependency in your other projects. In your pom.xml file, you need to add the following:

<distributionManagement>
    <repository>
        <id>my-repository</id>
        <url>http://localhost:8080/releases</url>
    </repository>
</distributionManagement>
This points maven on where the artifact should be deployed.

Also add the following to you ~/m2/settings.xml :

<server>
  <!-- Id has to match the id provided in pom.xml -->
  <id>my-repository</id>
  <username>{alias}</username>
  <password>{token}</password>
</server>
Remember alias and token are the ones you generated in the authorization step.

If everything has been setup correctly, go ahead and run the maven command:

mvn deploy
This will deploy your artifact to the repository.

To use the dependencies in your other projects, create the dependency in you pom.xml and add your custom repository as a repository entry in the pom.xml

<dependencies>
    <dependency>
        <groupId>com.mydomain</groupId>
        <artifactId>my-artifact</artifactId>
        <version>1.0.1</version>
    </dependency>
</dependencies>
<repositories>
<!--Your custom maven repo-->
    <repository>
        <id>my-repository</id>
        <name>My custom maven repo</name>
        <url>http://localhost:8080</url>
    </repository>
</repositories>
… and voila, you have your self hosted custom maven repository!!!

Conclusion
We have seen how to create a custom self hosted maven repository using Reposilite. Reposilite is really lightweight but powerful and I would encourage you to visit their documentation page for further configuration information.

57



