---
title: "Releasing OpenAPI Specs at Compile Time with .NET 9"
description: >
  Explore different OpenAPI generation strategies and learn how to use the new .NET 9 compile-time documentation feature.
summary: >
  In this article, we explore a new feature introduced in .NET 9 in a DevOps perspective:
  generating OpenAPI documentation at compile time, as part of the application’s promotion flow and more.
date: 2025-06-02
tags: [azure, open-api, ApiOps, ADO , Azure Devops, YAML, Pipeline, net9 ]
categories: ["Pipeline", "DevOps", "ApiOps", net9]
series: ["Pipeline, Devops", "ApiOps", net9]
author: ["Pieroci"]
ShowToc: false
TocOpen: false
draft: true
weight: 5
cover:
    image: "../img/8rpnb0ns.png" # image path/url
    alt: "Releasing OpenAPI Specs at Compile Time with .NET 9" 
    caption: "Releasing OpenAPI Specs at Compile Time with .NET 9"
    relative: true # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/pieroci/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

---

## 🧠 Why Generate Build-Time OpenAPI Document Matters

With .NET 9, Microsoft enables developers to:

- Generate OpenAPI specs **at compile time**, avoiding runtime dependency on Swashbuckle.
- Improve CI/CD integration by producing OpenAPI files as deterministic build artifacts.
- Ensure **secure and minimal** deployment environments.

You can view the full Microsoft documentation here: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-9.0&tabs=visual-studio%2Cvisual-studio-code

Before the tutorial, let's do a very quick summary of the context.

*PS.* 

*In the article I will not say the word "Swagger" but I will express it with the concept "document generation at RunTime ".* 

*In the article I will define CodeFirst only the RunTime approach to identify the gap with the CompileTime approach.*

*Don't hate me for this! :)*

One question I've been asking myself is why such an important feature is only being introduced now... 

## 🔍 "Open-api spec" and the evolution through the years



What is Open Api ? From wikipedia:

> The **OpenAPI Specification**, previously known as the **Swagger Specification**, is a [specification](https://en.wikipedia.org/wiki/Specification_(technical_standard)) for a [machine-readable](https://en.wikipedia.org/wiki/Machine-readable_medium) [interface definition language](https://en.wikipedia.org/wiki/Interface_definition_language) for describing, producing, consuming and visualizing [web services](https://en.wikipedia.org/wiki/Web_API).[[1\]](https://en.wikipedia.org/wiki/OpenAPI_Specification#cite_note-started-1) Originally developed to support the [Swagger](https://en.wikipedia.org/wiki/Swagger_(software)) framework, it became a separate project in 2015, overseen by the OpenAPI Initiative, an open-source collaboration project of the [Linux Foundation](https://en.wikipedia.org/wiki/Linux_Foundation).[[2\]](https://en.wikipedia.org/wiki/OpenAPI_Specification#cite_note-2)[[3\]](https://en.wikipedia.org/wiki/OpenAPI_Specification#cite_note-charter-3)

From the graph we can see how the specification is improved over the years. 

![openapi](.\img\openapi_spec_evolution.png)



## 🔍 "Open-api spec *generation* approach" evolution through the years in MS ecosystem

The OpenApi document generation approach of our API interface contracts has evolved over the years in the **microsoft ecosystem.**

Approaches to the Open Api Documentation generation:

- OpenApi documentation at Run Time (Code First) 
- ApiOps 
- Open Api documentation at CompileTime

Recently, the ApiOps approach (a GitOps methodology for APIs – see: https://learn.microsoft.com/en-gb/azure/architecture/example-scenario/devops/automated-api-deployments-apiops) has gained popularity.

Only from NET 9 MS introduce a way to obtain the documentation also at compile time.

![openapi](.\img\openapi_spec_release_evolution.png)

Here is the timeline chart showing the evolution of OpenAPI release approaches in the **Microsoft ecosystem.** Each line spans the years during which the approach has been in use and continues through 2025, indicating they are still relevant today.

Nuget trends show a deep use of RunTime Generation in the last six months.(you can see it here - https://www.nuget.org/stats/packages/Swashbuckle.AspNetCore.Swagger?groupby=Version)

 ![openapi](.\img\swagger-stats.png)

Statistics must necessarily be interpreted: numerically it is not a very important data if we consider that part of the downloads are coming from automation pipelines and the different net version...).

This number becomes important if we compare it with the OpenApi library used for the new Compile-Time feature  :

 The stats show the situation in the last six month.![openapi](.\img\openapi-stats.png)

**What does this number tell me?**

*Even if the introduction of the new "OpenApi Generated at Compile-Time" feature comes late (even if there are new methodologies like ApiOps) the IT world does not keep up with the evolution of technologies and continues to be poorly updated and does not take advantage of the improvements until years later.*

***Maybe this feature isn't that late after all!***
I've finished my polemic! :D
:arrow_down: Let's move on to the comparison of approaches...

## 🔍 Comparing OpenAPI Doc Generation Approaches

### 🧭 **ApiOps (Contract-First Approach)**

**✅ Pros:**

- **Separation of concerns:** API contracts are designed independently from code, encouraging better architecture and team collaboration.
- **Strong governance:** Contracts can be reviewed, approved, and versioned separately.
- **CI/CD integration:** Fits well into DevOps workflows and API Management tools like Azure APIM.
- **Prevents errors early:** Contract validation before implementation reduces mismatches and surprises in later stages.

**❌ Cons:**

- **Higher complexity:** Requires maintaining contracts separately from the codebase.
- **Learning curve:** Teams unfamiliar with contract-first design may need time to adopt it.
- **Risk of drift:** If not strictly enforced, code and contract can become out of sync.

------

### ⚙️ **Compile-Time OpenAPI Generation**

**✅ Pros:**

- **Automatic alignment:** The OpenAPI spec is generated directly from the code, always in sync.
- **Simplicity:** Less tooling required, ideal for teams focused on delivery speed.
- **Native support:** Supported out of the box in .NET 9, especially for Minimal APIs.
- **Better performance:** No runtime overhead; the spec is generated once at build time.

**❌ Cons:**

- **Less control:** May expose unintended details unless configured correctly.
- **Limited customization:** Harder to fine-tune the output compared to a hand-written contract.
- **Code-coupled:** Bugs or inconsistencies in code affect the generated contract directly.

------

### 🕒 **Runtime OpenAPI Generation**

**✅ Pros:**

- **Always up-to-date:** The documentation reflects the current state of the running application.
- **Easy to implement:** Often just requires adding libraries like Swashbuckle or NSwag.
- **Great for dev/testing:** Enables instant feedback and interactive testing tools like Swagger UI.

**❌ Cons:**

- **Performance impact:** Generating documentation at runtime can affect startup or request latency.
- **Security risk:** Could accidentally expose sensitive endpoints or internal APIs in production.
- **Not production-friendly:** Generally recommended only for development environments.

------

### 📊 **Comparison Table**

> ⚠️ **Note:** A “No” in this table doesn't necessarily imply a disadvantage. For example, the lack of separation in compile-time generation simplifies workflows and reduces overhead — which might be a benefit depending on your team and context.

| Feature                   | ApiOps (Contract-First) | Compile-Time (.NET 9)       | Runtime                |
| ------------------------- | ----------------------- | --------------------------- | ---------------------- |
| Contract/Code Separation  | ✅ Yes                   | ❌ No                        | ❌ No                   |
| Governance                | ✅ Strong                | ❌ Limited                   | ❌ Limited              |
| Ease of Implementation    | ❌ Complex               | ✅ Simple                    | ✅ Simple               |
| Customization Flexibility | ✅ High                  | ❌ Limited                   | ❌ Limited              |
| Automatic Sync with Code  | ❌ Manual                | ✅ Automatic                 | ✅ Automatic            |
| Risk of Data Exposure     | ✅ Low                   | ❌ Medium (if misconfigured) | ❌ High (in production) |
| Production-Readiness      | ✅ Yes                   | ✅ Yes                       | ❌ No                   |

------



## 🔍 Comparing RunTime vs Compile-Time  Open Api Generation in NET 9

Typically, runtime generation of Open API documentation has some prerequisites:

- Your Web API must be deployed before the code: It must be UP & RUNNING. 

  There's a waiting period where the app might not be immediately available due to deployment.

   And not always what you deploy works (e.g. missed configuration...) :) 

- You must call the endpoint that exposes it, possibly from your CD-pipeline/agent in a complex security context (e.g., route openings, inbound rules.... ).

Fortunately, there's now an easier way that avoids runtime generation of the documentation.

If you’re using .NET 9 Web API, in some scenarios — instead of using an ApiOps methodology — it might be more effective and efficient to generate the Open API documentation at compile time.

The Open Api Generation works with a mock server (According to Microsoft documentation):

> "Build-time OpenAPI document generation functions by launching the app’s entry point with a mock server implementation. A mock server is required to produce accurate OpenAPI documents because all information in the OpenAPI document can't be statically analyzed."



## Getting Started: Implementing Open Api Generation at Compile Time in a .NET 9 Web Api

1. On your .NET 9 project, install these two NuGet packages:

    - `Microsoft.AspNetCore.OpenApi`
    - `Microsoft.Extensions.ApiDescription.Server`

2. In your `Program.cs`, add the service extension method: `AddOpenApi();`

    ![openapi](.\img\openapi-example2.png)

3. And add the middleware: `app.MapOpenApi();`

    ![openapi](.\img\openapi-example3.png)

If you want to expose an Open API UI for a developer-friendly experience, you can add Scalar with `app.MapScalarApiReference()` to your web app. The UI will be available at a URL like `https://localhost:port/scalar/v1`.
In alternative you can use SwaggerUI. 
Or both :) ...  

4. Add this to your `.csproj` file:

```xml
<PropertyGroup>
  <OpenApiGenerateDocuments>true</OpenApiGenerateDocuments>
  <OpenApiDocumentsDirectory>bin\$(Configuration)\$(TargetFramework)</OpenApiDocumentsDirectory>
  <OpenApiGenerateDocumentsOptions>--file-name open-api</OpenApiGenerateDocumentsOptions>
</PropertyGroup>
```

This code will generate a file called `open-api.json` in your `bin` folder — that’s your documentation.

![openapi](.\img\openapi-example4.png)

## What if Dependency Injection (DI) is Broken or Requires Custom Config?

If that’s the case, it won’t work...  :(

You’ll need to exclude the problematic code in `Program.cs`. 

How we can do it? Here's a static method I created in a library:

```csharp
public class BuildTimeOpenApiGeneration
{
    public static bool IsOpenApiMockServer()
    {
        bool isOpenApiGeneratedAtCompileTime = false;
        try
        {
            isOpenApiGeneratedAtCompileTime = Assembly.GetEntryAssembly()?.GetName().Name == "GetDocument.Insider";
        }
        catch
        {
            isOpenApiGeneratedAtCompileTime = false;
        }
        return isOpenApiGeneratedAtCompileTime;
    }
}
```

This method wraps the potentially breaking code.

In build-time, the .NET mock server will skip any code in `Program.cs` marked to be excluded.

![openapi](.\img\openapi-example5.png)

Alternatively, you can use the `AddServiceDefaults` extension method in `Program.cs` which, per documentation, “adds common .NET Aspire services such as service discovery, resilience, health checks, and OpenTelemetry.”

```csharp
if (Assembly.GetEntryAssembly()?.GetName().Name != "GetDocument.Insider")
{
    builder.AddServiceDefaults();
}
```

## CI Step

After building your .NET 9 project in CI, you can publish a new artifact containing your documentation.
Please note that I use some local variables such as a variable to indicate where to retrieve the SourceCode to build...

```yaml
steps:
    - task: DotNetCoreCLI@2
      displayName: 'Build .NET'
      inputs:
        command: build
        projects: $(SourceFolder)/*.sln
        arguments: '--configuration $(build_configuration)'
        verbosityRestore: Detailed
        verbosityPack: Detailed

    - task: CopyFiles@2
      displayName: 'Copy OpenAPI JSON to publish directory'
      inputs:
        SourceFolder: '$(SourceFolder)'
        Contents: '**/bin/$(build_configuration)/**/open-api.json'
        TargetFolder: '$( TargetFolder)'
        flattenFolders: true

    - task: PublishBuildArtifacts@1
      displayName: 'Publish Artifact: openapi.json'
      inputs:
        PathtoPublish: '$( TargetFolder)/open-api.json'
        ArtifactName: $(artifactName)-openapi
      condition: succeeded()
```

## CD  Step 

In this example, deployment is to Azure API Management Service (apim). The shell script is called from a DevOps task. Here's a `template.yaml` and shell script. But remember that you could release your documentation anywhere. :)
**Open Api is a standard!**
(aH! By default, the standard used is open api spec v.3 (https://spec.openapis.org/oas/v3.0.0), but you can change it! see https://learn.microsoft.com/en-gb/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-9.0&tabs=visual-studio%2Cvisual-studio-code#customize-the-openapi-version-of-a-generated-document)

```yaml
parameters:
- name: serviceConnection
  type: string
  
  #Your Apim Resource Group
- name: apim_resourceGroup
  type: string
  #Your Apim Resource Name
- name: apim_ServiceName
  type: string
  #the api path for the apim api
- name: apim_apiPath
  type: string
  #the api Id for the apim api
- name: apim_apiId
  type: string
  #the path at your generated at compile time json :)
- name: specificationPath
  type: string
  #the api webserviceUrl --> open api deploy can override the backend url setted on you api policy... then I added it!]
- name: webServiceUrl
  type: string  

steps:
- task: DownloadPipelineArtifact@2
  displayName: "Download Open API JSON from artifact"
  inputs:
    buildType: 'specific'
    artifactName: $(artifactName)-openapi
    targetPath: $(artifact)
    project: '$(DEVAZURE-PROJECT-NAME)'
    definition: $(ID-PIPELINE-BUILD)
    buildVersionToDownload: 'specific'
    pipelineId: $(runIdBuildPipeline)

- task: AzureCLI@2
  displayName: "APIM API import from Artifact Code"
  inputs:
    azureSubscription: ${{ parameters.serviceConnection }}
    scriptType: 'bash'
    scriptLocation: inlineScript
    inlineScript: |
      export apimServiceName="${{ parameters.apim_ServiceName }}"
      export resourceGroupName="${{ parameters.apim_resourceGroup }}"
      export apiPath="${{ parameters.apim_apiPath }}"
      export apiId="${{ parameters.apim_apiId }}"
      export specificationPath="${{ parameters.specificationPath }}"
      export webServiceUrl="$(echo ${{ parameters.webServiceUrl }} | awk '{print tolower($0)}')"

      chmod a=rwx $(myscriptpath)/ImportApimFromCompileTime.sh
      . $(myscriptpath)/ImportApimFromCompileTime.sh
```

**Shell Script:**

```bash
#!/bin/bash

echo "##[info]apimServiceName=$apimServiceName"
echo "##[info]resourceGroupName=$resourceGroupName"
echo "##[info]apiPath=$apiPath"
echo "##[info]apiId=$apiId"
echo "##[info]specificationPath=$specificationPath"

specificationFormat=OpenApiJson

echo "Importing API..."

if [ ! -f "$specificationPath" ]; then
    no_env_file="Does not exist a file with path [$specificationPath]!"
    echo "##[error]$no_env_file"
    echo "##vso[task.logissue type=error;sourcepath=$specificationPath;]$no_env_file"
    exit 1
fi

az apim api import     --path $apiPath     --resource-group $resourceGroupName     --service-name $apimServiceName     --api-id $apiId     --display-name $apiId     --specification-format $specificationFormat     --specification-path $specificationPath     --service-url $webServiceUrl
```



## 🧩 Conclusion

There’s much more to explore — we could dive into managing APIM policies or advanced security validation. But the purpose of this article was to showcase how this **new .NET 9 feature simplifies OpenAPI handling in DevOps workflows** .

*Why only now?*
Microsoft's timing for introducing compile-time OpenAPI generation likely reflects both technical complexity and market readiness. Historically, Microsoft prioritizes stability and gradual innovation, especially in enterprise contexts. Additionally, the increased maturity of DevOps practices, community feedback, and widespread adoption of standardized API governance models (like ApiOps) probably contributed to the timing. Thus, even though this feature might seem late, it aligns well with the broader industry's readiness to embrace such changes.

**At the end** "**What’s the right strategy"?**

While **ApiOps** remains the ideal approach for enterprises that require strict governance and contract reuse, **compile-time generation** is perfect for agile teams focused on fast iterations and minimal tooling but your project needs to be at least net9. As for **runtime generation**, it still has utility in development or debugging, but it's no longer ideal for production.


It depends on your team’s scale, delivery model, governance requirements and used stack. And remember — these approaches are not mutually exclusive. You can even combine them in a hybrid architecture.
