---
layout: post
title: "Pushing to Maven Central with Sonatype"
date: 2013-04-10 15:07
comments: true
categories: 
 - Programming
 - Java
 - Maven
---
Getting your opensource JAR published to Maven Central is free, but requires a little bit of up front work. I use [Sonatype](https://sonatype.org) to help publish the jars for [metrics-statsd](https://github.com/organicveggie/metrics-statsd), which makes life much easier.
Most of the process is documented by Sonatype, which you can read about here:

[https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide)

There were a few tricky parts that either weren't documented well or I just found confusing. I have tried to document some of these parts below.
<!-- more -->
# Terminology

In case you're as easily confused as I was, Sonatype uses three different terms when talking about pushing jars:

1. __Deploy__ refers to deploying _snapshots_ to Sonatype.
2. __Staging__ refers to pushing potential release artifacts to Sonatype. Note that _staging_ an artifact does not automatically push it to Maven Central.
3. __Release__ refers to marking the artifacts for release on Sonatype's Nexus server so that they get pushed to Maven Central.

# Required Changes


## Artifacts

Pushing jars to Maven Central requires that you produce three artifacts:

1. Main jar with compiled classes
2. Sources jar
3. Javadocs jar

To automatically generate the second and third artifacts, add the following to the build plugins section of your `pom.xml`:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>2.1.2</version>
    <executions>
        <execution>
            <id>attach-sources</id>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>2.8.1</version>
    <executions>
        <execution>
            <id>attach-javadocs</id>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## Artifact Signing

Maven Central requires that you sign your released artifacts with GPG. Note that _snapshots_ do not need to be signed. Sonatype has a thorough article on signing your artifacts:

[How to Generate PGP Signatures With Maven](https://docs.sonatype.org/display/Repository/How+To+Generate+PGP+Signatures+With+Maven)

The key steps are:

1. Generate a key pair
2. Distribute your public key
3. Update your `pom.xml`

The first two steps are straightforward. The only trick with the third step is that you _only_ want to sign artifacts during the release process. If you add the `maven-gpg-plugin` to your main, every single build will get signed and Maven will prompt you for your passphrase with every build. Instead, you can define a Maven profile with a specific name and include the `maven-gpg-plugin` there.

```xml
<profiles>
  <profile>
    <id>release-sign-artifacts</id>
    <activation>
      <property>
        <name>performRelease</name>
        <value>true</value>
      </property>
    </activation>
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-gpg-plugin</artifactId>
          <version>1.1</version>
          <executions>
            <execution>
              <id>sign-artifacts</id>
              <phase>verify</phase>
              <goals>
                <goal>sign</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
      </plugins>
    </build>
  </profile>
</profiles>
```

## distributionManagement

Following the recommendation from Sonatype, you will remove the `<repositories></repositories>` section of your `pom.xml`. Once you do that, Maven no longer knows what server to use when _deploying_ or _staging_. To compensate, you need to add a `distributionManagement` section:

```xml
    <distributionManagement>
        <snapshotRepository>
            <id>sonatype-nexus-snapshots</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
        </snapshotRepository>
        <repository>
            <id>sonatype-nexus-staging</id>
            <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
        </repository>
    </distributionManagement>
```

## Sonatype Parent POM

You need to configure your `pom.xml` to inherit from the Sonatype Parent POM:

```xml
<parent>
    <groupId>org.sonatype.oss</groupId>
    <artifactId>oss-parent</artifactId>
    <version>7</version>
</parent>
```

# Deploying

Assuming you've followed the instructions from Sonatype and setup an account, you're now ready to deploy snapshots. The process is simple:

```bash
mvn clean deploy
```

That will automatically clean, build and push your snapshot to Sonatype.

# Releasing

Pushing a release jar to Maven Central consists of two parts: staging and releasing.

## Staging

In this part we clean everything up; prepare for the build by creating a tag for the release and updating the pom.xml; perform the build; deploy the artifacts to Sonatype.

The process consists of three commands:

```bash
mvn release:clean
mvn release:prepare
mvn release:perform
```

The `release:prepare` step will prompt you for release information. Here is some sample output from the 2.3.0 release of [metrics-statsd](https://github.com/organicveggie/metrics-statsd):

```bash
$ mvn release:prepare
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Metrics Statsd Support 2.3.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-release-plugin:2.2.1:prepare (default-cli) @ metrics-statsd ---
[INFO] Verifying that there are no local modifications...
[INFO]   ignoring changes on: pom.xml.next, release.properties, pom.xml.releaseBackup, pom.xml.backup, pom.xml.branch, pom.xml.tag
[INFO] Executing: /bin/sh -c cd /Users/sean/src/metrics-statsd && git status
[INFO] Working directory: /Users/sean/src/metrics-statsd
[INFO] Checking dependencies and plugins for snapshots ...
What is the release version for "Metrics Statsd Support"? (com.bealetech:metrics-statsd) 2.3.0: : 
What is SCM release tag or label for "Metrics Statsd Support"? (com.bealetech:metrics-statsd) v2.3.0: : 
What is the new development version for "Metrics Statsd Support"? (com.bealetech:metrics-statsd) 2.3.1-SNAPSHOT: : 
```

## Releasing

Once the artifacts are staged on Sonatype's Nexus, you need to go through a few annoying steps:

* Close out the staging release in Sonatype
* Release the artifacts in Sonatype

That process is well documented (with images) by Sonatype:

[https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-8a.ReleaseIt](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-8a.ReleaseIt)

Within two hours, Sonatype will push the artifacts to Maven Centrl.