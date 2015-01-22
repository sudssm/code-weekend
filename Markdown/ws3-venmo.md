Intro to APIs <a id="api-section"></a>
=============
Adding the internet to your app
-------------------------------

Topics to be covered:

- What APIs are, how they work, why we use them
- Sending HTTP requests from Node
- Authenticating with an API via OAuth
- Sending Venmo payments

[This zip file](assets/files/ws3.zip) contains what you should have at the end of this workshop. 
![OAuth](assets/img/ws3.jpg)

### Why we use APIs


### Preppin' app.js for our API
First open app.js. We need to set our Node app to try to reach venmo if someone using the app goes to localhost:3000/venmo. This is a route that takes us into venmo.js, and, importantly, relativizes the URL so that what here is /venmo/foo looks to venmo.js just like /foo.

	var venmo = require('./venmo');
	app.use('/venmo', venmo);

### Where all the venmo happens : venmo.js
In the same directory as your other .js files, write to venmo.js. We'll go through it line by line. First off, list dependencies, request, express, and router, which you've used before. We won't focus on those now, but to review, each dependency will be in this form: 
	var request = require('request');
	
We'll use the express router.
	var router = express.Router();

Now we get into config with Venmo. We need Venmo to know our requests are legit, so you'll need to be a venmo developer. You can do that [here.] (https://venmo.com/w/signup?from=oauth&client_id=1494&response_type=code&scope=access_feed,access_profile,access_email,access_phone,access_friends,make_payments,write_apps,access_webhooks,write_webhooks&state=/docs/authentication?) 

Once you've done that, make a new application, and it'll give you the iD and the secret. Make sure as well to set Web Redirect Url, under the application page, to "http://localhost:300/venmo/oauth" Whether or not you've gone through with that, let's go over the rest of the code.

First off after the five lines of Venmo config are three separate routes, for
        /
		/authorize
		/oauth
 
As you know, each of these routes is activated when the proper URL is requested by your app's adoring fans. As ay reminder, the fact that the function call is "router.get" does not mean you're "getting" something from the router (like Java's ArrayList.get(),) it means it's responding to a HTTP GET call from the browser.

Let's go through each of the routes. First up is the "root" or "/" venmo page. 
	First off, we initialize the variable venmo.
		venmo = {};
	This variable will hold the user's venmo information. Thus, the next thing we do is check to see if we have a cookie with the info. (Meaning the user has already logged in.)
	Regardless, it then renders "venmo.hjs", the actual HTML document that the user will see. We'll go over this in a bit.
		
Second up is "/authorize." The user clicks on a link from the root venmo page to come here, and its one line of code is why you had to tell venmo what your Web Redirect URL was. When the user hits this page, you hand them off to venmo (notice how you send them the authorizeUrl defined at the top of the file.) Venmo then sends them back to "/oauth", which is the path you define next!

The next route in our venmo app, "/oauth", is the second part of venmo's verification process. 
We already sent the user to Venmo so that Venmo could tell us they were all set. They called our /oauth page, and passed it req.query.code if the authorization went well. The first if statement under router.get('oauth') simply means that if the authorization failed, we don't continue with the sale.

In the event that the authorization went through, Venmo needs to verify us as a developer. This is the request module comes into play. We actually have to make a call directly to Venmo's website (using their API.) Notice that we're using a POST request, sending them both our unique clientId and clientSecret, and req.query.code, which venmo sent back to us. If this all checks out on Venmo's side, what we get back,
		req.session.venmo = JSON.parse(body)
is a session from venmo that's unique between your application and your client, so venmo knows that future payment requests are legit. If all goes well, it prints a notice to the user that they've been authenticated, and redirects them to your app's venmo hompage.

Okay, so, our last route in venmo.js, router.post('/send'), is where the magic happens. Handshakes have gone through, and if your user types in the amount, phone #, and note fields properly (checked by the first if statement), you're ready to do a payment.

This takes the form of another POST request to Venmo's API. You send them the relevant transfer information, as well as the unique user/client token that you generated in "/oauth." The following code defines how you handle Venmo's response.

		function(err, response, body) 
			if (err) {
				next(err);
			}

		var recipient = JSON.parse(body).data.payment.target.user;
		req.session.message = 'Sent $' + req.body.amount + ' to ' + req.body.phone + ' successfully';
		return res.redirect('/venmo');

Take a peek at this portion of the function here and in context in venmo.js. First, it checks to see if venmo returned an error. If not, it continues, and parses the Venmo response for user data. Finally, it prints a message to the user telling them that the payment was successful as it redirects them back to the root venmo page!

###OKAY but we're not done that.

So, especially if you learned HTML/CSS earlier in this file, you may be wondering, "what actually is determining what the user sees at page /vemno?" What a great question. They're seeing views/venmo.hjs!

So, you've seen .hjs before, but to reap, the biggest difference between .hjs and html is that you update certain parts of it dynamically based on req.session in node. So, if you remember in the last block of code,
	
		req.session.message = "yodelyodelyoda"

In venmo.hjs, when you see

		{{#message}}
			<h3>{{message}}</h3>
		{{/message}}

You're taking req.session.message and inserting it into {{message}} in venmo.hjs!

Take note that the field

{{#venmo}} means "if this field is present, show the following,"and the field

{{/venmo}} means "if this field is not present, show the following."


	

