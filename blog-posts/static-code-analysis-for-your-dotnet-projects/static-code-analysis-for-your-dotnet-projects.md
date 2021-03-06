---
published: true
title: 'Static Code Analysis for your .NET projects'
cover_image: ''
description: 'Collection of static code analysis tools for dotnet projects'
tags: analysis, dotnet, tools, sonarcloud
series: 'Static Code Analysis for .NET Core projects'
canonical_url: 'https://worldwildweb.dev/static-code-analysis-for-your-net-projects'
---

# What is Static Code Analysis

Every developer wants to write predictive, maintainable and high quality software. Unfortunately that's not always the case because of our human nature - we do make mistakes. That's why we try to automate all the things related to software development lifecycle: testing, deploying, running applications.

But what about the codebase? What do we do to enforce minimally complex and maintainable code, ensure proper code styles standards, prevent common pitfalls and violations, and predict what the code would do at runtime?

By applying RULES defined by your team, the platform or the programming language. And that's what Static Code Analysis is all about.

Static Code Analysis can be simple manual inspection such as code review or automated via some of the tools we will overview in this blog post.

**Keep digging**

_Eric Dietrich wrote very explanatory article about what exactly Static Analysis is here:_

[https://blog.ndepend.com/static-analysis-explanation-everyone/](https://blog.ndepend.com/static-analysis-explanation-everyone/)

_If you are curious about Dynamic Analysis you can also check out these articles:_

[https://securityboulevard.com/2021/02/dynamic-code-analysis-a-primer/](https://securityboulevard.com/2021/02/dynamic-code-analysis-a-primer/)

[https://github.com/analysis-tools-dev/dynamic-analysis](https://github.com/analysis-tools-dev/dynamic-analysis)

[https://www.overops.com/blog/static-vs-dynamic-code-analysis-how-to-choose-between-them/](https://www.overops.com/blog/static-vs-dynamic-code-analysis-how-to-choose-between-them/)

This post is Part 1 from the Static Analysis series. In the next post we will setup SonarCloud for a ASP.NET Core + React SPA project in CI pipeline. Check it out [here](https://dev.to/gmarokov/analyze-asp-net-core-with-your-react-spa-in-sonarcloud-5goj).

# Where to use Static Code Analysis

I found plenty of NuGet packages, IDE extensions and external services available on the market. That was hard to digest and probably I might miss some very helpful tools. Would be great if you guys share your opinion or favorite tools for the job.

## In development

Using build-time code analysis in Visual Studio /Code (or other preferred tool), we enable developers to quickly understand what rules are being broken. This enables them to fix code earlier in the development lifecycle, and we can avoid builds that fail later.

### Extensions for Visual Studio Code

- OmniSharp - [https://github.com/OmniSharp/omnisharp-roslyn](https://github.com/OmniSharp/omnisharp-roslyn)
  A go-to tool for C# development in VSC.
- Roslynator - [https://github.com/JosefPihrt/Roslynator](https://github.com/JosefPihrt/Roslynator)
  A collection of 500+ analyzers, refactorings and fixes for C#, powered by Roslyn.
- DevSkim - [https://github.com/microsoft/devskim](https://github.com/microsoft/devskim)
  DevSkim is a framework of IDE extensions and language analyzers that provide inline security analysis in the dev environment as the developer writes code.
- SonarLint - [https://www.sonarlint.org/vscode](https://www.sonarlint.org/vscode)
  Even that this extension doesn't scan your .NET projects it's still super useful for your frontend html, css, js, ts files.
- Sonar Dotnet - [https://github.com/yagoluiz/sonar-dotnet-vscode/](https://github.com/yagoluiz/sonar-dotnet-vscode/)
  Easy connect to SonarCloud from your development environment.

### Extensions for Visual Studio

- Roslynator - [https://github.com/JosefPihrt/Roslynator](https://github.com/JosefPihrt/Roslynator)
  Again the famous Roslynator analyzers for Visual Studio.
- ReSharper - [https://www.jetbrains.com/resharper/](https://www.jetbrains.com/resharper/)
  This is more than just analysis tool. If you haven't heard of ReSharper, definitely should be checked out.
- NDepend - [https://www.ndepend.com/](https://www.ndepend.com/)
  The "Swiss Army Knife" for .NET Developers, Architects and Teams.

### Other tools

- Rider - [https://www.jetbrains.com/rider/](https://www.jetbrains.com/rider/)
  Another great IDE for .NET developers which comes with the power of ReSharper.

## In build pipelines

NuGet packaged analyzers are the easiest, and they will automatically run as your project builds on the build agents. When a build encounters a code quality error, you can immediately fail the build, send alerts, or apply any other actions you and your team needs.

.NET Core SDK 3.0 or later, comes with included analyzers for Open APIs previously known as Swagger. To enable the analyzer in your project, include the IncludeOpenAPIAnalyzers property in the project file:

```csharp
<PropertyGroup>
	<IncludeOpenAPIAnalyzers>true</IncludeOpenAPIAnalyzers>
</PropertyGroup>
```

### NuGet packages

- Microsoft recommended code quality rules and .NET API usage rules - [https://www.nuget.org/packages/Microsoft.CodeAnalysis.NetAnalyzers/](https://www.nuget.org/packages/Microsoft.CodeAnalysis.NetAnalyzers/)
- Microsoft's CSharp analyzers for ASP.NET Core MVC - [https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Api.Analyzers](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Api.Analyzers)
- Roslynator analyzers as NuGet package - [https://www.nuget.org/packages/Roslynator.Analyzers](https://www.nuget.org/packages/Roslynator.Analyzers)
- SonarCloud analyzers - [https://www.nuget.org/packages/SonarAnalyzer.CSharp/](https://www.nuget.org/packages/SonarAnalyzer.CSharp/)
- Check for Async/await misuses - [https://www.nuget.org/packages/AsyncFixer](https://www.nuget.org/packages/AsyncFixer)
- An implementation of StyleCop's rules using Roslyn - [https://www.nuget.org/packages/StyleCop.Analyzers/](https://www.nuget.org/packages/StyleCop.Analyzers/)

**Security analyzers**

- Security static code analyzer for .NET - [https://www.nuget.org/packages/SecurityCodeScan.VS2019/](https://www.nuget.org/packages/SecurityCodeScan.VS2019/)
- DevSkim is a framework and language analyzer that provides inline security analysis - [https://www.nuget.org/packages/Microsoft.CST.DevSkim/](https://www.nuget.org/packages/Microsoft.CST.DevSkim/)

Different CI tools may provide their own tool for security analysis:

- Only Azure DevOps - [https://secdevtools.azurewebsites.net/helpcredscan.html](https://secdevtools.azurewebsites.net/helpcredscan.html)
- Only GitLab - [https://docs.gitlab.com/ee/user/application_security/sast/index.html](https://docs.gitlab.com/ee/user/application_security/sast/index.html)
- GitHub - https://github.com/marketplace?category=code-quality&type=apps
  Last but not least. GitHub's community driven marketplace provides so many tools for code quality, security and everything else you can think of.

**NuGet packages for the Test projects**

- Provides diagnostic analyzers to warn about incorrect usage of NSubstitute in C# - [https://www.nuget.org/packages/NSubstitute.Analyzers.CSharp](https://www.nuget.org/packages/NSubstitute.Analyzers.CSharp)
- Code Analyzers for projects using xUnit.net - [https://www.nuget.org/packages/xunit.analyzers](https://www.nuget.org/packages/xunit.analyzers)

### External services

- SonarCloud - [https://sonarcloud.io/](https://sonarcloud.io/)
  My go-to tool for .NET projects. They even have a separate scanner for .NET. Pretty nice integration with Azure DevOps. Free for public projects.
- Embold - [https://embold.io/](https://embold.io/)
  Fairly new tool with Free plan for 1M executable-lines-of-code for public repositories.
- CodeBeat - [https://codebeat.co/](https://codebeat.co/projects/gitlab-com-autohub-autohub-web-master)
  Free for public repositories.
- CodeFactor - [https://www.codefactor.io/](https://www.codefactor.io/dashboard)
  1 private and unlimited free repositories.
- CodeClimate - [https://codeclimate.com/](https://codeclimate.com/github/gmarokov/dotnet-trx2sonar)
  50 free repositories.
- Codacy - [https://www.codacy.com/](https://www.codacy.com/pricing)
  Paid service.

And many more counting. These are the one I found easy to get started without installing and configuring additional software.

# Conclusion

In first issues raised by static code analysis might be considered as overhead, but static code analysis brings huge benefits in long term which can be summarized to but not only:

- You have the confidence to release more frequently.
- This results in having a quicker TTM (Time to Market).
- Reduce business risks (data loss, vulnerabilities, application failures, ..)

Rules may sometimes get on your way and slow down your development, but you and your team are in charge to establish given rules or completely ignore/disable them.

In the next post I will configure SonarCloud for ASP.NET Core + React SPA so stay tuned.

Which are your favorite static code analysis tools? Please share your thoughts in the comments or [create a PR in GitHub](https://github.com/gmarokov/worldwildweb-dev-blogs/blob/master/blog-posts/static-code-analysis-for-your-dotnet-projects/static-code-analysis-for-your-dotnet-projects.md).

Happy analyzing :)

## Resources

[https://blog.tdwright.co.uk/2018/12/10/seven-reasons-that-roslyn-based-code-analysers-are-awesome/?preview=true](https://blog.tdwright.co.uk/2018/12/10/seven-reasons-that-roslyn-based-code-analysers-are-awesome/?preview=true)

[https://docs.microsoft.com/en-us/visualstudio/code-quality/?view=vs-2019](https://docs.microsoft.com/en-us/visualstudio/code-quality/?view=vs-2019)

[https://github.com/analysis-tools-dev/static-analysis](https://github.com/analysis-tools-dev/static-analysis)
