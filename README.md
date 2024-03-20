# Installation

## Install func-cli
to be downloaded from https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Cisolated-process%2Cnode-v4%2Cpython-v2%2Chttp-trigger%2Ccontainer-apps&pivots=programming-language-csharp

## Install VS Code extension
Extension to be installed: `Azure Functions`


# Dev
## Create a project

```shell
func init func-hello-world-python-v2 --worker-runtime python --model V2
# Found Python version 3.9.13 (python3).
# The new Python programming model is generally available. Learn more at https://aka.ms/pythonprogrammingmodel
# Writing requirements.txt
# Writing function_app.py
# Writing .gitignore
# Writing host.json
# Writing local.settings.json
# Writing C:\Users\amine.jallouli\Documents\projects\func-hello-world-python-v2\.vscode\extensions.json
```
Then, Navigate into the project folder:
```shell
cd func-hello-world-python-v2
```
## Create a function
Add a function to your project by using the following command, where the --name argument is the unique name of your function (HttpExample) and the --template argument specifies the function's trigger (HTTP).

```shell
func new --template "Http Trigger" --name HelloWorldHttpTrigger --authlevel anonymous
# Appending to C:\Users\amine.jallouli\Documents\projects\func-hello-world-python-v2\function_app.py
# The function "HelloWorldHttpTrigger" was created successfully from the "Http Trigger" template.
```

This command line modified the `function_app.py` as follows:

```py
import azure.functions as func
import datetime
import json
import logging

app = func.FunctionApp()

@app.route(route="HelloWorldHttpTrigger", auth_level=func.AuthLevel.ANONYMOUS)
def HelloWorldHttpTrigger(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello, {name}. This HTTP triggered function executed successfully.")
    else:
        return func.HttpResponse(
             "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.",
             status_code=200
        )
```

## Run the function locally

Run your function by starting the local Azure Functions runtime host from the `func-hello-world-python-v2` folder:

```shell
func start
```

PS: Ensure to create a python env as follows for the project:
```sh
# for creating the venv
python -m venv venv

# for activating venv
.\venv\Scripts\activate # for windows
```
## Create supporting Azure resources for your function
Before you can deploy your function code to Azure, you need to create three resources:

* **A resource group**, which is a logical container for related resources.
* **A Storage account**, which is used to maintain state and other information about your functions.
* **A function app**, which provides the environment for executing your function code. A function app maps to your local function project and lets you group functions as a logical unit for easier management, deployment, and sharing of resources.

```shell
# Login to Azure (just in case)
az login --tenant aminej.onmicrosoft.com

# 1. Creation of resource group
az group create --name rg-azure-function-crypto --location eastus

# 2. Create storage account
az storage account create --name storageazurefunccrypto --location eastus --resource-group rg-azure-function-crypto --sku Standard_LRS --allow-blob-public-access false 

# 3. Creation of functionapp in Azure
az functionapp create --resource-group rg-azure-function-crypto --consumption-plan-location eastus --functions-version 4 --name CryptoHelloWorldHttpTrigger --storage-account storageazurefunccrypto
## PS: --runtime: Runtime python not supported for os windows. Supported runtimes for os windows are: ['dotnet-isolated', 'dotnet-isolated', 'dotnet-isolated', 'dotnet-isolated', 'dotnet', 'node', 'node', 'node', 'node', 'java', 'java', 'java', 'powershell', 'powershell', 'custom']. Run 'az functionapp list-runtimes' for more details on supported runtimes.

# By default, the OS of the functionapp is Windows which does not support python functionapp.
# Consequently, deleting and recreating the function is needed.

# 3.1. Deleting the functionapp
az functionapp delete --name CryptoHelloWorldHttpTrigger --resource-group rg-azure-function-crypto

# 3.2. Creationg functionapp with OS=Linux and runtime=Python
az functionapp create --resource-group rg-azure-function-crypto --consumption-plan-location eastus --functions-version 4 --runtime python --name CryptoHelloWorldHttpTrigger --storage-account storageazurefunccrypto --os-type Linux
# Your Linux function app 'CryptoHelloWorldHttpTrigger', that uses a consumption plan has been successfully created but is not active until content is published using Azure Portal or the Functions Core Tools.
# Application Insights "CryptoHelloWorldHttpTrigger" was created for this Function App. 

# 4. Publish
func azure functionapp publish CryptoHelloWorldHttpTrigger 
# Getting site publishing info...
# [2024-03-20T16:21:46.376Z] Starting the function app deployment...
# Removing WEBSITE_CONTENTAZUREFILECONNECTIONSTRING app setting.
# Removing WEBSITE_CONTENTSHARE app setting.
# Creating archive for current directory...
# Performing remote build for functions project.
# Uploading 4.91 KB [###############################################################################]
# Remote build in progress, please wait...
# Updating submodules.
# Preparing deployment for commit id '26323cf1-1'.
# PreDeployment: context.CleanOutputPath False
# PreDeployment: context.OutputPath /home/site/wwwroot
# Repository path is /tmp/zipdeploy/extracted
# Running oryx build...
# Command: oryx build /tmp/zipdeploy/extracted -o /home/site/wwwroot --platform python --platform-version 3.11 -p packagedir=.python_packages/lib/site-packages
# Operation performed by Microsoft Oryx, https://github.com/Microsoft/Oryx
# You can report issues at https://github.com/Microsoft/Oryx/issues

# Oryx Version: 0.2.20230210.1, Commit: a49c8f6b8abbe95b4356552c4c884dea7fd0d86e, ReleaseTagName: 20230210.1

# Build Operation ID: ff693b6d817b862c
# Repository Commit : 26323cf1-1c1a-4b19-b470-371591fa1001
# OS Type           : bullseye
# Image Type        : githubactions

# Detecting platforms...
# Detected following platforms:
#   python: 3.11.8
# Version '3.11.8' of platform 'python' is not installed. Generating script to install it...


# Source directory     : /tmp/zipdeploy/extracted
# Destination directory: /home/site/wwwroot


# Downloading and extracting 'python' version '3.11.8' to '/tmp/oryx/platforms/python/3.11.8'...
# Detected image debian flavor: bullseye.
# Downloaded in 2 sec(s).
# Verifying checksum...
# Extracting contents...
# performing sha512 checksum for: python...
# Done in 9 sec(s).

# image detector file exists, platform is python..
# OS detector file exists, OS is bullseye..
# Python Version: /tmp/oryx/platforms/python/3.11.8/bin/python3.11
# Creating directory for command manifest file if it does not exist
# Removing existing manifest file

# Running pip install...
# Done in 2 sec(s).
# [16:22:15+0000] Collecting azure-functions
# [16:22:15+0000]   Downloading azure_functions-1.18.0-py3-none-any.whl (173 kB)
# [16:22:15+0000] Installing collected packages: azure-functions
# [16:22:15+0000] Successfully installed azure-functions-1.18.0
# WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
# WARNING: You are using pip version 21.2.4; however, version 24.0 is available.
# You should consider upgrading via the '/tmp/oryx/platforms/python/3.11.8/bin/python3.11 -m pip install --upgrade pip' command.
# Not a vso image, so not writing build commands
# Preparing output...

# Copying files to destination directory '/home/site/wwwroot'...
# Done in 0 sec(s).

# Removing existing manifest file
# Creating a manifest file...
# Manifest file created.
# Copying .ostype to manifest output directory.

# Done in 12 sec(s).
# Running post deployment command(s)...

# Generating summary of Oryx build
# Deployment Log file does not exist in /tmp/oryx-build.log
# The logfile at /tmp/oryx-build.log is empty. Unable to fetch the summary of build
# Triggering recycle (preview mode disabled).
# Linux Consumption plan has a 1.5 GB memory limit on a remote build container.
# To check our service limit, please visit https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale#service-limits
# Writing the artifacts to a squashfs file
# Parallel mksquashfs: Using 1 processor
# Creating 4.0 filesystem on /home/site/artifacts/functionappartifact.squashfs, block size 131072.

# [===============================================================|] 151/151 100%

# Exportable Squashfs 4.0 filesystem, gzip compressed, data block size 131072
#         compressed data, compressed metadata, compressed fragments,
#         compressed xattrs, compressed ids
#         duplicates are removed
# Filesystem size 439.95 Kbytes (0.43 Mbytes)
#         27.61% of uncompressed filesystem size (1593.38 Kbytes)
# Number of socket nodes 0
# Number of directories 17
# Number of ids (unique uids + gids) 1
# Number of uids 1
#         root (0)
# Number of gids 1
#         root (0)
# Creating placeholder blob for linux consumption function app...
# SCM_RUN_FROM_PACKAGE placeholder blob scm-latest-CryptoHelloWorldHttpTrigger.zip located
# Uploading built content /home/site/artifacts/functionappartifact.squashfs for linux consumption function app...
# Resetting all workers for cryptohelloworldhttptrigger.azurewebsites.net
# Deployment successful. deployer = Push-Deployer deploymentPath = Functions App ZipDeploy. Extract zip. Remote build.
# Remote build succeeded!
# [2024-03-20T16:22:28.798Z] Syncing triggers...
# Functions in CryptoHelloWorldHttpTrigger:
#     HelloWorldHttpTrigger - [httpTrigger]
#         Invoke url: https://cryptohelloworldhttptrigger.azurewebsites.net/api/helloworldhttptrigger

```


