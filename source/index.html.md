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

The other content you see in the source folder is the site for this tutorial. We use [Slate](https://github.com/lord/slate).

#GitHub Pages

>The SMART app will be available at:

```
https://<your-username>.github.io/smart-tutorial/example-smart-app/
```

For the purposes of this tutorial we will be hosting our SMART app through [GitHub Pages](https://help.github.com/articles/what-is-github-pages). GitHub Pages is a convenient way to host static or client rendered web sites.

Setting up GitHub pages is easy, so easy in fact that it's already done for you. GitHub pages works by hosting content from a gh-pages branch. Since you forked the tutorial the gh-pages branch is already created.

Because this tutorial is built from Slate, we have a handy built in script to deploy changes to the GitHub Pages branch.

> To re-deploy the GitHub Pages site commit your changes and run:

```bash
$ ./deploy.sh
```

<aside class="notice">
GitHub Pages sites have a limit of 10 builds per hour, so if your page isn't updating, this could be the reason.
</aside>

# Registering SMART App
Once we have the SMART App created per the Project Setup step, get the application hosted. Your application is now ready to be registered with Cerner. Go to the link [Developer Portal APP Registration](https://code.cerner.com/developer/smart-on-fhir/register), sign into your Cerner Care Account and fill up following details:

Field | Description
--------- | -----------
App Name | Any name for your APP you want to give
SMART Launch URI | URL to the launch.html file . Like https://username.github.io/smart-starter-app/launch.html
Redirect URI | Just put your base app url. Like https://username.github.io/smart-starter-app/index.html
App Type | Select Patient facing App.
FHIR Spec | Select DSTU2
Authorized | Select yes. Authorized App will go through secured OAuth2 login.
Standard Scopes | These are standard scopes that are required to launch SMART App.
User Scopes | Don't select anything here
Patient Scopes | Select Patient and Observation scopes

and click Register. This will send request to Cerner FHIR group for them to create client id for the app authorization.

After this you will receive an email stating what your Client ID and Launch URL is.

# App Launch Flow

![alt text](ehr_launch_seq.png "High Level EHR APP Launch Flow")

# Request Authorization

> launch.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>SMART on FHIR Starter App</title>
  </head>
  Loading...
  <body>
    <script src='./src/js/starter_app.js'></script>
    <script src='./lib/fhir-client.min.js'></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
    <script>
      FHIR.oauth2.authorize({
        'client_id': 'df7c5a17-52dd-4c88-8a32-cdfb557ba758',
        'scope':  'patient/Patient.read patient/Observation.read launch online_access openid profile'
      });
    </script>
  </body>
</html>
```

> Make sure to replace CLIENT_ID with the client id provided in the email.


In launch.html we are calling fhir-client.js to authorize the app and the scopes via OAUTH2 by calling FHIR.oauth2.authorize.

FHIR.oauth2.authorize will call the following URL with requested scopes *https://authorization.sandboxcerner.com/tenants/d075cf8b-3261-481d-97e5-ba6c48d3b41f/protocols/oauth2/profiles/smart-v1/personas/provider/authorize?client_id=df7c5a17-52dd-4c88-8a32-cdfb557ba758&response_type=code&scope=patient%2FPatient.read%20patient%2FObservation.read%20launch%20online_access-%20openid%20profile&redirect_uri=https%3A%2F%2Fparthivbhagat.github.io%2Fpb026393.github.io%2F&state=a0d52f29-8ff1-a186-8337-2a9018979f72&aud=https%3A%2F%2Ffhir.sandboxcernerpowerchart.com%2Fdstu2%2Fd075cf8b-3261-481d-97e5-ba6c48d3b41f&launch=d831018a-bd90-48b1-8c91-e6b346c7f1ea*

Before you are able to run any operations against the FHIR API using the JS client, you will need to initialize it first.

Based on the client_id, current EHR user, the EHR makes a decision to approve or deny access. This decision is communicated to the app by redirection to the app's registered redirect URL. So always make sure we replace the CLIENT_ID with client id provided after your APP gets registered.

When an EHR user launches your app, you get a “launch request” notification. Just ask for the permissions you need using OAuth scopes( like patient/Patient.read ) and once you’re authorized you’ll have an access token with the permissions you need – including access to clinical data and context like:

* which patient is in-context in the EHR
* which encounter is in-context in the EHR
* the physical location of the EHR user

Below are different scopes we can use.

Scope | Grants
--------- | -----------
patient/Patient.read | Permission to read Patient resource for the current patient
patient/Observation.read | Permission to read Observation resource for the current patient
openid, profile | Permission to retrieve information about the current logged-in user
launch | Permission to obtain launch context when app is launched from an EHR
launch/patient | When launching outside the EHR, ask for a patient to be selected at launch time
online_access | Request a refresh_token that can be used to obtain a new access token to replace an expired one, and that will be usable for as long as the end-user remains online.

For our APP we will use Patient.read, Observation.read.
We will always include launch, online_access, openid & profile scopes to our APP.

<aside class="notice">
Cerner does not allow use of wildcards(*). So instead of patient/\*.read you will need specify a particular scope of resource you will be using. Something like patient/Patient.read, patient/Observation.read etc. For list of resources visit [http://fhir.cerner.com/](http://fhir.cerner.com/)
</aside>

# Access FHIR Resource

> starter_app.js

```javascript
(function(window){
  window.extractData = function() {
    var ret = $.Deferred();

    function onError() {
      console.log('Loading error', arguments);
      ret.reject();
    }

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
                                'http://loinc.org|2089-1']
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
          var systolicbp = byCodes('8480-6');
          var diastolicbp = byCodes('8462-4');
          var hdl = byCodes('2085-9');
          var ldl = byCodes('2089-1');

          var p = defaultPatient();
          p.birthday = dobStr;
          p.gender = gender;
          p.fname = fname;
          p.lname = lname;
          p.age = parseInt(calculateAge(dob));

          if(typeof height[0] != 'undefined' && typeof height[0].valueQuantity.value != 'undefined' && typeof height[0].valueQuantity.unit != 'undefined') {
            p.height = height[0].valueQuantity.value + ' ' + height[0].valueQuantity.unit;
          }

          if(typeof systolicbp[0] != 'undefined' && typeof systolicbp[0].valueQuantity.value != 'undefined'&& typeof systolicbp[0].valueQuantity.unit != 'undefined')  {
            p.systolicbp = systolicbp[0].valueQuantity.value +
                                  ' ' + systolicbp[0].valueQuantity.unit;
          }

          if(typeof diastolicbp[0] != 'undefined' && typeof diastolicbp[0].valueQuantity.value != 'undefined' && typeof diastolicbp[0].valueQuantity.unit != 'undefined') {
            p.diastolicbp = diastolicbp[0].valueQuantity.value +
                                  ' ' + diastolicbp[0].valueQuantity.unit;
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

    FHIR.oauth2.ready(onReady, onError);
    return ret.promise();

  };

  function defaultPatient(){
    return {
      fname: {value: ''},
      lname: {value: ''},
      gender: {value: ''},
      birthday: {value: ''},
      age: {value: ''},
      height: {value: ''},
      systolicbp: {value: ''},
      diastolicbp: {value: ''},
      ldl: {value: ''},
      hdl: {value: ''},
    };
  }

  function isLeapYear(year) {
    return new Date(year, 1, 29).getMonth() === 1;
  }

  function calculateAge(date) {
    if (Object.prototype.toString.call(date) === '[object Date]' && !isNaN(date.getTime())) {
      var d = new Date(date), now = new Date();
      var years = now.getFullYear() - d.getFullYear();
      d.setFullYear(d.getFullYear() + years);
      if (d > now) {
        years--;
        d.setFullYear(d.getFullYear() - 1);
      }
      var days = (now.getTime() - d.getTime()) / (3600 * 24 * 1000);
      return years + days / (isLeapYear(now.getFullYear()) ? 366 : 365);
    }
    else {
      return undefined;
    }

  }

  window.drawVisualization = function(p) {
    $('#holder').show();
    $('#loading').hide();
    $('#fname').html(p.fname);
    $('#lname').html(p.lname);
    $('#gender').html(p.gender);
    $('#birthday').html(p.birthday);
    $('#age').html(p.age);
    $('#height').html(p.height);
    $('#systolicbp').html(p.systolicbp);
    $('#diastolicbp').html(p.diastolicbp);
    $('#ldl').html(p.ldl);
    $('#hdl').html(p.hdl);
  };

})(window);
```

Once the client is initialized, you can obtain the context in which it was launched (the user who authorized the client and, if applicable, the patient that has been selected) by using the following methods:

* *smart.user.read()*
* *smart.patient.read()*

Both of these return a jQuery Deferred object which you can register a success callback to process the returned FHIR resource.


Following operations available in [fhir.js](https://github.com/FHIR/fhir.js) are supported:

* *read* Read the current state of a given resource
* *search* Obtain a resource bundle matching specific search criteria
* *fetchAll* Retrieve the complete set of resources matching specific search criteria
and many others

Please see the fhir.js documentation for the complete list of available operations.

smart.patient.read() calls the FHIR server to get the Patient resource. The URL for the call looks like this *https://fhir.sandboxcernerpowerchart.com/dstu2/d075cf8b-3261-481d-97e5-ba6c48d3b41f/Patient/1316024*

smart.patient.api.fetchAll({type: 'Observation', query:''}) calls FHIR server to get Observation Resource. The URL looks like *https://fhir.sandboxcernerpowerchart.com/dstu2/d075cf8b-3261-481d-97e5-ba6c48d3b41f/Observation?code=http%3A%2F%2Floinc.org%7C8302-2%2Chttp%3A%2F%2Floinc.org%7C8462-4%2Chttp%3A%2F%2Floinc.org%7C8480-6%2Chttp%3A%2F%2Floinc.org%7C2085-9%2Chttp%3A%2F%2Floinc.org%7C2089-1%2Chttp%3A%2F%2Floinc.org%7C55284-4&patient=1316024*

# Displaying the Resource

>starter_app.js

```javascript
window.drawVisualization = function(p) {
    $('#holder').show();
    $('#loading').hide();
    $('#fname').html(p.fname);
    $('#lname').html(p.lname);
    $('#gender').html(p.gender);
    $('#birthday').html(p.birthday);
    $('#age').html(p.age);
    $('#height').html(p.height);
    $('#systolicbp').html(p.systolicbp);
    $('#diastolicbp').html(p.diastolicbp);
    $('#ldl').html(p.ldl);
    $('#hdl').html(p.hdl);
};
```
>index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv='X-UA-Compatible' content='IE=edge' />
    <meta http-equiv='Content-Type' content='text/html; charset=utf-8' />
    <title>SMART Starter App</title>

    <link rel='stylesheet' type='text/css' href='./src/css/starter_app.css'>
  </head>
  <body>
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
          <td id='birthday'></td>
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
    <script src='./src/js/starter_app.js'></script>
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
  </body>
</html>
```

We will put the display logic in draw_visualization function in starter_app.js file. In this file we are creating one div for error and one for results. For results div we have two tables. One table is for patient demographics and one for observations. In the script section we are calling extractData() from starter_app.js file. If the call is successful we will call drawVisualization() from starter_app.js which would make the div for holding results visible. If the call fails we will append text "Failed to call FHIR Service" to errors div.

# Test your App
Go to your developer portal and click on the App you have registered. Once you are in the App Details you will see client id, redirect url etc. for the App. There is also a Millennium username and password you can use to launch you App. At this time click Begin Testing. A screen will appear which will ask if you need a patient to launch this App. Say yes and select a patient. Once you do that click on Launch. At this time your App will open with the required demographics and observations for the selected patient.



