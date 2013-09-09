---
layout: post
title: [NTS] Maven Remote Resources
---

The [Maven Remote Resources plugin](http://maven.apache.org/plugins/maven-remote-resources-plugin) allows you to package a given set of resources in a JAR in a way that they can be used by other modules. This is particularly useful if you have some "configuration" files that must be shared between different modules.

The plugin is also able to pre-process the resources using the [Apache Velocity](http://velocity.apache.org/) template language: every file that ends in `.vm` will be parsed. By opportunely using the `<properties>` section in your `pom.xml` you can configure what are the actual values that will be macro-expanded in the templates (e.g., if you define a `<id>foo</id>` property, every `$id` variable in your `vm` files will be replaced by `foo`)

Now in order for the plugin to work, you must use the `bundle` goal in the module defining your resources module, otherwise it won't be recognize as an artifact containing "remote resources".

In order to do this you must use the `bundle` goal in a build execution:

{% highlight xml %}
<build>
  <plugins>
    <!-- This is needed so the JAR can be processed as a remote resource bundle -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-remote-resources-plugin</artifactId>
      <version>1.5</version>
      <executions>
        <execution>
          <goals>
            <goal>bundle</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
{% endhighlight %}

This basically creates a file called `META-INF/maven/remote-resources.xml` in the JAR artifact and it's used to mark which files are to be considered as "remote resources" to be processed.

To include remote resources in a module, on the other hand, you must use the `process` goal:

{% highlight xml %}
<build>
  <plugins>
    <!-- This is needed so the JAR can be processed as a remote resource bundle -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-remote-resources-plugin</artifactId>
      <version>1.5</version>
      <executions>
        <execution>
          <goals>
            <goal>process</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
{% endhighlight %}

This will copy all the remote resources in your `target/maven-shared-archive-resources` (unless you've specified a different output directory in the plugin's configuration section)

These resources will then be available to the build as if they were resources defined in the module itself.


