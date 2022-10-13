### Flow diagram 
![Image](https://user-images.githubusercontent.com/66588814/192434879-2bb5a9c8-c534-4b6a-9d28-37e7152b415a.png)
###

###
F & Q
- [ ] Everything is fine but i am not able to browse in chrom. (reposilite)
- [ ] How tocreate user in reposilite
- [ ] How to access reposilite console
- [ ] How tolook at server log
- [ ] How will you upload others jars to repository-lite
- [ ] How to install reposilitory-lite as servive

###


Recently, I needed to have a custom self-hosted maven repository for .jar libraries that I had and are not in the Maven Repository or any other public repository. I had to manually install the library to my local repo using the following maven command:

    mvn install:install-file -Dfile=<Name of jar>.jar -DgroupId=my.group.id -DartifactId=the-artifact -Dversion=1.0.1 -Dpackaging=jar

The issue with this is I had to run this command on every server that needed to compile the application. Also for a project with multiple developers, each developer has to run the command to include the dependency in their local repository.

## **Enter Reposilite**

So I searched for a solution but the solutions were either expensive or too complicated. I just needed a simple one that I can host on my server and have full control over.

I stumbled upon  [Reposilite](https://reposilite.com/),  _a lightweight repository manager for maven artifacts,_ as described in their site. And truly, it is lightweight! The jar file’s size is around 23MB and that’s it. That’s all you need!

This tutorial will take you step by step from installation to deployment of your artifacts. Let’s begin, shall we.

## Installation

Download the .jar file from their  [Github](https://github.com/dzikoysk/reposilite/releases)  and place it in a folder in your server. Ensure you place it inside some empty folder since Resposilite will generate files and folders in the location where the .jar is placed.

Run the jar file. By default the reposilite application runs on port 80. Since port 80 is common and there is a high probability of it bound to another application, it’s best if you change the default port to something else in the  **_reposilite.cdn_**  config file. The config file is generated on first run. You can also specify a port using the following command:

java -Xmx32M -Dreposilite.port=8080 -jar reposilite.jar

When the applications starts, it provides a Command Line Interface (CLI) to interact with the application. Just type  _help_ to view a list of commands available.

## Authorization

The first step is to create an access token. The elaborate guide provided in their site shows how to manage different users but for simplicity we will create a single user:  _admin_.

To create the user run the following command in the CLI:

keygen <path> <alias> [<permissions>]e.g.  
keygen / admin m

This will create a new access token for  **_admin_** with  **_full access_**  permissions.

Use these credentials to login to the application via a web browser on http://localhost:8080. You should see something like this:

![](https://miro.medium.com/max/1400/1*msUYAwvuLjHywymWsQSQZg.png)

Reposilite Default Page

Click on the feather icon on the right and that should take you to the login page. Enter the  **_alias_** and  **_token_** (shown on the command line interface after generation).

## Deployment

For the deployment phase, I will show you in two ways:

1.  Deploy a jar file
2.  Deploy a project using  **_mvn deploy_**  command

## 1. Deploy a jar file

This one is relatively easy.

![](https://miro.medium.com/max/1400/1*TEUy-DLyPYQeQgAwAh92hQ.png)

Select the repository from the drop down (releases or snapshot). Enter the groupId, artifactId and version. Drop the jar file in the file drop area and click  **_Generate stub pom file for this artifact_**  if you don’t have the .pom (yes I said .pom and not pom.xml) file and click upload. That’s it.

## 2. Deploy a project using mvn deploy command

Say you have your own library that you are creating and you need to add as a dependency in your other projects. In your  **_pom.xml_** file, you need to add the following:

<distributionManagement>  
    <repository>  
        <id>my-repository</id>  
        <url>http://localhost:8080/releases</url>  
    </repository>  
</distributionManagement>

This points maven on where the artifact should be deployed.

Also add the following to you  **_~/m2/settings.xml :_**

<server>  
  <!-- Id has to match the id provided in pom.xml -->  
  <id>my-repository</id>  
  <username>{alias}</username>  
  <password>{token}</password>  
</server>

Remember  **_alias_** and  **_token_** are the ones you generated in the authorization step.

If everything has been setup correctly, go ahead and run the maven command:

mvn deploy

This will deploy your artifact to the repository.

To use the dependencies in your other projects, create the dependency in you  **_pom.xml_** and add your custom repository as a repository entry in the pom.xml

<dependencies>  
    <dependency>  
        <groupId>com.mydomain</groupId>  
        <artifactId>my-artifact</artifactId>  
        <version>1.0.1</version>  
    </dependency>  
</dependencies><repositories>  
<!--Your custom maven repo-->  
    <repository>  
        <id>my-repository</id>  
        <name>My custom maven repo</name>  
        <url>http://localhost:8080</url>  
    </repository>  
</repositories>

… and voila, you have your self hosted custom maven repository!!!

## FAQ

### How to allow port before starting reposilite
```
sudo ufw allow 2222
```
Output
```
Rule added
Rule added (v6)
```
### How to create first user in reposilite

```
token-generate --secret=admin admin m
```

### Blocked mirror for repositories: [reposilite-repository-releases  default, releases+snapshots)]

Could not transfer artifact github:example-package-1:pom:1.2-SNAPSHOT-sources from/to maven-default-http-blocker (http://0.0.0.0/): Blocked mirror for repositories: [reposilite-repository-releases (http://www6c.virtualdoxx.com:8082/releases, default, releases+snapshots)]
```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0  
 http://maven.apache.org/xsd/settings-1.0.0.xsd">  
  
    <mirrors>  
        <mirror>  
            <id>internal-repository</id>  
            <name>Maven Repository Manager running on https://repo1.maven.org/maven2</name>  
            <url>https://repo1.maven.org/maven2</url>  
            <mirrorOf>*</mirrorOf>  
        </mirror>  
        <mirror>  
            <id>other-mirror</id>  
            <mirrorOf>central</mirrorOf>  
            <name>www6c Mirror Repository</name>  
            <url>http://www6c.virtualdoxx.com:8082/releases</url>  
        </mirror>  
    </mirrors>  
</settings>
```
run command
```
mvn -U clean install --settings settings.xml
```
## Conclusion

We have seen how to create a custom self hosted maven repository using Reposilite. Reposilite is really lightweight but powerful and I would encourage you to visit their documentation page for further configuration information.

https://stackedit.io/app#

[LinkedIn Link Article Link](https://www.linkedin.com/pulse/custom-self-hosted-maven-repository-rupesh-kumar)
    
### [How to make reposilite service thats help to UP the service always](https://github.com/apoorvpandey-ap/How-to-create-a-Systemd-service-in-Linux)     
### installation manaul https://reposilite.com/guide/manual
