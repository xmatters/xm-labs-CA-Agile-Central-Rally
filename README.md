
---

# CA Agile Central (CAAC), previously known as Rally
CA Agile Central is an enterprise-class platform that’s purpose-built for scaling agile development practices. Provide a hub for teams to collaboratively plan, prioritize and track work on a synchronized cadence.

This xM Labs Integration is far from a complete integration, think more of it as a good starting point. It allows you to connect up a CAAC cloud instance to an xMatters On Demand account to inject events into xMatters based on triggers you set in CAAC, for example when a defect status gets set to Submitted i.e. a new defect has been created.

This is a full closed loop integration, also allowing you to respond via the xMatters notification and have that take effect within CAAC, for example move the defect from Submitted to Open.


# Pre-Requisites

* A CAAC account (https://www.ca.com/gb/products/ca-agile-central.html).  If testing you can sign up for a free 10 user account via the website. 
* An xMatters account.


# Files

* [CAAgileCentralRally.zip](CAAgileCentralRally.zip) - The comm plan (that has all the scripts and email format and such).
* [CAAC_Inbound_Integration.txt](CAAC_Inbound_Integration.txt) - The Integration Builder JS to setup the inbound integration into xMatters from CAAC, should you need it standalone to the Comm Plan.
* [CAAC_Outbound_Integration.txt](CAAC_Outbound_Integration.txt) - The Integration Builder JS to setup the outbound integration from xMatters back to CAAC, should you need it standalone to the Comm Plan.
* [CA_Agile_Central_was_Rally.mp4](Media/CA_Agile_Central_was_Rally.mp4) - Video example of the finished integration.


# Installation

## Basic flow

1. Get an CAAC API key (https://rally1.rallydev.com/login/accounts/index.html#/keys)
2. Import comm plan
3. Inbound integration, change path
4. Outbound integration, set CAAgile endpoint, set ZSESSIONID cookie
5. Create webhook (with trigger) in CAAC via API
6. Create a defect in CAAC
7. Receive your xM alert, respond 'Set to Open'
8. In CAAC, defect is moved from Submitted to Open and the notes section gets updated!  (Not sure if notes is the best place to put the update but it works as a demo).


For general information on the CAAC API look here https://rally1.rallydev.com/slm/doc/webservice/

For information on creating CAAC webhooks look here https://rally1.rallydev.com/notifications/docs/webhooks 


## The set up (xMatters and CA Agile Central)

1. Log into CAAC, then browse to https://rally1.rallydev.com/login/accounts/index.html#/keys and add an API key.  Copy the API key value, e.g. _G0909ODSU7DFG98HEHKJEWKRJH9FSIF
2. Load the attached Comm Plan into xMatters.  Review the Form (CA Agility Central - Defect) configuration:
	- You may want to change the message options (i.e. turn off voice in devices) and add recipients.  
	- Copy the form's Web Service URL.
3. Edit the xMatters inbound integration (CA Agile Insight - Rally):
	- Set the path to be the Web Service URL of the 'CA Agility Central - Defect' form above.
	- This is the complicated part... lines 43 to 56 parse out the parameters we want from the large and ugly incoming json packet.  You'll see some of the lines, such as line 45 for the title, have a UUID in them, I suspect they'll be different values for you.  If so you can look at the bottom of this page (https://rally1.rallydev.com/notifications/docs/webhooks) to see the section 'How to find an Attribute UUID'.  I found it easier (not knowing CAAC) to just set up the integration and from the xMatters activity stream look at the entire incoming json (in TextWrangler or similar) and work my way through them.  For example, if you search for Notes you get this section...

		"abb5af67-e70a-4a98-ad21-26f71bf4fe83":{"value":"These are some notes","type":"Text","name":"Notes","display_name":"Notes","ref":null},

		You then take the UUID and use that in line 49 etc.  Sorry, it's a bit painful but you'll get there.
	- Copy the webhook URL for the inbound integration
4. Edit the outbound integration:
	- Check that the URL for the CAAgile endpoint is set to https://rally1.rallydev.com
	- Change the ZSESSIONID value to be that of your CAAC API key from step 1
5. Create a webhook in CAAC, this has criteria which when matched send the info to a webhook (xMatters).  This was a bit tricky to work out as you can delete them through the web UI but have to craete them via the API.  Here's my example of creating one that fires when a new defect gets created (STATE = Submitted).
	- Get a REST client tool, I use the one from Wiztools, available at https://github.com/wiztools/rest-client/releases.  I tried using a browser based tool but that kept failing authentication.
	- Set the URL as https://rally1.rallydev.com/notifications/api/v2/webhook and the method as POST
	- Set content type as application/json
	- Set a cookie as ZSESSIONID=your_CAAC_API_key_from_step_1.  It is important that this is set as a cookie not a header value.
	- Set the body to be a string and use
	
	{
  "AppName":     "xMatters Integration",
  "AppUrl":      "xMatters",
  "Name":        "xMatters Integration",
  "TargetUrl":   "https://emeademo.eu1.xmatters.com/api/integration/1/functions/c7e1d2c0-xxxx-xxxx-ab7e-5eee0d86e676/triggers?apiKey=18573f21-xxxx-4f1d-xxxx-3183d9xxxxc8",
  "Expressions": [
    {
      "AttributeName": "State",
      "Operator":      "=",
      "Value":         "Submitted"
    }
  ]
}

	- Change the target URL to be the incoming webhook URL from the xMatters inbound integration.
	- You can change AppName, AppURL and Name to be whatever you want (see https://rally1.rallydev.com/notifications/docs/webhooks)
	- You can change the expression to be whatever you like, again see those webhook docs for examples and syntax. 


# Testing

To test the integration, create a new defect within CAAC.  If you correctly set everything up you should see the alert created in xMatters and the message sent out.  If you respond to the message as 'Set to Open' it should go back to CAAC and set the defect state to open (needs a screen refresh).


# Troubleshooting

Check the activity stream on the inbound and outbound integrations.
If you're still stuck, reach out to an xPert. 