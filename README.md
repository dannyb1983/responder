
# Responder
## What does Responder do?

**Responder** listens to events as they occur on your cloud platform and then provides a simple way to determine how to respond to those events.
For example, **Responder** can "respond" to certain events by sending out notifications to your email or slack etc. (for a full list of the current integrations see: [Out-of-the-box Actions](#out-of-the-box-actions)).  **Responder** is also capable of addressing events by remediating them! For more examples and of **Responder**'s features check out [AWS Use Cases](#aws-use-cases) to see what **Responder** comes with right out of the box.


As of now, **Responder**'s functionality is centered around  AWS, but we plan to expand it's functionality to other cloud platforms in the future!

<p>&nbsp;</p>

# Set up
### Local AWS set up

1. Clone this repository locally to your machine.

2.  Make sure you must have the AWS CLI installed. For information check out [Installing, updating, and uninstalling the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html). 
  Once the CLI is installed, configure your CLI profile with admin privileges to your target AWS account. Check out [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
 
3.  You must have have an instance of AWS CloudTrail active on your AWS account in order for **Responder** to work. For info on setting up CloudTrail, check out [Creating a Trail for your AWS Account](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html)

4. Install the [Serverless Framework](https://github.com/serverless/serverless#readme) globally 
```   npm install -g serverless   ```

    Once Serverless is installed, make sure to [configure it with   your aws credentials](https://www.serverless.com/framework/docs/providers/aws/cli-reference/config-credentials/)
  
5.  Install all the dependencies
  ```
  npm install
  ```
<p>&nbsp;</p>



## AWS Presets/Use cases

By default the projects comes with a few handy preset flows that are presented below. These rules can be disabled or reconfigured easily.

  

-  *`Root User has Logged in`*<br />

This flow will detect any login that the Root accounts makes, it will send a Slack message to notify anyone or anyteam of such event

-  *`Suspicious log in hour`*<br />

An IAM has logged in outside of the specified work hours, it will send a Slack message to notify anyone or anyteam of such event

  

-  *`Suspicious log in from an unknown IP address`*<br />

An IAM has logged in outside of the specified IP addresses, it will send a Slack message to notify anyone or anyteam of such event

  
<p>&nbsp;</p>

# Important Terms 

  
To get started, it's important to get familiar with the terms `rule`, `action` and `flow` as well as the file [`actions.yml`](actions.yml).

- `rule` is a piece of logic that will capture or detect a specific event of interest. For instance, somebody has logged in, a resource has been created, etc.
-  `action` defines how to "respond" to a (rule) event; it can be sending a Slack message, waiting, sending emails, check something, remove an unauthorized user etc.
- `flow` is a combination of a rule, and a series of actions.

- The `actions.yml` is a the main file where all your flows are described.


  <p>&nbsp;</p>



# Usage
<p>&nbsp;</p>

### Configuring **Responder** to your needs
<p>&nbsp;</p>

Several `config-default.yml` files exist inside the actions/rules folders. The purpose of these files is to setup **Responder**'s default/basic behavior. These files are part of the base software so while they can be modified, they cannot be removed. If you are interested in customizing **Responder**'s behavior, the `config-default.yml` files are where you should start. A list of out-of-the-box actions can be found [here](#out-of-the-box-actions)

### Creating your own configuration
<p>&nbsp;</p>


example: *Slack* 

The module can be found under `actions/SendSlack`

Create a `config.yml` file and copy the `default-config.yml` content to it and configure your own Webhook URL
```yml
  webhookURL: "https://hooks.slack.com/your/unique/webhook"

```


<p>&nbsp;</p>

example: *Email*

The module can be found under `actions/SendEmail`


Create a `config.yml` file and copy the `default-config.yml` content to it and configure your own emails you want to send a nbotification to. You can assign multiple emails as per the example below

```
      Subscription:
        - Endpoint: "jane_doe@corporation.com"
          Protocol: "email"
        - Endpoint: "lucy_luke@corporation.com"
          Protocol: "email"

```
<p>&nbsp;</p>


## Testing your configuration
<p>&nbsp;</p>

You might want to test **Responder** locally to make sure all of your alerts are set up, [configured](#usage) , and arriving as expected. Local testing will save you from having to re-deploy **Responder** every single time you want to change/test a configuration.

To do this just cd to the main directory, and run the index.js file with node.

  ```

  node index.js

  ```

<p>&nbsp;</p>

# Deploy
Once you have set up your configurations, deploy them to your AWS account:

```
npm run deploy
```
This will:
- Create APIKey in SecretsManager in us-east-1 and its replicas in all enabled regions, this key will be used to secure the events that are sent from various regions to *Responder* (installed in us-east-1)
- Deploy the Serverless Cloud Responder application in us-east-1
- Deploy this EventBridge [CloudFormation template](aws-eventbridge.yml) into every region

<p>&nbsp;</p>

# Remove

```
npm run remove
```

This will:
- Remove the EventBridge CloudFormation stacks in all regions
- Remove the Serverless app
- Remove the APIKey from SecretsManager in every region
<p>&nbsp;</p>


## How it works:
<p>&nbsp;</p>

### On AWS

*The EventBridge Rules*<br />
On AWS, the installation creates a Rule on EventBridge. This rule capture CloudTrail events and send these events to a Target.
The Target is an API Connection that sends the event to the Cloud Responder via a HTTP request.
Because the EventBridge is region specific, it has to be done in every region.

*The Serverless app*<br />
The serverless app exposes a HTTP endpoint that will receive POST requests sent by the API Connections installed in all regions. The the KeyPOST requests are authenticated via an APIKey tha is shared between the API Connection and the Serverless HTTP endpoint (API Gateway). An Authorizer (Lambda function) will authenticate the request by checking the API Key. Once authenticated the detection/capture sequence can start.

### Anywhere

*Detection/Capture sequence*<br />
The capture sequence, is a rule that is going to be run again the incoming event. The rule checks if the event is of interest. The rule must return an array of findings if the the event is of interest. Returning one or more findings will start a flow sequence.

*The flow sequence*<br />
A flow sequence will start when a finding has been discoverd, it a series of actions that can be taken sequentially or in parallel.
<p>&nbsp;</p>


# Documentation

  [General documentation](./documentation/README.md#table-of-contents)
  <p>&nbsp;</p>


# Out-of-the-box Actions

- [Send Slack](actions/SendSlack/documentation.md)
- [Send Email](actions/SendEmail/documentation.md)
- [Send Discord](actions/SendDiscord/documentation.md#send-discord-action)
- [Wait](actions/Wait/documentation.md)


