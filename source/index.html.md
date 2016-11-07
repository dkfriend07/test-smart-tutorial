---
title: SMART on FHIR Starter APP

language_tabs:
  - code

search: true
---

# Introduction

This tutorial will walk you through creating an app in Cerners SMART on FHIR ecosystem.

After completing this tutorial you know know how to:

* Create a basic SMART on FHIR app.
* Self register an app with Cerner.
* Run an app in Cerners SMART on FHIR sandbox.

# Project Setup

Fork this tutorial from [smart-tutorial](https://github.com/parthivbhagat/smart-tutorial) using githubs ui.

Clone a copy down to your local machine.

> git command to clone your repo:

```bash
$ git clone https://github.com/<your-username>/smart-tutorial
```

Example-smart-app, located in the source folder, includes several notable files:

**fhir-client.min.js**

Located in the lib folder, this is a minified version of [fhir-client.js](https://github.com/smart-on-fhir/client-js) which is an open source library designed to assist with FHIR's OAUTH2 and resource transactions.

It provides apis to get authorization tokens, provide information about the user and patient record in context, and issue API calls to fetch FHIR resources. This tutorial will lead you through the basics of building a SMART app using the JavaScript client.

SMART JS client uses the open-source library [fhir.js](https://github.com/smart-on-fhir/fhir.js) to interface with SMART API servers. Upon successful initialization and negotiation of the SMART authorization sequence, the client will expose instances of the fhir.js client via the following handles :

* *smart.api*
  * Non-context aware API for executing operations on all authorized resources.
* *smart.patient.api*
  * Context aware API which automatically applies its operations to the patient in context. Only exposed when a patient is in context.

Additional documentation on fhir-client.js can be found [here](http://docs.smarthealthit.org/clients/javascript/).

<aside class="notice">
This tutorial is designed to have a minimal footprint so we made the decision to directly include the minified version of fhir-client.js for simplicity. For your production applications we'd recommend pulling in fhir-client.js using npm or some other package manager to easily keep your application up to date.
</aside>

**launch.html**

Launch.html is the first page Cerner SMART server will redirect to when the SMART app is launched from the EHR. Its purpose is to authorize the app with the appropriate FHIR server for the required scopes.

**index.html**

Index.html well be re-directed to following a successful authorization. This file is the entry point to the application.

The other content you see in the source folder is the site for this tutorial. We used [Slate](https://github.com/lord/slate).

#GitHub Pages

>index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv='X-UA-Compatible' content='IE=edge' />
    <meta http-equiv='Content-Type' content='text/html; charset=utf-8' />
    <title><your-username>Example-SMART-App</title>
    ...
```

> To re-deploy the GitHub Pages site commit your changes and run:

```bash
$ ./deploy.sh
```

>The SMART app will be available at:

```
https://<your-username>.github.io/smart-tutorial/example-smart-app/
```
>Health check

```
https://<your-username>.github.io/smart-tutorial/example-smart-app/health
```

For the purposes of this tutorial we will be hosting our SMART app through [GitHub Pages](https://help.github.com/articles/what-is-github-pages). GitHub Pages is a convenient way to host static or client rendered web sites.

Setting up GitHub pages is easy, so easy in fact that it's already done for you. GitHub pages works by hosting content from a gh-pages branch. Since you forked the tutorial the gh-pages branch is already created, however GitHub won't publish your site until you make a change to the gh-pages branch, so lets make a change. Modify the index.html page to include your GitHub user-name in the title commit and push the change to master.

Now that we have a change, lets redeploy the App. Because this tutorial is built from Slate, we have a handy built in script to deploy changes to the GitHub Pages branch. Go ahead run ./deploy.sh to deploy your SMART app.

Once the app has been redeployed go to ```https://<your-username>.github.io/smart-tutorial/example-smart-app/health``` to ensure your app is available.

<aside class="notice">
GitHub Pages sites have a limit of 10 builds per hour, so if your page isn't updating, this could be the reason.
</aside>

# Registration
Now that we have a deployed SMART app, lets register it to access Cerner's FHIR resources. We have created a self registration console to allow any developer to be able run a SMART app against our development environment. Navigate to our [code console](https://code.cerner.com/developer/smart-on-fhir/smart_apps), if you don't have a Cerner Care Account, go ahead and sign up for one (it's free!). Once logged into the console, click on the "+ Register New App" button in the top right toolbar and fill in the following details:

Field | Description
--------- | -----------
App Name | "My amazing SMART app". Any name will do.
SMART Launch URI | ```https://<your-username>.github.io/smart-tutorial/example-smart-app/launch.html```
Redirect URI | ```https://<your-username>.github.io/smart-tutorial/example-smart-app/index.html```
App Type | Provider
FHIR Spec | DSTU2 The latest spec version supported by Cerner.
Authorized | Yes. Authorized App will go through secured OAuth2 login.
Standard Scopes | These scopes are required to launch the SMART app.
User Scopes | None
Patient Scopes | Select the Patient and Observation scopes

Click "Register" to complete the process. This will create add the app to your account and create a client id for app authorization.

The new client-id will be displayed in a banner at the top of the page and can be viewed at any time by clicking on the application icon to view more details.

# App Launch

We have now created our own SMART app and registered that app with Cerner to access the FHIR resources. Before we continue on with the next steps, lets take a moment to talk about the flow of a SMART app launch.

The SMART app launch flow beings with the EHR. Through some method a user has indicated that they wish to launch a smart application. The EHR redirects to the SMART ```Launch URI``` that was registered above.

In this example ```Launch URI``` is launch.html. launch.html redirects to the FHIR authorization server which in-turn redirects to the ```Redirect URI```, index.html, upon a successful authentication.

Post authentication index.html exchanges the returned authorization token for an access token and is then able to request resources from the FHIR server. Lets take a deeper look at launch.html and get it ready for authentication. For more information about the SMART app launching vist the [SMART Health IT site](http://docs.smarthealthit.org/authorization/)

![alt text](ehr_launch_seq.png "High Level EHR APP Launch Flow")

# Request Authorization

> launch.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>Example-SMART-App</title>
  </head>
  Loading...
  <body>
    <script src='./lib/fhir-client.min.js'></script>
    <script>
      FHIR.oauth2.authorize({
        'client_id': 'df7c5a17-52dd-4c88-8a32-cdfb557ba758',
        'scope':  'patient/Patient.read patient/Observation.read launch online_access openid profile'
      });
    </script>
  </body>
</html>
```

> Make sure to replace CLIENT_ID with the client id provided in the email and redeploy your site.

The responsibility of launch.html is to redirect to the appropriate FHIR authorization server. As you can see in the code, fhir-client makes our job pretty easy. All we have to do is call ```FHIR.oauth2.authorize``` and supply the client_id generated by the code console during registration and the scopes we registered.

The client_id is found in the app details page that can be accessed by clicking on the application icon in the [code console](https://code.cerner.com/developer/smart-on-fhir/smart_apps). Copy the client_id into the authorize call, commit the changes back to your repo and redeploy your site.

For the purposed of this tutorial you don't need to modify the scopes, but if you want to add any more scopes, remember to update this list.

Below is some additional information about the scopes we've selected for our app.

Scope | Grants
--------- | -----------
patient/Patient.read | Permission to read Patient resource for the current patient.
patient/Observation.read | Permission to read Observation resource for the current patient.
openid, profile | Permission to retrieve information about the current logged-in user. Required for EHR launch.
launch | Permission to obtain launch context when app is launched from an EHR. Required for EHR launch.
launch/patient | When launching outside the EHR, ask for a patient to be selected at launch time. Required for EHR launch.
online_access | Request a refresh_token that can be used to obtain a new access token to replace an expired one, and that will be usable for as long as the end-user remains online. Required for EHR launch.

For our app we will use Patient.read, Observation.read.
We will always include launch, online_access, openid & profile scopes to our app.

<aside class="notice">
Cerner does not allow use of wildcards(*). So instead of patient/\*.read you will need specify a particular scope of resource you will be using. Something like patient/Patient.read, patient/Observation.read etc. For list of resources visit [http://fhir.cerner.com/](http://fhir.cerner.com/)
</aside>

So just what exactly is the ```FHIR.oauth2.authorize``` method doing?

Through a EHR launch, launch.html will be supplied with two query params ```iss``` and ```launch```

```iss``` is the EHR's FHIR end point and ```launch``` is an identifier that will be passed along to the authentication server.

```FHIR.oauth2.authorize``` queries the FHIR endpoint to find the URI for authorization.
It then simply redirects to that endpoint, filling out the required api which includes the supplied client id, scopes and the launch parameter passed in from the ehr. (There are a few more params that can be read about [here](http://docs.smarthealthit.org/authorization/)). Additionally the function generates an appropriate ```state``` parameter that will then be checked after redirecting to the index page.

Following the ```FHIR.oauth2.authorize```, the app will redirect to the authentication server, which, on a successful authentication, will redirect back to the ```Redirect URI```, in this case, index.html

<aside class="notice">
The oauth2 client id is an identifier, not a secret as such it does not need to be hidden. It's used in conjunction with the other information provided through app registration to launch your application. If another app has access to your oauth2 client id they will not be able to masquerade as your application.
</aside>

# Access Token Retrieval

>index.html

```html
...
<script src='./src/js/example-smart-app.js'></script>
<script src='./lib/fhir-client.min.js'></script>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
<script>
  extractData().then(
    //Display Patient Demographics and Observations if extractData was success
    function(p) {
      drawVisualization(p);
    },

    //Display 'Failed to call FHIR Service' if extractData failed
    function() {
      $('#errors').html('<p> Failed to call FHIR Service </p>');
    }
  );
</script>
...
```

> example-smart-app.js - extractData

```javascript
...
window.extractData = function() {
    var ret = $.Deferred();
    ...
    ...
    FHIR.oauth2.ready(onReady, onError);
    return ret.promise();
  };
```

Now that the app has successfully been authenticated, it's time to call a FHIR resource, but first we need to obtain an oauth2 access token. We have an authorization code that was passed as a query param to the redirect URI (index.html) by the authorization server. The authorization code is exchanged for a access token through post to the authorization server. Again fhir-client.js makes this easy for us.

The "index.html" file includes a script which calls into the ```extractData``` function in example-smart-app.js.

```extractData``` uses the ```FHIR.oauth2.ready()``` function to exchange the authorization code for the access token and store it in session storage for later use.

# Access FHIR Resource
> example-smart-app.js - onReady

```javascript
...
function onReady(smart)  {
  if (smart.hasOwnProperty('patient')) {
    var patient = smart.patient;
    var pt = patient.read();
    var obv = smart.patient.api.fetchAll({
                  type: 'Observation',
                  query: {
                    code: {
                      $or: ['http://loinc.org|8302-2', 'http://loinc.org|8462-4',
                            'http://loinc.org|8480-6', 'http://loinc.org|2085-9',
                            'http://loinc.org|2089-1', 'http://loinc.org|55284-4']
                          }
                         }
                });

    $.when(pt, obv).fail(onError);

    $.when(pt, obv).done(function(patient, obv) {
      var byCodes = smart.byCodes(obv, 'code');
      var gender = patient.gender;
      var dob = new Date(patient.birthDate);
      var day = dob.getDate();
      var monthIndex = dob.getMonth() + 1;
      var year = dob.getFullYear();

      var dobStr = monthIndex + '/' + day + '/' + year;
      var fname = '';
      var lname = '';

      if(typeof patient.name[0] !== 'undefined') {
        fname = patient.name[0].given.join(' ');
        lname = patient.name[0].family.join(' ');
      }

      var height = byCodes('8302-2');
      var systolicbp = getBloodPressureValue(byCodes('55284-4'),'8480-6');
      var diastolicbp = getBloodPressureValue(byCodes('55284-4'),'8462-4');
      var hdl = byCodes('2085-9');
      var ldl = byCodes('2089-1');

      var p = defaultPatient();
      p.birthdate = dobStr;
      p.gender = gender;
      p.fname = fname;
      p.lname = lname;
      p.age = parseInt(calculateAge(dob));

      if(typeof height[0] != 'undefined' && typeof height[0].valueQuantity.value != 'undefined' && typeof height[0].valueQuantity.unit != 'undefined') {
        p.height = height[0].valueQuantity.value + ' ' + height[0].valueQuantity.unit;
      }

      if(typeof systolicbp != 'undefined')  {
        p.systolicbp = systolicbp;
      }

      if(typeof diastolicbp != 'undefined') {
        p.diastolicbp = diastolicbp;
      }

      if(typeof hdl[0] != 'undefined' && typeof hdl[0].valueQuantity.value != 'undefined' && typeof hdl[0].valueQuantity.unit != 'undefined') {
        p.hdl = hdl[0].valueQuantity.value + ' ' + hdl[0].valueQuantity.unit;
      }

      if(typeof ldl[0] != 'undefined' && typeof ldl[0].valueQuantity.value != 'undefined' && typeof ldl[0].valueQuantity.unit != 'undefined') {
        p.ldl = ldl[0].valueQuantity.value + ' ' + ldl[0].valueQuantity.unit;
      }
      ret.resolve(p);
    });
  } else {
    onError();
  }
}
...
```
With access token in hand we're ready to request a FHIR resource and again, we will be using fhir-client.js.

For the purposes of this tutorial we'll be retrieving basic information about the patient and a couple of basic observations to display.

The fhir-client.js library defines several useful api's we can use to retrieve this information.

* ```smart.patient.read()```
  * This will return the context for the patient the app was launched for.
* ```smart.patient.api```
  * ```fetchAll()```
    * This will use the [fhir.js](https://github.com/FHIR/fhir.js) api to retrieve a complete set of resources for the patient in context.

Both of these functions will return a jQuery deferred object which we unpack on success.

Unpacking is fairly straight forward. We're taking the response from the patient and observation resources and placing it into a "patient" data structure.

The last function from fhir-client.js is the ```byCodes``` utility function that returns a function to search a given resource for specific codes returned from that response.

The fhir-client.js library defines several more api's that will come in handy while developing smart app, read about them [here](http://docs.smarthealthit.org/clients/javascript/).

# Displaying the Resource

>index.html

```html
...
<h2>SMART on FHIR Starter App</h2>
<div id='errors'>
</div>
<div id="loading">Loading...</div>
<div id='holder' >
  <h2>Patient Resource</h2>
  <table>
    <tr>
      <th>First Name:</th>
      <td id='fname'></td>
    </tr>
    <tr>
      <th>Last Name:</th>
      <td id='lname'></td>
    </tr>
    <tr>
      <th>Gender:</th>
      <td id='gender'></td>
    </tr>
    <tr>
      <th>Date of Birth:</th>
      <td id='birthdate'></td>
    </tr>
    <tr>
      <th>Age:</th>
      <td id='age'></td>
    </tr>
  </table>
  <h2>Observation Resource</h2>
  <table>
    <tr>
      <th>Height:</th>
      <td id='height'></td>
    </tr>
    <tr>
      <th>Systolic Blood Pressure:</th>
      <td id='systolicbp'></td>
    </tr>
    <tr>
      <th>Diastolic Blood Pressure:</th>
      <td id='diastolicbp'></td>
    </tr>
    <tr>
      <th>LDL:</th>
      <td id='ldl'></td>
    </tr>
    <tr>
      <th>HDL:</th>
      <td id='hdl'></td>
    </tr>
  </table>
</div>
...
```

>example-smart-app.js - drawVisualization

```javascript
...
window.drawVisualization = function(p) {
  $('#holder').show();
  $('#loading').hide();
  $('#fname').html(p.fname);
  $('#lname').html(p.lname);
  $('#gender').html(p.gender);
  $('#birthdate').html(p.birthdate);
  $('#age').html(p.age);
  $('#height').html(p.height);
  $('#systolicbp').html(p.systolicbp);
  $('#diastolicbp').html(p.diastolicbp);
  $('#ldl').html(p.ldl);
  $('#hdl').html(p.hdl);
};
...
```

The last remaining task for our application is displaying the resource information we've retrieved. We won't be doing anything fancy here. In "index.html" we define a table with several id place holders. On a success from ```extractData``` we'll call drawVisualization which will show the table div as well as filling out the relevant sections.

# Test your App
> To re-deploy the GitHub Pages site commit your changes and run:

```bash
$ ./deploy.sh
```

Now that we have a snazzy SMART app, it's time to test it. First, commit to master any changes and deploy the site again.

Next log back into the code console and click on the app you've registered (My amazing SMART app). At the top of the page you will see a millennium user-name and password. To launch your app through the code console click the "Begin Testing" button. The console will ask if the app your launching requires a patient in context. Our app requires a patient, so select yes and choose a patient. Finally, click launch and the console will redirect to your application.

#Next Steps
Though this tutorial we have:

* Created a basic SMART on FHIR app.
* Registered that app with Cerner.
* Run the app in Cerners SMART on FHIR sandbox.

We've created a very basic application that meets the base requirements of being a SMART app. This application would require a fair amount of polish before being ready to be deployed in a production environment. A couple of next steps you could look at are:

* Try calling another resource.
* Write unit tests for the application.
* Pull in fhir-client.js through a package manager like webpack.
* Style the site using Cerners open source UX library Terra.

We're excited to see what you'll build next!
