# CNOE: An Open-Source IDP Framework

Our first hands-on demo will make use of the [CNOE Open Source IDP Framework](https://cnoe.io/), and will give you a good understanding of the fundamentals of how to setup an IDP for simple use cases. We will be using the [CNOE Reference Implementation](https://cnoe.io/docs/reference-implementation/integrations/reference-impl) as the foundation for this tutorial!

The CNOE framework is setup to function in a more traditoinal "devops" context; there isn't a separation of concerns between developers and operations in this particular instance. With that in mind, this is great for small teams and applications.

You'll be invited to an instance of [Coder](https://coder.com) at the end of this week's lecture. Please let us know if you're unable to access it by emailing us. Using the Coder setup, we'll scaffold an IDP centered around a Kind cluster on a virtual machine.

### Prerequisites
* A Coder account for the PE Course instance
* [Coder CLI](https://coder.com/docs/install/cli) installed

### Getting Started
Inside of Coder, you'll see tabs for `Workspaces` or `Templates`. Navigate to the `Templates` tab and select the `pe-course-cnoe` template.

Fill out the form with your required details and take note of your workspace name. *Do not change the repository URL.*

***We recommend you select the region nearest you, and the middle or largest available instance type.***

Hit `Create Workspace` when you're ready, and wait until the instance is completed. *This can take 10+ minutes; please be patient!*

### Setup
The Coder workspace will do a few things; it will create a Virtual Machine and inside of it spin up a Kind cluster, a Gitea instance, as well as prepare Backstage and other pre-built configurations. 

Once the Coder workspace is ready, you'll need to do a few things in order to access it. 
```
coder port-forward <CODER-WORKSPACE-NAME> --tcp 8443:443
```

Now, we can get ready to access the Coder instance itself. Start by running the following:
```
coder config-ssh
```

Follow the prompts, and then use the next command to actually login to the remote Coder-hosted machine:
```
ssh coder.<CODER-WORKSPACE-NAME>
```

The Coder setup has already set up the reference implementation for Coder and set up URLs to access each:
* Argo CD: https://cnoe.localtest.me:8443/argocd
* Argo Workflows: https://cnoe.localtest.me:8443/argo-workflows
* Backstage: https://cnoe.localtest.me:8443/
* Gitea: https://cnoe.localtest.me:8443/gitea
* Keycloak: https://cnoe.localtest.me:8443/keycloak/admin/master/console/

## Using CNOE
In order to use any of the components installed above, we'll need to get some credentials. In order to do this, you'll want to use the following command:
```
idpbuilder get secrets
```
This will return an output similar to:
```aiignore
Name: argocd-initial-admin-secret
Namespace: argocd
Data:
  password : <password>
  username : <user>
---------------------------
Name: gitea-credential
Namespace: gitea
Data:
  password : <password>
  token : <token>
  username : <user>
---------------------------
Name: keycloak-config
Namespace: keycloak
Data:
  KC_DB_PASSWORD : <password>
  KC_DB_USERNAME : <user>
  KEYCLOAK_ADMIN_PASSWORD : <password>
  POSTGRES_DB : <user>
  POSTGRES_PASSWORD : <password>
  POSTGRES_USER : <user>
  USER_PASSWORD : <password>
```

## Step 1: Simple Deployment with Backstage
The first thing we'll do is scaffold an application via a simple deployment using Backstage. Visit the [local Backstage instance](https://cnoe.localtest.me:8443/) to get started and login with `user1` and the password from the `USER_PASSWORD` variable in the secrets output above.

On the left-hand menu, hit `Create`. Select the `Create a Basic Deployment` option from the available pre-configured services. Name your service, and create it!

You'll want to take a look at the scaffolded application inside [Gitea](https://cnoe.localtest.me:8443/gitea); the login information is also available in the portion of the secrets file starting with `gitea-credential`.

Once this service is deployed, you also should take a look at it inside of [Argo CD](https://cnoe.localtest.me:8443/argocd) by using the credentials under `argocd-initial-admin-secret` in the secrets output.

## Step 2: Adding a New Resource
Now, we'll do something a little bit more complex; we'll now deploy a new application with associated cloud resources. In this case, we're going to deploy an S3 bucket.

Make sure that Crossplane is installed into your CNOE IDP by following [these instructions](https://github.com/cnoe-io/stacks/blob/main/crossplane-integrations/README.md). Once that's been confirmed, select `Add a Go App with AWS Resources` in the `Create` menu on Backstage.

Follow the walk-through to deploy the new application; note that an actual AWS S3 resource will _not_ be deployed as the actual credentials to an AWS account are missing. If you would like to try this out, you'll need to [update the credentials file](https://cnoe.io/docs/reference-implementation/integrations/reference-impl#:~:text=the%20credentials%20secret%20file) accordingly.

View this app in Backstage to understand what pieces were scaffolded with the application. 

### Going Beyond the Basics
There are a lot of other things you can do with CNOE, as its fairly extensible. Note, however, that there is less separation between the developer and operations experience; keep in this in mind as you explore it!

Some other key components to check out:
* [Plugins](https://cnoe.io/docs/category/plugins)
* [CNOE CLI](https://cnoe.io/docs/reference-implementation/integrations/generated)
* [More about CNOE](https://cnoe.io/blog/welcome)