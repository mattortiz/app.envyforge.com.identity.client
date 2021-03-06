# Envy Forge Identity Client

This single page application (Identity Client) is designed to work the with the accompanying Identity Service. The application is based on an AWS Quickstart called [SaaS identity and isolation with Amazon Cognito](https://aws.amazon.com/quickstart/saas/identity-with-cognito/).

Some difference between the AWS Quickstart and this implementation are the following:
* An install option was added to the application to allow creation of the initial system Tenant
* The application uses action + reducers to maintain the state while getting data from the microservices.
* It uses the service Discover process to get the URLs for each of the microservices. A single serviceURL is passed to the client and it then looksup the specific URL for each of the microservices.  So, if things are moved, the client would not have to be rebuilt.

## Prerequisites

You will need the following items installed before you can start the installation.
- Node.js v12.x or later
- Serverless CLI v1.9.0 or later. See [Serverless Quickstart](https://serverless.com/framework/docs/providers/aws/guide/quick-start/) for more details.
- An AWS account. A free tier account will work with minimal cost. The account should have admin permissions, or ability to create resources on AWS.
- Setup provider Credentials. See [Serverless Quickstart](https://serverless.com/framework/docs/providers/aws/guide/quick-start/) for more details.
- Clone this repository.
- **Install and deploy the identity service.** Instructions are in the **`<code-dir>/app.envyforge.com.identity/service/readme.md`**.

## Configuration

You will need to make a couple configuration changes before you build the application.

### Create Bucket and Wbsite Name

- Edit "serverless.yml" file. Change the following line at the top of the file to something unique. Note: the service property is used as part of a bucket name for the static website URL.

Change:
```
service: envyforgeidentity-client
```
to:
```
service: <yournameidentity-client>
```

### Change AWS region

- The client assumes this is being deployed in the us-west-2 region. If you installed the services in another region you will need to update the client configuration.
  - Edit the src/App/config.js file
  - this file has different sections based on stage. default is dev, so under dev: change the apiGateway.REGION to be a valid AWS region name.  

### Set the Service Discovery Endpoint

First we need to get the Identity Service Endpoint.
```
# in the <code-dir>/app.envyforge.com.identity/service/ directory 
cd serviceDiscovery
sls info -v 
```
- Find "ServiceEndpoint" under Stack Output. Copy the URL. it should look something like below where API_REST_ID is a unique code created by AWS.
``` 
https://API_REST_ID.execute-api.us-west-2.amazonaws.com/dev
```
- using the above URL, we need to update the src/App/config.js file.   

  - edit the src/App/config.js
  - change the 'YOUR_DEV_SERVICE_API_URL' to be the URL you got above. 
``` 
   SERVICE_URL: 'YOUR_DEV_SERVICE_API_URL',
```

## Install & Deploy the Client Application (Website)

We've made available a first time setup script that will install dependencies, build the client, deploy the S3 bucket, and sync the build output to the bucket. It simply makes multiple nmp calls that you will also find useful seperatly.

##### THIS WILL CREATE THE CLIENT APPLICATION AND DEPLOY THE NECESSARY AWS RESOURCES. RUN ON FIRST TYPE SETUP ONLY. 

- To build and deploy the client app to AWS, run the setup-client.sh from the **`<code-dir>/app.envyforge.com.identity/client/`** directory.

```
.../client > ./setup-client.sh
``` 

## Update the Application (Website)

If you make any code changes, you will need to build the application and they sync with the S3 bucket. The following commands will do this.
``` 
.../client > npm run build
.../client > sls s3sync
```

## Access the Application (Website)

Access the static website. Use the URL for the S3 Bucket. To get this URL enter the following command and get the URL from the "StaticWebsiteURL" under Stack Outputs
``` 
.../client > sls info -v
```
Copy and paste the URL into your browser.

## Use the Application

The client application works just like the AWS Quickstart [SaaS identity and isolation with Amazon Cognito](https://aws.amazon.com/quickstart/saas/identity-with-cognito/). You can use the guide for details about the application and services structure.

There are some slight differences between the two applications. 
* The AWS Quickstart uses CloudFormation to deploy the application AND create the system tenant based on initial parameters.
* This application uses an Install step from the client to create the system Tenant. The steps are listed below in Initial setup.

Otherwise the applications should work virtually the same.

### Initial setup

The first step we need to do is create the System User or System Admin Tenant.
- Navigate to  **`<StaticWebsiteURL>/install`**.
where StaticWebsiteURL is the URL of the website (see 'To Start Application' section above)
  - Fill in the form. You MUST provide a valid email address, because an email will be sent with a temporary password to login.
  - You should get a message saying the "Admin Registration Successful".
- Check your email for a message with your temporary password.
- Navigate to **`<StaticWebsiteURL>/login`**
  - Enter your email address and the temporary password. You will be promted to change the password.
  - Once completed, you have an admin tenant setup and can check status of services and view tenants.

### Register new tenant

Now you can create another user tenant.
- Click on the register link
- Fill out the form using a Different email than the Admin tenant.
- When you login with this new account, you will need to change the password.
- Then you can play with the products and orders part of the application.

## Cleanup

When you are done, you will need to cleanup and remove the resources so that you are not charged for any usage in your account. This is a 2 phase process.

##### DANGER: THIS WILL TEARDOWN YOUR AWS ENVIRONMENT - ONLY USE FOR NON-PRODUCTION OR NON-CRITICAL ENVIRONMENTS

### Phase 1 - Client Teardown

#### Tenant-specific data, roles, and userpools

1. Login with System tenant you created with the install process.
2. Go to Tenents in the top navbar. 
3. Click on the Delete icon to remove each tenant. This will remove each tenants data, associated roles, and userpools.
4. Finally delete the system Tenant using the same procedure.

#### Identity Client AWS Infrastructure

- From the the **`<code-dir>/app.envyforge.com.identity/client/`** directory run the following command: 

```
.../client > sls remove
```

### Phase 2 - Services Teardown

- Go the **`<code-dir>/app.envyforge.com.identity/service/readme.md`** document and follow the Cleanup Process.
