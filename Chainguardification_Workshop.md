# Chainguard Hands-on Workshop

### 1. CONNECT TO EC2 INSTANCE
Each participant will be provided with a .pem file and the ip address of their AWS EC2 instance.
```
ssh -i <PATH_TO_CERT>/chainguard-workshop.pem ec2-user@<YOUR_SERVER_IP>
```
<br/><br/>
### 2. JOIN OUR CHAINGUARD ORGANISATION
Check your email and accept the invitation to join our Chainguard organisation.



<br/><br/>
### 3. INSTALL CHAINCTL & GRYPE
Install Chainctl
(https://edu.chainguard.dev/chainguard/chainctl-usage/how-to-install-chainctl/)
```
sudo curl -o chainctl "https://dl.enforce.dev/chainctl/latest/chainctl_$(uname -s | tr '[:upper:]' '[:lower:]')_$(uname -m | sed 's/aarch64/arm64/')"
sudo install -o $UID -g $(id -g) -m 0755 chainctl /usr/local/bin/
```
Install Grype
(https://github.com/anchore/grype)
```
curl -sSfL https://get.anchore.io/grype | sudo sh -s -- -b /usr/local/bin
```
<br/><br/>
### 4. BEGIN WORKSHOP
For the purposes of this workshop we'll run everything as the root user.
** This is NOT recommended for production environments **
```
sudo -i
```
```
cd /opt
```
Enter Chainguard partner console (https://edu.chainguard.dev/chainguard/chainguard-images/how-to-use/images-directory/)
<br/><br/>
### 5. SET CHAINGUARD ORG
Set the chainguard partner workshop Org as default:
```
chainctl config set default.group somerfordassociates.com-partner
```
<br/><br/>
### 6. AUTHENTICATE
If you accepted your Chainguard Org invitation and used Google, Github or Gitlab, use the following command to authenticate with Chainguard.
```
chainctl auth login --headless
```
Copy and paste the link into a web browser and login.  Once logged in a confirmation will be shown in the terminal.
<br/>
If you used Email, Organisation name or have an Identity Provider ID, use the following command:
```
chainctl auth login
```
<br/><br/>
### 7. IMAGE REPOS
List your partner Org repos:
```
chainctl images repos list --parent=somerfordassociates.com-partner
```
Should show something like this:

[cgr.dev/somerfordassociates.com-partner]\
├ [adoptium-jdk]\
├ [adoptium-jre]\
├ [jdk]\
├ [jre]\
├ [kafka-iamguarded]\
...

List `python` image tags that we can use:
```
chainctl images list --repo=python
```

Should show something like this:

[cgr.dev/somerfordassociates.com-partner]\
└ [python]\
  ├ sha256:023bd1612ce4a8f9b3d41a86047d0ab32141806c3c65ab36ee7c78ad06cc1648\
  │ ├ [3.9]\
  │ └ [3.9.25]\
  ├ sha256:03059af9839592d2ec7d31b0271db88f67ee3cb875ca8bfd3ba36b1d4c8de52c\
  │ ├ [3.12]\
  │ └ [3.12.12]\
...
<br/><br/>
### 8. LIBRARY ENTITLEMENTS
List your partner Org library entitlements:
```
chainctl libraries entitlements list --parent=somerfordassociates.com-partner
```
Should show something like this:\
                            ID                             | ECOSYSTEM  \
-----------------------------------------------------------|------------\
 547c9d93f34ada9c0f59aa998aa3334aca10f7d7/2b16fc4ebdf97777 | JAVASCRIPT \
 537c8d93f34ada9c0f59aa998aa2234aca10f7d7/37e80f06263e0882 | JAVA\
 667c8d96f34ada9c0f66aa666aa2234aca10f7d7/4a3dbce6fc6d2a2b | PYTHON
 
This means that we have access to [Chainguard libraries](https://www.chainguard.dev/libraries) for Java, JavaScript and Python.
<br/><br/>
### 9. APP - THE STANDARD WAY
Using example [Spring Boot app] (https://github.com/spring-guides/gs-spring-boot/tree/main/initial)

```
git clone https://github.com/spring-guides/gs-spring-boot.git
cd gs-spring-boot/initial
```
Create the `.dockerignore` file inside `/opt/gs-spring-boot/initial` with following:
```
cat << EOF > /opt/gs-spring-boot/initial/.dockerignore
Containerfile*
EOF
```
This should make builds faster. Changes in Containerfiles have no impact on build context and caching.

Test base image candidate and get infos:
```
docker run -ti --rm docker.io/eclipse-temurin:17-jdk-ubi9-minimal /bin/bash
```

Execute inside the image:
```
whoami
```
```
pwd
```
```
java -version
```

As we see, this image runs as user `root` inside root `/` and has java version `21`.

CTRL+D to exit the container
<br/><br/>
Create the `Containerfile.jdk` file inside `/opt/gs-spring-boot/initial` using the following:
```
cat << EOF > /opt/gs-spring-boot/initial/Containerfile.jdk
FROM docker.io/eclipse-temurin:17-jdk-ubi9-minimal AS build
# user: root!
COPY . .
RUN ./gradlew clean build

FROM docker.io/eclipse-temurin:17-jre-ubi9-minimal
COPY --from=build /build/libs/spring-boot-0.0.1-SNAPSHOT.jar /
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/spring-boot-0.0.1-SNAPSHOT.jar"]
EOF
```
This is a multistage build using a JDK to build the app and a JRE container to run it.

Build image (includes app build):
```
docker build -t my-spring-jdk -f Containerfile.jdk .
```
Attention: Clean local build and .gradle files if you encounter file permission problems.

Test app inside container:
```
docker run -ti -p 8080:8080 --entrypoint /bin/bash my-spring-jdk
```
```
java -jar /spring-boot-0.0.1-SNAPSHOT.jar
```

Open your browser at http://<YOUR_SERVER_IP>:8080

You should see: `Greetings from Spring Boot!`

CTRL+C to close the app\
CTRL+D to exit the container

<br/><br/>
### 10. APP - THE CHAINGUARD WAY
Go to the [Chainguard console](https://console.chainguard.dev/) and search for a Java image.

The `Organization` tab shows all images that you can use at your partner organisation.
The `Chainguard catalog` tab shows all Chainguard Images. You may ask Chainguard to provide you with images from our catalog so that they are available in your org.

Test Chainguard base image candidate and get info:

Username and password will be provided by the presenter.
```
docker login "cgr.dev" --username "############" --password "#############"
docker run -ti --rm cgr.dev/somerfordassociates.com-partner/jdk:openjdk-17 /bin/sh
```
Execute inside the image:
```
whoami
```
```
pwd
```
```
java -version
```

This image runs as user `java` inside `/home/build` and has java version 17.
Running as root in production is a no-go. OpenShift will also prevent the container from starting.

CTRL+D to exit the container

Change the greetings inside `/opt/gs-spring-boot/initial/src/main/java/com/example/springboot/HelloController.java` to `Greetings from Chainguard!`.
```
vi /opt/gs-spring-boot/initial/src/main/java/com/example/springboot/HelloController.java
```

Create the `Containerfile.cg` file inside `/opt/gs-spring-boot/initial` with following content:
```
cat << EOF > /opt/gs-spring-boot/initial/Containerfile.cg
FROM cgr.dev/somerfordassociates.com-partner/jdk:openjdk-17 AS build
COPY . .
RUN ./gradlew clean build

FROM cgr.dev/somerfordassociates.com-partner/jre:openjdk-17
COPY --from=build /home/build/./build/libs/spring-boot-0.0.1-SNAPSHOT.jar /home/build/
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/home/build/spring-boot-0.0.1-SNAPSHOT.jar"]
EOF
```
This is also a multistage build using a Chainguard JDK image to build the app and a Chainguard JRE container to run it.

Build:
```
docker build -t my-spring-cg -f Containerfile.cg .
```

Attention: Clean local build and .gradle files if you encounter file permission problems.

Test app inside container:
```
docker run -ti -p 8080:8080 --entrypoint /bin/sh my-spring-cg
java -jar ./build/libs/spring-boot-0.0.1-SNAPSHOT.jar
```

You should get an error. The entrypoint differs from the dockerfile.
The images are minimal and hardened. This to keep the attack surface as small as possible.

Run container:
```
docker run -p 8080:8080 my-spring-cg
```

Open your browser at http://<YOUR_SERVER_IP>:8080/

You should see: `Greetings from Chainguard!`

CTRL+C to close the app
<br/><br/>
### 11. GET LIBRARIES

Now we'll also use Chainguard Libraries.

(https://edu.chainguard.dev/chainguard/libraries/access/) states following to get access to the [Java libraries](https://edu.chainguard.dev/chainguard/libraries/java/overview/):

```
chainctl auth pull-token --repository=java --parent=somerfordassociates.com-partner --ttl=4h
```

We will use the eval command that will set the required environment variables:
```
eval $(chainctl auth pull-token --output env --repository=java --parent=somerfordassociates.com-partner)
```

Check the Gradle build configuration (`/opt/gs-spring-boot/initial/build.gradle`) for dependencies.

Chainguard provides the `spring-boot-starter-web` library:

https://libraries.cgr.dev/java/org/springframework/boot/spring-boot-starter-web/


Now we have to configure Gradle to use Chainguard libraries that are available through the Chainguard repository: `https://libraries.cgr.dev/java/`

Extend the file `/opt/gs-spring-boot/initial/build.gradle` to get Chainguard libraries:
```
vi /opt/gs-spring-boot/initial/build.gradle
```

```
repositories {
	maven {
		url = uri("https://libraries.cgr.dev/java/")
		credentials {
			username = "CHAINGUARD_JAVA_IDENTITY_ID"
			password = "CHAINGUARD_JAVA_TOKEN"
		}
	}
	mavenCentral()
}
```

Official documentation: https://edu.chainguard.dev/chainguard/libraries/java/build-configuration/#gradle

We will build the application outside of the container.
We need to clean the Gradle cache.

Clean the Gradle cache:
```
rm -rf .gradle/caches/
rm -rf ~/.gradle/caches/
```

Build the app:
```
./gradlew clean build
```

Run the app:
```
java -jar build/libs/spring-boot-0.0.1-SNAPSHOT.jar 
```

Open your browser at http://<YOUR_SERVER_IP>:8080

You should see: `Greetings from Chainguard!`

CTRL+C to close the app


Now we can verify the usage of Chainguard libraries inside our Java app:
```
chainctl libraries verify --detailed --parent=somerfordassociates.com-partner ./build/libs/spring-boot-0.0.1-SNAPSHOT.jar
```

Official documentation: https://edu.chainguard.dev/chainguard/libraries/verification/
<br/><br/>
### 12. SCAN IMAGES

We will scan and compare both images.

Check the Grype tutorial for using the Grype container: https://edu.chainguard.dev/chainguard/chainguard-images/staying-secure/working-with-scanners/grype-tutorial/

Scan both images to compare the vulnerabilities:

```
grype my-spring-jdk:latest
```
```
grype my-spring-cg:latest
```

What are the differences?
<br/><br/>
### 13. HOW-TO AND LINKS

* [Chainguard Console](https://console.chainguard.dev/)
* [Chainguard Containers how to use](https://edu.chainguard.dev/chainguard/chainguard-images/how-to-use/how-to-use-chainguard-images/)
* [chainctl Reference](https://edu.chainguard.dev/chainguard/chainctl/)


