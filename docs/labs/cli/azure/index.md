# Running Accurics CLI

## Prerequisites

* Terraform version > .14
* An Accurics user account with an Operator or greater role
* An Azure subscription with enough permissions to create a resource and network security group
* Setup an environment on the Accurics Console to scan your IAC repository that you will be using to create the CI/CD builds
* Install the [Accurics CLI](../installing.md) suitable for your operating system

## Step 1: Log in to Azure

For Terraform to be able to successfully `plan`, you must first authenticate to Azure. There are multiple ways to do this, depending on your organizations preference. Please see the HashiCorp documentation links below for more information:

[Azure Provider: Authenticating using the Azure CLI](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli)

[Azure Provider: Authenticating using managed identities for Azure resources](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/managed_service_identity)

[Azure Provider: Authenticating using a Service Principal with a Client Certificate](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/service_principal_client_certificate)

[Azure Provider: Authenticating using a Service Principal with a Client Secret](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/service_principal_client_secret)

## Step 2: Download configuration file

Depending on your organization, you may already have a copy of your Accurics configuration file. If one hasn't been supplied to you, it can be downloaded from the Accurics Console.

1. Log into the Accurics console
2. Click the three vertical dots to open the menu for your environment
3. Click **Download Config**

    ![Download CLI](../../../assets/images/cli_download_config.png)

4. Save the file to your computer to a folder of your choosing.

## Step 3: Sample Terraform Code

Please copy the Terraform code below and save it as `azure_example.tf` to the same folder as the configuration file.

!!! danger
    The Terraform code below is an example of what could be considered a bad practice (opening SSH to the world) and should not be used in your environment.

```terraform
--8<-- "docs/assets/iac/azure/broken/azure_example_broken.tf"
```

## Step 4: `accurics init`

`accurics init` is a wrapper around `terraform init` that downloads all the required Terraform providers, and is required prior to running any futher commands.

```
> accurics init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/azurerm from the dependency lock file
- Installing hashicorp/azurerm v2.50.0...
- Installed hashicorp/azurerm v2.50.0 (signed by HashiCorp)

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## Step 5: `accurics plan`

We're at the point where it's time to scan the sample Terraform you saved above! Let's run `accurics plan` which is a wrapper around `terraform plan` that:

1. Runs a `terraform plan`
2. Runs an analysis that compares the Terraform code to the resources that Terraform will create
3. Generates a dependancy graph
4. Outputs JSON (`accurics_report.json`) and HTML (`accurics_report.html`) files listing any violations
5. Gives you a summary of how many resources are in the Terraform code, and number of violations sorted by severity
6. Uploads the results to the Accurics Console so they can be viewed online

To run the plan, open a Terminal/Command Prompt to the directory where the Terraform and configuration file is saved and run the following command:

!!! note
    You may have saved the configuration file to another location, please adjust the path accordingly. By default the Accurics CLI looks in `~/.accurics/config` and the CWD that `accurics plan` is executed from.

```
> accurics plan -config=config
2021/03/10 15:49:51 runPlan...
2021/03/10 15:49:51 [plan -out=1615412991082.out]
2021/03/10 15:50:03 Running Accurics analysis...
[snipped]/iac/azure
2021/03/10 15:50:03 mapping terraform resources to source code...
2021/03/10 15:50:03 Repo Root Path... [snipped]/iac/azure
2021/03/10 15:50:03 Current working directory ... [snipped]/iac/azure
2021/03/10 15:50:03 getting source code for all the resources present in '[snipped]/iac/azure'
2021/03/10 15:50:03 resources to source code mapping done!
2021/03/10 15:50:03 Creating dependency graph...
2021/03/10 15:50:03 GetDotFileUsingGraph Directory: [snipped]/iac/azure
2021/03/10 15:50:03 Using configuration file:-  [snipped]/config
----------------------------------------------------------------------------------------------------------------

Accurics successfully scanned the repository! Following is the summary - for details visit Accurics Web Console.

{
  "resources": 3,
  "violation": 2,
  "low": 1,
  "medium": 0,
  "high": 1,
  "native": 2,
  "inherit": 0,
  "drift": 0,
  "iacdrift": 0,
  "clouddrift": 0
}

----------------------------------------------------------------------------------------------------------------
```

## Step 6: Viewing results
The Accurics CLI outputs results in a few ways.

1. To `stdout` as a summary of the *quantity* and *severity* of what was found.
2. A JSON blob in the directory you ran the Accurics CLI from
3. An HTML file that is also in the directory you ran the Accurics CLI from

The HTML is probably the most human readable format, so let's open that and see what it says!

Do you see the violation for `SSH (TCP:22) is exposed to the entire public internet`? Let's try fixing that and rerunning `accurics plan`

!!! note
    In this lab, there will always be an policy violation for Azure Resource Groups. This is because the code needs to be deployed to the cloud for Accurics to detect the lock.

    * Resource Manager Locks allow administrators to lock down Azure resources and prevent deletion or changing of resources. You can set the lock level to CanNotDelete or ReadOnly. In the portal, the locks are called Delete and Read-only respectively. It is recommended to have locks enabled to prevent accidental or malicious change or deletion.
    * Ensure that Azure Resource Group has resource lock enabled

## Step 7: Fixing bad IaC and rescanning

Please remediate your Terraform with the highlighted changes below:

```terraform hl_lines="32"
--8<-- "docs/assets/iac/azure/good/azure_example_good.tf"
```

Let's rerun `accurics plan` and see how many violations it's found after fixing the two violations:

```json
{
  "resources": 4,
  "violation": 1,
  "low": 1,
  "medium": 0,
  "high": 0,
  "native": 1,
  "inherit": 0,
  "drift": 0,
  "iacdrift": 0,
  "clouddrift": 0
}
```
