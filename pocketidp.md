# PocketIDP: Experimenting with Platform Orchestrators

We're going to look at an example of an IDP powered by a Platform Orchestrator in this tutorial. For fun, we like to call this the `5 Minute IDP` or `Pocket IDP`. 

This type of IDP separates the "buckets" of Developer and Operations/Infrastructure teams; the idea is that the Developer experience is abstracted *away* from configs and infrastructure short of declaring dependencies. The purpose of a Platform Orchestrator is to connect the Developer's application source code to the Platform source code laid out by Platform/infra teams.

You'll be invited to an instance of [Coder](https://coder.com) at the end of this week's lecture. Please let us know if you're unable to access it by emailing us. Using the Coder setup, we'll scaffold an IDP with the Humanitec Platform Orchestrator at the centre, and set up a local environment to test in.

### Prerequisites
* A Coder account for the PE Course instance
* [Coder CLI](https://coder.com/docs/install/cli) installed

Note: Unlike previous versions of the PocketIDP Coder instance, this time, a Humanitec free trial account will automatically be created for you. You will receive an invite to the email you registered for the course with (which is the one you use to log in to Coder); *you need to accept this in order to go through any of the PocketIDP tutorials!*

### Getting Started
Inside of Coder, you'll see tabs for `Workspaces` or `Templates`. Navigate to the `Templates` tab and select the `pe-course-pidp-xe` template.

Fill out the form with your required details; make sure to use the Humanitec Org ID from the free trial account you created earlier (from `https://app.humanitec.io/orgs/<YOUR-ORG-ID>/resources/definitions`), as well as the Service User API token from the prerequisites. 

***We recommend you select the region nearest you, and the middle or largest available instance type.***

Hit `Create Workspace` when you're ready, and wait until the instance is completed. *This can take 10+ minutes; please be patient!*

### Setup
The Coder workspace will do a few things; it will create a Virtual Machine and inside of it spin up a Kind cluster, a Gitea instance, as well as prepare Backstage and other pre-built configurations. Additionally, it will create environment types, resource definitions, and applications in your Humanitec account.

Once the Coder workspace is ready, you'll need to do a few things in order to access it. 

First, login to the PlatformEngineering.org Coder environment:
```
coder login https://sandbox.platformengineering.org/
```

Next, you'll want to set up port forwarding to be able to access the various components of the Coder setup through your browser. Do this in your terminal by running the following command and swapping in the name of your coder instance at `<CODER-WORKSPACE-NAME>`:

```
coder port-forward <CODER-WORKSPACE-NAME> --tcp 8443:443
```

Now, we can get ready to access the Coder instance itself. Start by running the following in a *new terminal window*:
```
coder config-ssh
```

Follow the prompts, and then use the next command to actually login to the remote Coder-hosted machine:
```
ssh coder.<CODER-WORKSPACE-NAME>
```

So we can do a few actions later with the Humanitec CLI, run `humctl login` and follow the prompted instructions.

Once this is done, you'll now be able to go to the Gitea instance through your browser. Go to: https://git.localhost:8443

Sign in to Gitea with the username and password. Both are pre-set as: `5minadmin`

### Setting Up Backstage and Scaffolding an Application
Find the Backstage repository in Gitea, and make a change in the readme. This will be deployed via Actions. You can watch the progress in of the “Actions” tab (similar to GitHub Actions) in the repository on Gitea.

Once this is completed, go into [Humanitec’s UI](https://app.humanitec.io), visit the Applications screen, and click through to the appropriate application name for Backstage. This should be formatted similar to `5min-backstage-lgjd`. Then, click on the `5min-local` environment and grab the URL generated at the bottom of the `Current Deployment` section. This URL is likely similar in format to `https://5min-backstage-lgjd-vtoy.localhost/`. To access this in your browser, make sure to add `:8443` to the end (ex: `https://5min-backstage-lgjd-vtoy.localhost:8443/`)

You’ll only need guest access to continue from here; click the button in Backstage to login. Click “Create”, and create a new service from one of the two available templates. **Note the name of your application!**

You’ll be able to view the repository of this created app, see it in Backstage, and it also now will show up in the Humanitec UI. Wait for it to finish deploying before accessing the app via the URL in the Humanitec UI which will be the app name you selected in Backstage plus a random string of letters — just like before, make sure to add `:8443` to the end of the URL to access (ex: `https://your-app-abcd.localhost:8443`).


## PE Golden Path: Add a New Database Resource
We’re going to now simulate the PE experience by adding a new available database resource type to our platform. For this, we’ll be working in Humanitec to set up a new Resource Definition.

Let’s add a Postgres, to be made available specifically to our newly-scaffolded application inside of the 5min-idp environment type.

We are going to use [this](https://developer.humanitec.com/examples/resource-packs/in-cluster/postgres/) in-cluster Postgres example for this exercise. We’re also going to make a resource class, which is a great way to give developers options for different possible configurations of a particular resource type.

### Step 1: Create the Resource Class

Run the following command inside of your terminal:

```
humctl api post "/orgs/${HUMANITEC_ORG}/resources/types/postgres/classes" \
  -d '{
  "id": "5min",
  "description": "Specifically to be used when using 5min IDP settings"
}'
```

If successful, this API call should return something like the following:

```
{
  "created_at": "2025-01-26T23:36:48.798306702Z",
  "created_by": "b428c48c-cb11-4223-b513-716852197eb9",
  "description": "Specifically to be used when using 5min IDP settings",
  "id": "5min",
  "resource_type": "postgres"
}
```

Now, let’s create the new resource definition!

### Step 2: Create the Postgres Resource Definition

Clone the following repository inside the Coder instance:

`git clone https://github.com/humanitec-architecture/resource-packs-in-cluster`

Navigate into this folder:
`cd /resource-packs-in-cluster/examples/postgres`

If you didn’t login to humctl earlier, do so now with `humctl login`

Next, we’ll use Terraform inside this folder to generate the resource definitions:

```
terraform init
terraform plan
terraform apply
```


Once this is done, you can confirm that the Resource Definition has been created via the Humanitec API, or by checking the list of available resource definitions using the Humanitec CLI with `humctl get resource-definitions`

The created Resource Definition will be named `hum-rp-postgres-ex-postgres` or something similar.

Let’s now make sure that we can use this Resource Definition inside our 5min-idp environment type and using a the resource class we created earlier — setting this is called working with Matching Criteria.

To do this, we’ll run the following CLI command, making sure to replace `<resource-definition-id>` with the name of your Resource Definition:

```
humctl api PUT /orgs/${HUMANITEC_ORG}/resources/defs/<resource-definition-id>/criteria --data '[{"env_type": "5min-local","class":"5min"}]'
```

If you receive a return that looks like this, you’re all done!

```
[
    {
        "class": "5min",
        "env_type": "5min-local",
        "id": "3969f0f56557414e"
    }
]
```


You can also validate this inside the UI by finding the resource in the Resource Definitions tab, and selecting Matching Criteria.

That’s it! Now, let’s take on the role of developer and make use of the resource definition we just created.


## Dev Golden Path: Request a Resource With a Deployment
Now, we’re going to simulate developer experience and request a Postgres database with our deployment. This will use our newly-created resource definition and class to spin up a Postgres instance for us, without us having to do any config work!

Let’s go back to our local Gitea instance and find the repository for our scaffolded application. You can do this directly from Backstage, or by going to https://git.localhost:8443/

For this example we are going to edit a file right inside of git - obviously, though, don’t do this in real life!

Find the `score.yaml` file and open it for editing. Look for the section called `resources` - it should look similar to this:

```
resources:
  dns: # We need a DNS record to point to the service 
    type: dns
  route:
    type: route
    params:
      host: ${resources.dns.host}
      path: /
      port: 80
```
For our deployment, we know that we want to be able to request a Postgres database. Let’s make sure this is available to us in our current context!

Go back to your terminal, and run the following command:

```
humctl score available-resource-types
```
This should return a list of all resource types available to us, including any specified classes, similar to the following:
```
Name             	Type       	Category 	Class  	Description                                         
Environment      	environment	score    	default	-                                                   
Service          	service    	score    	default	-                                                   
DNS              	dns        	dns      	default	-                                                   
Persistent Volume	volume     	datastore	default	-                                                   
Route            	route      	ingress  	default	-                                                   
TLS certificate  	tls-cert   	security 	default	-                                                   
Postgres         	postgres   	datastore	5min   	Specifically to be used when using 5min IDP settings
Postgres         	postgres   	datastore	default	-                                                   
MySQL            	mysql      	datastore	default	-    
```
We can see we have _two_ different class of Postgress available: `default` and `5min`. For this particular exercise, let's use the `5min` class!

Going back to the Git repository and the `score.yaml` file, we're going to add three lines under resources. Your `score.yaml` file should now look similar to this:
```
resources:
  dns: # We need a DNS record to point to the service 
    type: dns
  route:
    type: route
    params:
      host: ${resources.dns.host}
      path: /
      port: 80
  database: # Adding our Postgres database and associated class
    type: postgres
    class: 5min
```
Commit the change to the `main` branch, and pop over to the Actions tab to follow the deployment. You can also follow the deployment in the Humanitec UI!

If you're looking inside the Humanitec UI, you'll be able to see the Resource Graph now show the Postgres database with the appropriate clsas we created.

The Developer experience is that simple!

## Video Tutorials
* [How to get started with Coder + PocketIDP](https://drive.google.com/file/d/10DfS4XS_AP5C7a-GAgbnv2JGIZr7zeEI/view?usp=drive_link)
* [Quick reminder](https://drive.google.com/file/d/1vRg-S8YVGBM5CcKlr8cN2ynTOXq359d7/view?usp=drive_link)
* [Scaffold an app](https://drive.google.com/file/d/1gjX42K9oBks6wd9smHncuR409bF9B_HK/view?usp=drive_link)
* [Add a resource type](https://drive.google.com/file/d/15gZYh-e9OVZY7Ayg9812z8xvhfn8blKX/view?usp=drive_link)
* [Request a new dependency](https://drive.google.com/file/d/1gxIi1PYF_kKlRdTBw_wsh_zU5_N06b_z/view?usp=drive_link)
* [Troubleshooting](https://drive.google.com/file/d/1yinDyRVXsekhUbsjPqIPGq3ObjaIUlcl/view?usp=drive_link)
* [Ephemeral Environments](https://drive.google.com/file/d/1PiAGgzzpnBd2a1OwQcTxA_xDy4i1HyZn/view?usp=drive_link)
* [Rollbacks](https://drive.google.com/file/d/1OefxvfA4ipVeZ7twtGHEDK58IuZfEDqi/view?usp=drive_link)

## Experiment Further: Next Steps 🚀

This tutorial is just a taste of what a powerful IDP using vendor-based services and a Platform Orchestrator can look like.

If you want to go beyond the Pocket IDP, you can find a lot of useful advanced use cases [here](https://developer.humanitec.com/training/master-your-internal-developer-platform/introduction/).

