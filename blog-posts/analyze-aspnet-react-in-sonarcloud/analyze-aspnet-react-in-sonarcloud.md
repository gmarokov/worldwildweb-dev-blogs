---
published: true
title: 'Analyze ASP.NET Core with React SPA in SonarCloud'
cover_image: ''
description: 'Setup SonarCloud analysis with ASP.NET Core and React SPA in GitLab CI'
tags: analysis, dotnet, react, sonarcloud
series: 'Static Code Analysis for .NET Core projects'
canonical_url: 'https://worldwildweb.dev/analyze-asp-net-core-with-react-spa-in-sonarcloud/'
---

[SonarCloud](https://sonarcloud.io/) is well known cloud based tool for Static Code Analysis which supports most of the popular programming languages - JavaScript, TypeScript, Python, C#, Java and counting. The tool is also known as [SonarQube](https://www.sonarqube.org/) which is the self hosted version of the analyzer. SonarCloud is completely free for public repositories and SonarQube is even open sourced. These characteristics make it my go-to tool for static code analysis for this project - setup SonarCloud for ASP.NET Core with React single page application.

This post is the second part from the series for Static Code Analysis for .NET Core projects. In the previous post we learned what Static Code Analysis is and introduced well known tools for the job. If you missed [that post you can check it out here](https://dev.to/gmarokov/static-code-analysis-for-your-net-projects-3l0d).

The agenda for today is:

- Overview of the different source control management platforms in SonarCloud
- Available options for analyzing your ASP.NET Core SPA app
- Build pipeline in GitLab

I will use React for the demo, but you can use whatever framework you need for the job. React/Angular/Vue or any other - it doesn't really matter, the flow stays the same, only the build or test running commands may differ.

Shall we begin? Let's deep dive!

# Different source control management platforms

SonarCloud works with the most popular SCM platforms - GitHub, GitLab, BitBucket and Azure DevOps. Different platforms but the declarative yml pipeline execution is what they all have in common.

Good to know is that SonarCloud provides 2 scanners - 1 for Dotnet projects and 1 for everything else. The good news is that the dedicated Dotnet scanner can also analyze files from your frontend app - JavaScript, TypeScript, CSS and HTML files.

Lets quickly go over the platforms and focus on GitLab with a full blown setup from scratch.

## GitHub

If you are using GitHub there is huge chance that you are already using GitHub Actions.

This is the easiest setup because SonarCloud generates pipeline setup for you. Of course you can use other CI tools as Circle CI, Travis CI or any other but you have to setup the dotnet-sonarscanner yourself. Check the **Build pipeline in GitLab** section as it has pretty relevant scenario.

## BitBucket

Before going into BitBucket beware that the platform (not yet?) supports apps targeting .NET Framework directly, but of course you can always use containers for the purpose.

SonarCloud doesn't provide any ready to go templates for .NET Core projects and BitBucket's pipeline. You still need to install and configure everything yourself.

## Azure DevOps

I read somewhere that dotnet-sonarscanner was developed with the partnership of Microsoft so no wonder the best integration with SonarCloud is with the famous Azure DevOps platform.

To enable SonarCloud in your pipelines first you need to install SonarCloud extension from Visual Studio marketplace and then follow the super descriptive guide which mostly involved clicking and can be easily accomplished with the GUI builder.

## GitLab

Nothing differs from the BitBucket setup. Later in the post comes full setup in GitLab.

## Local (Manually)

- Using the VSCode extension Sonar Dotnet gives you the ability to directly analyze from the editor. All the setup is through the GUI and reports are pushed to SonarCloud.
- Using the CLI - To use the CLI you must have .NET SDK, Java and the scanner installed and run the commands from the CI setup directly in the terminal. Check the requirements in the [official docs](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/).

# Available options for analysis

On the road to analyze the combined single page application, we have two paths we can choose to take.

### Option 1: Analyze frontend and backend at once

The dedicated scanner for .NET projects possess the power to also scan JS, TS, HTML, CSS etc. files. We only need to include frontend's files with wildcard in the `.csproj` as follows:

```
<ItemGroup>
	  <!-- Don't publish the SPA source files, but do show them in the project files list -->
    <Content Remove="Frontend\**" />
    <None Remove="Frontend\**" />
    <None Include="Frontend\**" Exclude="Frontend\node_modules\**" />
</ItemGroup>
```

Or if you are using .NET Core 3.1 and above, the default template includes the frontend in your ASP.NET Core project in a common way.

### Option 2: Analyze frontend and backend separately

This option is useful when you have a monorepo with your backend and frontend in it, but they have a separate startup process or even different teams working on them. This option will require to create a 2 separate projects in SonarCloud. The option will also require to use the default SonarCloud analyzer for your frontend.

# Build pipeline in GitLab

Let's recap everything we discussed so far and put it to work. To cover most of the cases for setuping SonarCloud analysis, I will try to walk you through the whole setup with a example project from the ASP.NET Core with React SPA sample with a separate scan tasks for frontend and backend.

Before we start lets create empty `.gitlab-ci.yml` file in the root directory.

For GitLab CI file reference checkout official docs: [https://docs.gitlab.com/ee/ci/yaml/gitlab_ci_yaml.html](https://docs.gitlab.com/ee/ci/yaml/gitlab_ci_yaml.html)

## Frontend

Starting with the creation of our frontend Sonar project which needs to be done manually. Just throw some descriptive name and a project key and you are ready to go. Once done, Sonar will provide **SONAR_TOKEN** and **SONAR_HOST_URL**. Make sure to add them as Environment variables.

Next step is to define the variables for the CI job:

```csharp
variables:
  SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
  GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
```

After that comes the stage definitions of the job. In this case we will have two - one for frontend and one for the backend:

```csharp
stages:
  - frontend
  - backend
```

Create the frontend's actual stage definition with the following task. You can have as many task for a stage as you like but we will stick to just one:

```csharp
frontend.build.test.analyze:
  stage: frontend
  image:
    name: sonarsource/sonar-scanner-cli:latest
    entrypoint: [""]
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script:
    - cd Frontend
    - npm install
    - npm run build
    - npm test
    - sonar-scanner
        -Dsonar.projectKey=sonar.example.frontend
        -Dsonar.organization=gmarokov-1
        -Dsonar.sources=src
        -Dsonar.exclusions="/node_modules/**,/build/**,**/__tests__/**"
        -Dsonar.tests=src
        -Dsonar.test.inclusions=**/__tests__/**
        -Dsonar.javascript.lcov.reportPaths="coverage/lcov.info"
        -Dsonar.testExecutionReportPaths="reports/test-report.xml"
  only:
    - merge_requests
    - master
    - tags
```

A lot is happening in this task so lets walkthrough:

`frontend.build.test.analyze`

The name of the job, its up to you to give it a descriptive name

`stage: frontend`

The name of the stage which this task belongs to. Must be predefined which we did above.

```csharp
image: # We can use existing docker images
    name: sonarsource/sonar-scanner-cli:latest
    entrypoint: [""]
```

Here we specify a Docker image which comes with sonar-scanner-cli preinstalled. This Scanner CLI is used for all languages except for Dotnet as I mentioned above.

```csharp
cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
```

We specify the cache and not download the image every time we run the job. This should be good.

```csharp
script:
    - cd Frontend
    - npm install
    - npm run build
    - npm test
```

Nothing fancy here, regular npm stuff but note that tests are run with coverage report and special `jest-sonar-reporter` in the `package.json` which converts test result data to Generic Test Data which is one of the supported formats by SonarCloud.

```csharp
 - sonar-scanner
    -Dsonar.projectKey=sonar.example.frontend
    -Dsonar.organization=gmarokov-1
    -Dsonar.sources=src
    -Dsonar.exclusions="/node_modules/**,/build/**,**/__tests__/**"
    -Dsonar.tests=src
    -Dsonar.test.inclusions=**/__tests__/**
    -Dsonar.javascript.lcov.reportPaths="coverage/lcov.info"
    -Dsonar.testExecutionReportPaths="reports/test-report.xml"
```

Here comes the actual scan. Required parameters are **projectKey**, **organization** and the early added **SONAR_TOKEN** and **SONAR_HOST_URL** which are taken from the env variables.

Then comes the configuration of the source directories, directories to exclude, test directories and the paths to the generated reports for coverage and test execution.

More about the parameters can be found here: [https://docs.sonarqube.org/latest/analysis/analysis-parameters/](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)

And our frontend is good to go. Coming next is the backend.

## Backend

For the backend another project needs to be created manually. Since we already have environment variable with the name of **SONAR_TOKEN**, you can save the token for this project as **SONAR_TOKEN_BACKEND** for example. We will manually provide it anyway.

When it comes to the backend scan, it will be a little different since we will use the dedicated scanner for Dotnet.

```csharp
backend.build.test.analyze:
  stage: backend
  image: gmarokov/sonar.dotnet:5.0
  script:
   - dotnet sonarscanner begin
        /k:"sonar.example.backend" /o:"gmarokov-1"
        /d:sonar.login="$SONAR_TOKEN_BACKEND"
        /d:sonar.host.url="$SONAR_HOST_URL"
        /d:sonar.exclusions="**/Migrations/**, /Frontend"
        /d:sonar.cs.opencover.reportsPaths="**/coverage.opencover.xml"
        /d:sonar.sources="/Backend/Backend.Api"
        /d:sonar.tests="/Backend/Backend.Api.Tests"
        /d:sonar.testExecutionReportPaths="SonarTestResults.xml"
   - dotnet build Backend/Backend.sln
   - dotnet test Backend/Backend.sln --logger trx /p:CollectCoverage=true /p:CoverletOutputFormat=opencover /p:ExcludeByFile="**/Migrations/*.cs%2CTemplates/**/*.cshtml%2Ccwwwroot/%2C**/*.resx"
   - dotnet-trx2sonar -d ./ -o ./Backend/SonarTestResults.xml
   - dotnet sonarscanner end /d:sonar.login="$SONAR_TOKEN_BACKEND"
  only:
    - branches
    - master
    - tags
```

Let's walkthrough the whole task:

`image: gmarokov/sonar.dotnet:5.0`

Again Docker image which will be used to spin a container on which we will execute our task. This image have Dotnet SDK, Java runtime, SonarDotnet and Dotnet-Trx2Sonar global tools. The image can be found on [DockerHub](https://hub.docker.com/r/gmarokov/sonar.dotnet) which looks like this:

```csharp
*# Image with Dotnet SDK, Java runtime,* SonarDotnet, Dotnet-Trx2Sonar *dotnet tools*
FROM mcr.microsoft.com/dotnet/sdk:5.0-focal
ENV PATH="$PATH:/root/.dotnet/tools"

*# Install Java Runtime*
RUN apt-get update
RUN apt install default-jre -y

*# Install SonarCloud dotnet tool*
RUN dotnet tool install --global dotnet-sonarscanner

# Install Trx2Sonar converter dotnet tool
RUN dotnet tool install --global dotnet-trx2sonar
```

You might spot the following suspicious parameter:

`/p:ExcludeByFile="**/Migrations/*.cs%2CTemplates/**/*.cshtml%2Ccwwwroot/%2C**/*.resx"`

That's because of the the underling PowerShell parser fails to parse the comma as separator so we need to use encoded value.

`dotnet-trx2sonar -d ./ -o ./Backend/SonarTestResults.xml`

The dotnet-trx2sonar tool will help us to convert .trx files (Visual Studio Test Results File) generated by Xunit to Generic Test Data which is the format specified by SonarCloud. The converted file will help us to browse the tests in SonarCloud UI.

Anddd that's it! Pipeline is ready to go and provide analyzes on every CI run. I also added some nice badges to indicate the SonarCloud analysis status directly in the repo.

[The full demo project can found on GitLab here](https://gitlab.com/gmarokov/sonar-aspnet-react-example).

# Conclusion

Benefits of these type of analyses are enormous and setup can be dead simple. Yes, delivery is important, but static code analysis compliments it perfectly making delivery more predictable, secure and stable by catching common pitfalls and violations as early as the developer writes code or commits.

If you haven't used any static code analysis tools before, now you don't have any excuse not to!

# Resources

[https://codeburst.io/code-coverage-in-net-core-projects-c3d6536fd7d7](https://codeburst.io/code-coverage-in-net-core-projects-c3d6536fd7d7)

[https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871](https://community.sonarsource.com/t/coverage-test-data-generate-reports-for-c-vb-net/9871)

[https://dotnetthoughts.net/static-code-analysis-of-netcore-projects/](https://dotnetthoughts.net/static-code-analysis-of-netcore-projects/)

[https://sonarcloud.io/documentation/analysis/scan/sonarscanner-for-msbuild/](https://sonarcloud.io/documentation/analysis/scan/sonarscanner-for-msbuild/)

[https://sonarcloud.io/documentation/analysis/scan/sonarscanner/](https://sonarcloud.io/documentation/analysis/scan/sonarscanner/)
