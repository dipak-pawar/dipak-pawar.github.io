---
layout: post
author: Dipak Pawar
title: Integration of Arquillian-Governor with Arquillian-Recorder
date: 2016-09-26 13:48:47 
tags:
- Arquillian
thumbnail_path: blog/arquillian-logo.png
---
**Arquillian Govenor**

You can use it to programmatically choose what test methods of your Arquillian tests you want to execute and what you want to be skipped by putting your custom annotations on the test methods.

**Arquillian Recorder**

It gives neat reports for your arquillian tests in different format i.e. JSON, HTML, XML, Asciidoctor.

Now How to get properties like Issue Id, Force Value & Environment detectors used in Arquillian governor to programmatically choose execution of your method?

In arquillian-governor release `1.0.3.Final` this feature is not available. We have added this feature to support for Jira, GitHub, Redmine Governor in this release. You can get such properties by only import `arquillian-governor` & 'arquillian-recorder' dependencies. Following dependencies are for Arquillian Jira Governor.

{% highlight xml %}
<dependency>
   <groupId>org.arquillian.extension</groupId>
   <artifactId>arquillian-recorder-reporter-impl</artifactId>
   <version>${version.arquillian.recorder}</version>
   <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.arquillian.extension</groupId>
    <artifactId>arquillian-governor-jira</artifactId>
    <version>${version.jira.governor}</version>
    <scope>test</scope>
 </dependency>
{% endhighlight %}

*Note*: Make sure to use version.jira.governor > `1.0.3.Final`.

### Let's check it in action for Arquillian Jira Governor:

Create TestClass in which you will add method annotated with Jira issue which you would like to test.

{% highlight java %}
@RunWith(Arquillian.class)
@RunAsClient
public class JiraGovernorTest {

    @Test
    @Jira(value = "ARQ-2005")
    public void wontBeSkippedBecauseResolved() {
        Assert.assertTrue(true);
    }

    @Test
    @Jira(value = "ARQ-831", force = true)
    public void willBeSkippedBecauseUnresolved() {
        Assert.assertTrue(false);
    }

    // This test case will be executed on all operation systems except for Windows.If the current OS is Windows then the status of JIRA issue will be checked.
    
    @Test
    @Jira(value = "ARQ-1000", detector = @Detector(OS.Windows.class))
    public void environmentDetectorTest(){
        Assert.assertTrue(true);
    }
}
{% endhighlight %}

To run above test you should have configure arquillian.xml in which you have to add `governor-jira` & `governor-recorder` extension with appropriate property which will look like following:

*Arquillian.xml*

{% highlight xml %}
<?xml version="1.0"?>
<arquillian xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://jboss.org/schema/arquillian"
            xsi:schemaLocation="http://jboss.org/schema/arquillian
    http://jboss.org/schema/arquillian/arquillian_1_0.xsd">

    <extension qualifier="governor-jira">
        <property name="closePassed">true</property>
        <property name="force">false</property>

        <!-- your jira username -->
        <property name="username"></property>

        <!-- your jira password -->
        <property name="password"></property>

        <!-- your jira instance, e.g https://issues.jenkins-ci.org -->
        <property name="server">https://issues.jboss.org</property>
    </extension>
    <extension qualifier="reporter">
    </extension>
</arquillian>
{% endhighlight %}

To run above test you should have `junit`, `governor-jira`, `arquillian-junit-standalone`, `arquillian-recorder` dependencies in your pom.xml.

*pom.xml*
{% highlight xml %}

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.integration</groupId>
    <artifactId>governor-recorder-integration</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.arquillian.extension</groupId>
            <artifactId>arquillian-governor-jira</artifactId>
            <version>1.0.4.Final</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.arquillian.extension</groupId>
            <artifactId>arquillian-recorder-reporter-impl</artifactId>
            <version>1.1.2.Final</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.arquillian.junit</groupId>
            <artifactId>arquillian-junit-standalone</artifactId>
            <version>1.1.11.Final</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
{% endhighlight %}

If you run above project using `mvn test`, you will get report for your arquillian-test in [target/arquillian-report.xml](https://gist.githubusercontent.com/dipak-pawar/803570639bb22298364e691d2243daf5/raw/34abdbb26e265fbf44b54b2b23e8bc1e23c950d8/arquillian-report.xml) file.

We have configured Arquillian to test it like test library. You can Configure it to run it with Remote/Managed/Embedded container with deployment method.

Same way you can configure your GitHub Govenor & Redmine Governor to get all configuration information of respective governor in your report.

Don't forget to add appropriate dependency along with recorder dependency.

**GitHub Governor & Recorder Dependecy**:

{% highlight xml %}
<dependency>
    <groupId>org.arquillian.extension</groupId>
    <artifactId>arquillian-governor-github</artifactId>
    <version>${version.github.governor}</version>
    <scope>test</scope>
</dependency>
<dependency>
   <groupId>org.arquillian.extension</groupId>
   <artifactId>arquillian-recorder-reporter-impl</artifactId>
   <version>${version.arquillian.recorder}</version>
   <scope>test</scope>
</dependency>
{% endhighlight %}

**Redmine Governor & Recorder Dependency:**

{% highlight xml %}
<dependency>
    <groupId>org.arquillian.extension</groupId>
    <artifactId>arquillian-governor-redmine</artifactId>
    <version>${version.redmine.governor}</version>
    <scope>test</scope>
 </dependency>
<dependency>
   <groupId>org.arquillian.extension</groupId>
   <artifactId>arquillian-recorder-reporter-impl</artifactId>
   <version>${version.arquillian.recorder}</version>
   <scope>test</scope>
</dependency>
{% endhighlight %}

Visit [Arquillian Governor](https://github.com/arquillian/arquillian-governor) & [Arquillain Recorder](https://github.com/arquillian/arquillian-recorder) to see How you can configure arquillian.xml depending on your requirement?

