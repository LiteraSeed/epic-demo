SMART ON FHIR APPLICATION PROTOTYPE

This project is a functional prototype of a client-side SMART on FHIR application. It was built to demonstrate the core EHR launch workflow, authorize against a FHIR server using OAuth 2.0, and display basic patient and provider data. The application is built with plain HTML and client-side JavaScript, using the fhir-client.js library to manage the connection.


HOW IT WORKS: THE SMART EHR LAUNCH FLOW

The application follows the SMART on FHIR EHR Launch protocol.

1.The flow begins when a user inside an EHR system or a simulator launches the app. The EHR opens the app's URL and passes two key parameters: 'iss' and 'launch'.

2.The code detects these initial parameters and understands this is the start of an EHR launch. It uses the fhir-client.js library to immediately redirect the browser to the EHR's authorization endpoint. This request for authorization includes the specific permissions the app needs, its registered redirect_uri, and the 'launch' token it just received.

3.At the EHR's website, the user is authenticated. After logging in, the user is presented with a consent screen where they must explicitly authorize the app's request to access their data.

4.Once authorized, the EHR redirects the browser back to our application's redirect_uri. This time, the URL contains a temporary, one-time-use 'authorization_code' and a 'state' parameter for security. The application's code, again using fhir-client.js, detects this 'code'. In the background, it makes a secure POST request to the EHR's token endpoint, exchanging the code for a durable 'access_token'.

5.With this 'access_token', the app is finally authorized. It can now make secure API calls to the EHR's FHIR server to request the data it has permission to see, such as patient demographics and provider information. The fhir-client.js library manages attaching the access token to all subsequent FHIR API requests.

HOW FHIRCLIENT.JS WORKS


The fhirclient.js library simplifies this complex workflow by providing a few key functions. It's loaded into our app via a script tag from a CDN, creating a global 'FHIR' object.

Key functions from the library that our code uses:

FHIR.oauth2.authorize({ clientId, scope, iss, launch, redirect_uri }): This function starts the process. It takes configuration details and redirects the browser to the EHR's authorization server.

FHIR.oauth2.ready(): This function handles the return trip from the EHR. It automatically looks for the parameters in the URL, handles the secure token exchange, and if successful, provides a powerful 'client' object.

client: This object is the result of a successful authorization. It holds the access token and all the launch context. We use it to interact with the FHIR server.

client.patient.read(): This is a convenience method on the client object that uses the stored access token to fetch the FHIR Patient resource for the patient currently in context.

client.user.read(): This is another convenience method that uses the access token to fetch the FHIR resource for the user who authorized the app, typically a practitioner.

THE CODE'S WORKFLOW

The script waits for the HTML page to load, then runs a 'main' function.

First, it checks the URL to see what parameters are present.

If the 'iss' and 'launch' parameters are found and an authorization 'code' is not, the script knows this is the beginning of the launch. It calls FHIR.oauth2.authorize() to redirect the user to the EHR's login screen.

If the 'iss' and 'launch' parameters are not found, or if a 'code' parameter is found, the script knows it's on the return trip from the EHR. It calls FHIR.oauth2.ready(). This function attempts to exchange the 'code' for an 'access_token'.

If the token exchange is successful, the 'onReady' function is called. This function uses the 'client' object to make API calls like client.patient.read() and client.user.read() to fetch data. After the data is fetched, helper functions are used to update the HTML to display the patient and provider information.

If at any point FHIR.oauth2.ready() fails, the 'onError' function is called to display an error message.

HOW TO RUN AND TEST

TESTING WITH THE PUBLIC SMART APP LAUNCHER

First, go to the SMART App Launcher website at https://launch.smarthealthit.org/
Configure the launcher with the following settings:
Launch Type: Provider EHR Launch
FHIR Version: R4
App's Launch URL: https://literaseed.github.io/epic-demo/smart_app.html

Second, click the Launch button. This will simulate an EHR launch.

TESTING WITH THE EPIC SANDBOX

This requires additional setup and configuration changes.

First, download Epic Hyperspace

Second, obtain account and sign in

Third, go to integration setup. Turn test integration on. Click SMART on FHIR. For the url enter: https://literaseed.github.io/epic-demo/smart_app.html  For the client ID enter: ***REDACTED*** Launch type: select embedded. You can omit Launch context. 

Finally, to run it hit Patient chart, and from the dropdown select "Trigger OpenPatient"





