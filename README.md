# Licensing Maven Plugin

## Getting Started

```shell
$ mvn org.neo4j.build.plugins:licensing-maven-plugin:check -DfailIfMissing=false
```

This will walk through your multimodule project and create
target/third-party-licensing.xml everywhere.

```shell
$ mvn org.neo4j.build.plugins:licensing-maven-plugin:collect-reports
```

This will walk through your multimodule project, finding all those
third-party-licensing.xml files and aggregate them together into a
target/aggregated-third-party-licensing.xml.

If you're attaching this to an execution, use:

```shell
$ mvn org.neo4j.build.plugins:licensing-maven-plugin:aggregate
```

This will only generate a `target/aggregated-third-party-licensing.xml` in your parent project (without failing your build).

## Examples

Here's an example licensing requirements XML file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<licensing-requirements>

        <!-- Many projects forget to include a <license/> block, so we need to explicitly list them here. -->
        <missing-licenses>
                <artifact id="org.springframework:web-mvc:3.0.6">
                        <license>Spring Source License</license>
                </artifact>
        </missing-licenses>

        <!-- Sometimes different people call the same license the same thing, we fix this up here. -->
        <!-- Note that *we* understand different versions of the same license are effectively different -->
        <!-- licenses and do not coalesce those together. You may not care though. -->
        <coalesced-licenses>
                <license name="GPLv2">
                        <aka>GNU Public License Version 2</aka>
                        <aka>The GNU Public License, Version 2.0</aka>
                </license>
        </coalesced-licenses>

        <!-- Maybe you don't like the GPL, maybe you don't like a particular countries open source license. Here's  -->
        <!-- where you declare which licenses you don't like. Other than AGPL/GPL, open source licenses don't really -->
        <!-- have an impact on your code, but sometimes people like to exclude the LGPL as the baby in the bathwater. -->
        <disliked-license>GPLv2</disliked-license>
        <disliked-license>LGPL</disliked-license>

	<!-- And with any set of strict rules, there are exceptions. Here we list artifacts that we wish to make -->
	<!--exempt from failing dislike checks. -->
        <dislike-exemption>xom:xom:jar:1.0</dislike-exemption>

</licensing-requirements>
```

Here's an example of how to bind the plugin into your build:

```xml
<plugins>
  <plugin>
    <groupId>org.neo4j.build.plugins</groupId>
    <artifactId>licensing-maven-plugin</artifactId>
    <version>1.5-SNAPSHOT</version>
    <executions>
      <execution>
        <id>enforce-licensing</id>
        <phase>compile</phase>
        <goals>
          <goal>check</goal>
        </goals>
        <configuration>
          <failIfDisliked>true</failIfDisliked>
	  <failIfMissing>true</failIfMissing>
          <licensingRequirementFiles>
	    <!-- You can specify more than one file here. -->
            <licensingRequirementFile>licensing-requirements.xml</licensingRequirementFile>
          </licensingRequirementFiles>
	  <!-- You can exclude your own stuff here -->
          <excludedGroups>org.example</excludedGroups>
        </configuration>
      </execution>
      <execution>
        <id>generate-licensing-xml</id>
        <phase>install</phase>
        <goals>
          <goal>aggregate</goal>
        </goals>
        <configuration>
	 <!-- aggregate goal does not support failIf* configuration like above. -->
          <licensingRequirementFiles>
            <licensingRequirementFile>licensing-requirements.xml</licensingRequirementFile>
          </licensingRequirementFiles>
          <excludedGroups>org.example</excludedGroups>
        </configuration>
      </execution>
    </executions>
    <dependencies>
     <!-- The licensing requirements XML should live in its own separate project to avoid chicken/egg problems. -->
      <dependency>
        <groupId>org.linuxstuff.example</groupId>
        <artifactId>license-requirements</artifactId>
        <version>1.0-SNAPSHOT</version>
      </dependency>
    </dependencies>
  </plugin>
</plugins>

```

## TODO
 - attach aggregated-third-party-licensing.xml to be deployed
 - format output XML 
 - make a proper maven site for the plugin
