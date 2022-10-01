# Google Cloud Functions to Cloud Services tutorial
This tutorial will show you how to connect Firebase Cloud Functions or Google Cloud Functions to Google Cloud Services such as Translate using IAM service accounts in the Google Cloud Console.

Credentials vs. Service Accounts
-------------------------------

There are two ways to hook up a Firebase Cloud Function to a Google Cloud Service. The old way was to download credentials that looked like this:

```js
{
    "type": "service_account",
    "project_id": "my-awesome-project",
    "private_key_id": "1234567890",
    "private_key": "-----BEGIN PRIVATE KEY-----\n-your-sister's-phone-number=\n-----END PRIVATE KEY-----\n",
    "client_email": "google-cloud-translate@my-awesome-project.iam.gserviceaccount.com",
    "client_id": "1234567890",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/google-cloud-translate%40my-awesome-project.iam.gserviceaccount.com"
}
```

There is a way to import these credentials into your Firebase Cloud Function. I haven't figured this out. Maybe you use the [Google Cloud Secret Manager](https://cloud.google.com/secret-manager).

The new way is to use service accounts. You don't have to deal with credentials. Instead you work in your [Google Cloud Console](https://console.cloud.google.com/).

IAMs and Service Accounts
-------------------------

The Google Cloud Console can be overwhelming at first. You start with a user account, which might be your gmail address. Then you make an organization, and in your organization you make a project.

Then look for `IAM & Admin`. `IAM` stands for *Identity Access Management*. IAMs look like email addresses. IAMs have *names* and *roles*. One of your IAMs should be your email address with your name and the role *Owner*.

Think of the other IAMs as your minions, who do your bidding. Some were automatically created when you signed up for Google Cloud Services.

You create new IAMs when you want to do stuff. For example, I wrote a Firebase Cloud Function that translates English into Spanish. I want this function to call Google Cloud Translate. I need a particular type of IAM to do this: a *service account*. A service account is an IAM designed to do one thing. The name of all service accounts is *Service Account* and their email addresses always end in `iam.gserviceaccount.com`.

Because a service account does something you assign a role to a service account. In this case, my service account calls Google Cloud Translate so I assign it the role `Cloud Translation API User`.

Because a service account does *only one thing* you assign a *limited role* to a service account. My translation service account doesn't get to do stuff in the Google Cloud Dataplex, for example. It also doesn't get to be a Cloud Translation API Editor or Admin. It's just a Cloud Translation API User.

Make a Service Account
----------------------

In your Google Cloud Console, under `IAM & Admin` look for `Service Accounts` in the left sidebar. In the top bar look for `+ CREATE SERVICE ACCOUNT`. 

You can name your service account anything you want. I suggest giving it the name of your Firebase Cloud Function. This makes it clear what the service account is for. To make extra clear write in the `Service account description` something like `This service account hooks up the Firebase Cloud Function SuperTranslator to Google Cloud Translate.` At some point in the future you'll have too many service accounts and you'll want to delete some.

Click `DONE`.

Assign a Role to a Service Account
----------------------------------

Now click `IAM` under `IAM & Admin` in the left sidebar. Your service account should be there, because services accounts are a special type of IAM.

On the right side click the pencil associated with your service account. Under `Assign roles` scroll through the hundreds of options to find the Google Cloud Product or Service that you want your service account to use. After selecting the product or service select a role.

As I noted earlier, assign the lowest role that enables your function to work. In the case of my translation function, there are four choices: Admin, Editor, User, and Reader. The lowly Reader minions are only allowed to read policies and org descriptions. That won't work. User minions get to access the Google Cloud service. That's what I want my service account to do. Editor gets to edit policies and org descriptions and Admins have full access to policies and org descriptions. I don't want my service account to do that stuff.

Back on your `IAM` page, you'll notice on the right that each IAM has a count of *excess permissions* (in blue). This might be a Admin role that is only asked to perform User level stuff. Clicking the blue triangle sometimes recommends that you do something to reduce excess permissions. To rephrase what I said earlier, the purpose of IAMs is to give each minion just enough authority to do its job, and no more.

Hook Up Your Service Account to Your Cloud Function
------------------------------------------------------------
Put your service account in your Cloud Function. Here's my code:

```js
const { TranslationServiceClient } = require('@google-cloud/translate');
const translationClient = new TranslationServiceClient('my-awesome-function@my-awesome-app.iam.gserviceaccount.com');
```

This instantiates a new Translation Service Client and hooks up my service account.

If you don't do this step, i.e., you do this:

```js
const { TranslationServiceClient } = require('@google-cloud/translate');
const translationClient = new TranslationServiceClient();
```

then the Google Cloud will fall back on the deployed service account, which is the next step. In other words, if you do the next step correctly you can skip this step.

Deploy Your Cloud Function
--------------------------

*Don't* deploy your Firebase Cloud Function like this:

```
firebase deploy --only functions:myAwesomeFunction --service-account my-awesome-function@my-awesome-project.iam.gserviceaccount.com

```

The Firebase CLI doesn't know about service accounts. Instead deploy Firebase Cloud Function using `gcloud`:

```
gcloud functions deploy myAwesomeFunction --service-account my-awesome-function@my-awesome-project.iam.gserviceaccount.com
```

I tried deploying without the service account:

```
gcloud functions deploy myAwesomeFunction
```

with and without my service account in `new TranslationServiceClient()`. My function executed perfectly. So either Google remembers which service account goes with each function, once you've deployed it correctly, or the above instructions are a waste of time. :-)

Test Your Function
------------------

Trigger your function and look at your logs and you should see your Firebase Cloud Function execute.
