---
layout: post
title:  "The Azure Developer CLI"
date:   2023-10-31 11:26:18 +1000
categories: general

---

My findings about the Azure Developer CLI

## Installation

You can find out how to install it here:  
[https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd?tabs=winget-windows%2Cbrew-mac%2Cscript-linux&pivots=os-windows](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd?tabs=winget-windows%2Cbrew-mac%2Cscript-linux&pivots=os-windows)

I installed in on my Windows box:

```powershell
winget install microsoft.azd
```

## Why Would I use the Azure Developer CLI

If you are not skilled up on Azure Architecture and Best Practices it seems like the Azure Developer CLI is built with you in mind. Using an Azure Developer CLI template, you can use the Azure Developer CLI to create and deploy applications to the Azure Cloud. These templates adhere to best practices and take care of things like:

* The **infrastructure** that your application requires
* Your **application** code
* A **database** if you need it

Although I would not use this myself without understanding every file that I am deploying I could use it for the following reasons:

* Quickly deploy something I am not familiar with such as an Angular Website with a database and then learn from what I have done to increase my Angular skills. (**Note**: I am not a front end developer)
  * I could use this template and replace the front end and database schema with my own requirements and probably save myself days if not weeks!
* Quickly deploy a proof of concept for a client
* Learn about the Azure Developer CLI which I am doing here

## Azure Developer CLI templates

There are many open source templates available that are written by Microsoft and other developers. To see the available templates you can look at this link [Azure Developer CLI templates](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/azd-templates?tabs=csharp#choose-a-template). There is also a list of open source templates here [https://azure.github.io/awesome-azd/](https://azure.github.io/awesome-azd/) or run the following command:

```powershell
azd template list
```

I am going to attempt to use the following template: [https://github.com/Azure-Samples/todo-csharp-sql](https://github.com/Azure-Samples/todo-csharp-sql) which is described as:

> A blueprint for getting a React web app with a C# API and a SQL database on Azure. The blueprint includes sample application code (a ToDo web app) which can be removed and replaced with your own application code. Add your own source code and leverage the Infrastructure as Code assets (written in Bicep) to get up and running quickly.

Iam going to follow the instructions in the `README` [here](https://github.com/Azure-Samples/todo-csharp-sql/blob/main/README.md) and make write down what I do and find as I go.

## Steps and Summary of my learnings

* Ensure I have all the **Prerequisites** install on my machine - ✅
  * Regarding `Node.js` I opted to install the latest version [20.8.0](https://nodejs.org/dist/v20.8.0/node-v20.8.0-x64.msi)
* Log in to azd. `azd auth login`
  
  ```powershell
  azd auth login
  ```
* Make a new directory for `todo-csharp-sql` and navigate to it
* Run:
  
  ```powershell
  azd init --template Azure-Samples/todo-csharp-sql
  ```

* I was prompted to enter a new environment name and as I didn't totally understand what it was asking I enter `?` and the following was displayed:

  ![New Environment](../assets/azd_enter_env_name.png)

  * I entered: `todo-csharp-sql-dev` as now it was clear that I may want more than one environment in Azure such as `dev`, `test` and `prod` for example.

* After this I got a successful message: `SUCCESS: New project initialized!` ✅

* I then opened the folder I made earlier in VSCode and could see a list of all the files locally that were created by the template:

![Files created by the template](../assets/azd_file_system_in_vscode.png)

* I decided to push these to a new GitHub repository I just created so I can revert anything that I might break along this journey.

* Next I ran the following command to provision everything in Azure:

```powershell
# Provision and deploy to Azure
azd up
```

* Hopefully I end up with this:

  ![DesiredArchitecture](../assets/azd_architecture.png)

* I was then prompted for the following:
  * Azure subscription
  * Region - I put it in Brazil South as that is where I am sitting now
  * After that, everything was packaged up into zip files and deployed. This took about 15 minutes and to be honest, when does a Microsoft demo ever work 100%? Well it did:

  ![Success!](../assets/azd_up_deployment_success.png)

* If I look in Azure I can see all my resources:

  ![All Azure Resources Deployed](../assets/azd_all_azure_resources.png)

* Then I looked at the API that backs the website:

  ![API Swagger](../assets/azd_swagger.png)

* and then the website its self:

  ![Web Front End](../assets/azd_web_front_end.png)

* As you can see, I even added a `TODO` item expecting things to fail and everything worked as expected 😲

## Next Steps

Ok so that was a very nice experience but I don't want to ever really deploy from my local straight to Azure. I would prefer to deploy using **CI/ CD** so next, I will attempt to do this whilst still following the instructions [here](https://github.com/Azure-Samples/todo-csharp-sql/blob/main/README.md)

All `azd` templates include a default GitHub Actions and Azure DevOps pipeline configuration file called `azure-dev.yml``, which is required to setup CI/CD. This configuration file provisions your Azure resources and deploy your code to the main branch. You can find `azure-dev.yml`:

* For GitHub Actions: in the `.github/workflow` directory.
* For Azure DevOps: in the `azdo/pipeline` directory.

So here we go, let's try it:

* I will run the following command:

  ```powershell
  azd pipeline config
  ```

  * During the running of this command:
    * I was asked to Authenticate to GitHub
    * I was asked to Choose a repository - I chose the existing one mentioned above in this post
    * It set up my GitHub action secrets that conainted the Azure service pricipal for example:

    ![GitHub action secrets](../assets/azd_github_action_secrets.png)

  * here is the actual successful deployment:

    ![Successful Deployment](../assets/azd_github_action_actual_deployment.png)  

## Finally

Tear everything down as this is deployed to my personal Azure subscription and I don't want to pay for it.

```powershell
azd down 
```

As you can see in the following picture, all my resources have be deprovisioned:

![AZD Down](../assets/azd_down.png)

## And in Summary

I don't think I have ever spent 6 hours on a Microsoft demo without something not working so this is a first. From start to finish everything was so easy and worked first time. 

## Useful Links

* [Overview](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/overview)
* [Installation](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd?tabs=winget-windows%2Cbrew-mac%2Cscript-linux&pivots=os-windows)
* [Azure Developer CLI templates](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/azd-templates?tabs=csharp#choose-a-template)
* [Configure a pipeline and push updates](https://learn.microsoft.com/en-gb/azure/developer/azure-developer-cli/configure-devops-pipeline?tabs=GitHub)
* [Monitor your app using Azure Developer CLI](https://learn.microsoft.com/en-gb/azure/developer/azure-developer-cli/monitor-your-app) - ⌛
