---
layout: post
title: "Exporting sonar data with SonarLint"
categories: infrastructure
tags: [sonarqube,sonarlint,static-code-analysis]
#image:
  #feature: mountains.jpg
  #teaser: mountains-teaser.jpg
  #credit: Death to Stock Photo
  #creditlink: ""
---

SonarQube is currently the most popular static code analisys tool. There are a lot of features there, but one is not found: the possibility to export metrics.
In this post I'll demonstrate a way to do this.

## The problem

Recently, I was facing the scenario where I have to export metrics from SonarQube
of one client for further analysis. However, SonarQube doesn't have
this feature. You can export rules, but not the data generated after analysis.

## Using SolarLint CLI to export data

I discovered the SonarLint (some tool commonly used for IDEs such as Eclipse
and IntelliJ to connect to and execute rules from SonarQube environments) has also a command
line tool. So, my first thought was to use the SonarLint CLI (command-line
interface) to export the previously generated metrics. However, when executed in
the parent project, the generated HTML file was too big to be properly opened
in the web browser. So, it was needed to execute the SonarLint CLI to each subproject
from the parent project.

Since the client used Windows, one batch script was created in order to export
the data within those constraints. The batch is presented below:

```batch
@echo off

set drive-tool=d:
set drive-report=c:
set sonarlint-path=%drive-tool%\tools\sonarlint-cli-2.1.0.566\bin
set sonarlint-report-path=%drive-report%\Users\SomeUser\Desktop\sonarlint-data
set app-path=%drive-tool%\apps\target-app

set PATH=%PATH%;%sonarlint-path%

for /D %%f in (%app-path%\*) do ( cd %drive-report% & mkdir
"%sonarlint-report-path%\target-app\%%~nxf\" & cd "%%f" & sonarlint.bat
--html-report "%sonarlint-report-path%\target-app\%%~nxf\%%~nxf.html"
--charset ISO-8859-1 --src "**/*.java" )
```
