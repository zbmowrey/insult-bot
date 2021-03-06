# Insult-Bot
A Slack Bot which responds to Slash Commands with one of many possible insults. 

## The Future

This bot started life as a tribute to Bertram Gilfoyle from a very popular show on a very popular network. 
While that purpose continues, we intend to extend it to provide for a variety of characters and quotations, 
on both random or filtered bases. 

Future usage will be something like: 

    /insult (no additional input)
    ... triggers us to pick a character at random, then move to that character's pointer and return a response.

    /insult gilfoyle
    ... the same as above, without the randomness.
    
The intent is that we'll perform a partial match and take the first result, then jump to that character's pointer
in the current quotation queue, returning one of their quotations.

I'm not sure yet how we can successfully handle a case where multiple characters match aside from the above. 
Suggestions are definitely welcomed. 

## Setup

### Backend

Rename .github/workflows/terraform/backend.tf.example (drop .example from the name). 

Then configure the backend in that file to your liking. 

### Terraform Vars File

Rename .github/workflows/terraform/terraform.tfvars.example (drop .example from the name). 

Edit this file to provide values for the following: 

    organization - the Terraform Cloud organization you'll be using for local testing. 
    workspace - the workspace PREFIX you'll use. I append the Environment string to the end of this to name the workspace.
    owner - the person responsible for managing the app in your infrastructure. 
    cost-center - a string to help you find this app's costs in billing reports. 
    app-name - a name for the application. This will affect most resource naming. 
    insults-store - the name of the dynamodb table we'll create. 
    insults-url - a https:// webhook pointing to Slack that allows the bot to post messages. 

### Workspace(s)

The full production workflow of this app makes use of **main**, **develop**,
and **staging** branches. Most developers will not need to use all 3 of these.
I recommend that you start with just *develop* for now. 

My convention has been to use **workspace names** that identically match the
**branch name** and my **AWS profile names**. This allows me to switch workspaces and
automatically point to the correct AWS account, and to tag/name resources with
the appropriate environment name. 

Run `terraform workspace new develop` and then `terraform init`. Then go into Terraform 
Cloud and **update the settings on your workspace**. You want to run deployments 
LOCALLY rather than remote. We do this because TF Cloud isn't aware of the workspace name. 

### AWS Creds

I recommend that you do not rely on long-lived key/secret pairs for authentication. Github
supports OpenID Connect, as does AWS. You can find an example TF configuration 
[here](https://github.com/zbmowrey/cloud-admin/blob/main/deployment-to-develop.tf).

Combining the above with SSO & MFA on your aws-cli will allow you to remove access keys
from all IAM users and improve your account security. 

### Github Secrets

Create this secret and populate it with your tf cloud token:

    TERRAFORM_CLOUD_TOKEN

Provide AWS_DEVELOP_ACCOUNT, AWS_STAGING_ACCOUNT, and AWS_MAIN_ACCOUNT ids (numerical)
for each of the environments you intend to use.

Populating these values will allow Github to automatically deploy to your AWS account(s)
whenever code is merged/pushed to develop, staging, or main branches. You can watch
the deployment process in the Actions tab of your repository. 

### Terraform Apply

Terraform apply will deploy all necessary infrastructure for this and will output the endpoint your bot needs. When the endpoint receives a POST from a Slack Slash Command, it will respond by sending a POST request to the slack-url you specify in terraform.tfvars. 
