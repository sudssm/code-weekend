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

So, you've seen .hjs before, but let's go over this file more or less line by line. The first field you see is

		<title>{{title}}<title>

The <title> tag determines, you guessed it, the title of the page, and the syntax {{variable}} means that the actual content is determined dynamically in your node app. The next variable, {{message}}, is determined in venmo.js by whether  So, if you remember in the last block of code,
	
		req.session.message = "Sent $' + req.body.amount + ' to ' + req.body.phone = ' successfully';

This sets the message that then is shown. So, in venmo.hjs, when you see

		{{#message}}
			<h3>{{message}}</h3>
		{{/message}}

You're taking req.session.message and inserting it into {{message}} in venmo.hjs! To be more accurate, the .hjs is first looking to see if the message field is present, and then printing out the contents (in this case, the <h3> element, if it is, in fact, present.

This becomes more important at the next "if statement",
		{{#venmo}}
			code
		{{/venmo}}

There's a lot of code between these tags, and none of it is shown unless the "venmo" field, denoting that the user has authenticated, is present. This is important, because not only does the venmo variable decide what would be shown if you happened to include {{venmo}} as part of the text, but it actually acts as an if statement for what should be shown to the user.

So, let's assume that the user has authenticated. In this case, it show the fields {{display_name}} and {{username}}, and most importantly, includes a <form>. Let's see the whole thing: 

		<form action='/venmo/send' method='post'>
			Pay $<input type='text' name='amount' placeholder='Amount in dollars'> to
			<input type='text' name='phone' placeholder='Phone Number'> for:<br/>
			<textarea name='note' rows='3' cols='80' placeholder='For...'></textarea>			<br/>
			<input type='submit' name='Send Payment'>
		</form>

The form action designates what type of call the the browser will make when the form is submitted. In this case, the form is for the user to actually make a payment, and we have a venmo.js route that we created to handle a POST request to /venmo/send, so we want the browser to send the request to that URL (action='/venmo/send'), with POST (method='post').

In this form, there are 3 input fields. The first is the total amount of money being transfered. While we will parse it as a decimal, its input type='text'. The 'name' field we use to tell our app what variable it is assigned to, so we can access it in venmo.js. The placeholder, as you can probably tell, determines what is shown when the user has not yet typed anything in.

The next form is, by most measures, the same. Make sure to note, however, that the field name is assigned the value 'phone', again so we can reference the input as a variable by that name in venmo.js. The third field is of type <textarea> instead of type <input>. The biggest difference between a textarea and an input to understand is that a textarea has better accounting for large input of paragraph-length. Note that we specify the field rows='3', and cols='80'. Thus, unlike an <input>, we can guess that this field might be a sentence or two but probably no longer, and tailor the size of hte textfield to that. Furthermore, it handles newlines well, so the user can happily use multiple paragraphs.

The final important piece of your form is 
		<input type='submit' name='Send Payment.'>
Note that the type='submit' field is not specifying a variable, but is a keyword in your form that your browser knows to interpret as a submission of the form. the name field determines what is shown on the button. Because this submit button is within the same form as the other input fields (see that it comes before the ending </form> tag), it will submit everything within that form.

Notice now the end tag {{/venmo}}. This is the end of our if statement. i.e., if the user hadn't authenticated yet, she would have seen none of what we just went over. Now, on the next lines, we see

		{{^venmo}}
			<p>You have not authorized yet. <a href='authorize>Click here</a> to authorize with Venmo.</p>
		{{/venmo}}

So, if {{#venmo}} is the beginning of an if statement, {{^venmo}} is the beginning of an "if not" statement. Thus, the first time your user sees a venmo page, since they will not yet have authenticated, this statement will return "true", and they'll be given the link to the "authorize" page, and be allowed to authorize. Very tricky, very cool. As always, make sure to end your if with a {{/venmo}}, just like you always remember to end your <strong> tags, right? Right. ... </strong>


