---
layout: post
title: "SeleniumIDE execution and report exhibition in Jenkins"
categories: infrastructure
tags: [selenium,continuous-integration,jenkins]
image:
  #feature: mountains.jpg
  #teaser: mountains-teaser.jpg
  #credit: Death to Stock Photo
  #creditlink: ""
---
Exploring Selenium IDE and Jenkins.

- Table of Contents
{:toc .large-only}

Jenkins is the most used tool to provide continuous integration and automation of tasks in this regard.
In this article, I'll explain how can we use Jenkins to run SeleniumIDE test suites and then present the results in reports.

## A little about Selenium IDE
Selenium IDE is a Firefox plugin that allows the application tests to record functional, black-box tests. As output, it generates
a HTML file that can be used as a script for further executions. Is suggested that both SeleniumIDE (2.9.1.1-signed) and Selenium Expert (Selenium IDE) (0.25.1-signed.1-signed) Firefox plugins are installed for proper Selenium execution.

After that, it's all about recording actions in Firefox and then save them as scripts for further usage.

### Tests cases and suites

These concepts are important. The test suite is a grouping of tests cases. Each test case generates as output a single HTML file, and
the test suite is another HTML file, whose purpose is to group all the HTML test case files.

It's important to know this because Jenkins plugins consider only test suites, instead of test cases. So, your test cases
must be grouped in suites before being automated in Jenkins.

## Jenkins installation

Download and install Jenkins latest version.
After that, the following plugins must also been installed:

- Selenium HTML report
- Hudson Seleniumhq plugin
- Environment Injector Plugin

You can install these plugins directly by the Jenkins interface (as admin, navigate through `/Manage Jenkins/Manage plugins`). Some application restart may be required.

### Installation of supporting tools

Two more tools are necessary:

- `selenium-html-runner-3.0.1.jar:` this library has the responsibility of execute the selenium script, providing results. It will be executed by Jenkins Job. It can be downloaded from the [SeleniumHQ website](http://www.seleniumhq.org/download/), and also using the [direct link](https://goo.gl/Br1P0Z). At the moment of this writing, the latest release version, which I'm adopting, is 3.0.1;
- `geckodriver`: the required bridge from W3C WebDriver to Gecko browser technology. It will also be used by the Jenkins Job. It can be downloaded from git [Github repository releases'page](https://github.com/mozilla/geckodriver/releases). At the moment of this writing, the latest release version, which I'm adopting, is v.0.13.0.

Download and save these artifacts (they'll be referenced by Jenkins Jobs, as described in the next sections). To execute the examples in this article, they must be saved in `/Users/dma/apps/selenium-server`.

## Creating and configuring the job

Since the installation step is done, let's proceed to the configuration.

Jenkins is composed of several `Jobs`. These jobs can be configured to execute a very broad range of actions. Each job execution is called a `build`, and each build has it's own history of events and procedures. In this scenario, let's create a single Job and configure it to process SeleniumIDE script test suite, providing the result of tests execution using some sort of reports.

### Configuring the Job

- Create the new Job. Let's call it `PoC-selenium`.
- In the `Configure` options/`Build` section:
  - Add a new `Inject Environment Variables` section
  - Enter the following variables (remember to change the content of these variables according to your own environment):

```shell
SITE_DOMAIN="http://localhost:8081"
SUITE_TEST_DIR=/Users/dma/data/
SUITE_TEST_NAME=selenium-test-jmeter-stub-application-suite

SUITE="*firefox"
REPORT_DIR=selenium-reports
SEL_PATH=/Users/dma/apps/selenium-server/

SUITE_TEST_PATH=$SUITE_TEST_DIR/$SUITE_TEST_NAME.htm
TMP_REPORT_PATH=$SUITE_TEST_DIR/$REPORT_DIR
REPORT_PATH=$WORKSPACE/$REPORT_DIR/$SUITE_TEST_NAME-results.html
```

- Still in the `Configure` options/`Build` section:
  - Add a new `Execute shell` section
  - Enter the following script ([^1]):

```shell
java -Dwebdriver.gecko.driver=$SEL_PATH/geckodriver -jar $SEL_PATH/selenium-html-runner-3.0.1.jar -htmlSuite $SUITE $SITE_DOMAIN $SUITE_TEST_PATH $TMP_REPORT_PATH > /dev/null 2>&1 || true
mv $TMP_REPORT_PATH/firefox.results $REPORT_PATH
```


- Still in the `Configure` options/`Post-Build` section:
  - Create a `Publish Selenium HTML Report`
    - set the value to `selenium-reports`
  - Create a `Publish Selenium Report`
    - set the value to `selenium-reports/*.html`

## Results

After Job execution, you can check the report in a Job-basis (enter into the Job details to see information related to the latest execution results) or in a build-basis (enter into the build details and you'll see an option to show details about Selenium execution in that build).

## Final considerations

[^1]: There is a [bug](https://github.com/mozilla/geckodriver/issues/210) in Selenium HTML Runner which does not return the execution properly, causing the process to fail, even if the report is properly generated. So, in the shell script, the output is send to infinity and the execution is assumed as successfully, so the report can be properly generated.
