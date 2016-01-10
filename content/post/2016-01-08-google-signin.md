+++
Description = ""
Tags = []
date = "2016-01-08T15:50:00+09:00"
title = "Google Sign-in"
+++

## Using Google Sign-in on Website X

**Disclaimer:**  [Read the docs](https://developers.google.com/identity/sign-in/web/sign-in) too.  This post doesn't cover everything.

A week ago I was completely clueless as to how Google sign-in works.  I set out to write about it and learned a few things.


### Overview

Using Google sign-in on a website requires first doing the following in the [Google developer's console](https://console.developers.google.com/home/dashboard):

* creating a project
* creating a sign-in client ID for that project
* associating the domain(s) of the website with the sign-in client ID

Sign-in is done using javascript on the web page to talk directly to Google's servers.  The javascript is loaded from Google's servers.  It is not necessary to involve the server for website X at all.

When Joe the Hacker attempts to sign in to website X, a popup dialog appears.  The contents of the dialog depend on Joe's current signed-in state.

If Joe is not signed in to Google at all, then a sign-in dialog appears.  If he's signed in to more than one account, then an account chooser dialog appears.  If he's signed into exactly one account, then the sign-in part is skipped.

If this is the first time he's attempted to sign in to website X, then he'll be asked to give permission for website X to have access to his profile information (name, picture) and email address.

In the case that Joe needs neither the sign-in dialog nor the permissions dialog (i.e. he's already signed in to exactly one account and is a returning user), then the pop-up closes itself immediately without any user action.

The browser remembers that Joe is signed in to website X using Google sign-in.  He can sign out of website X and still be signed into Google.  However, if he signs out of Google, then he'll automatically be signed out of website X as well.  He can't be signed in to website X using his Google ID and not also be signed in to Google.

If the webpage making the sign-in call is served from a domain that has not been registered in the console, then Joe will see a 400 error (redirect_uri_mismatch) and a picture of a broken robot when trying to sign in.  The error page also exposes the email address of the account that the project is made under.


### Javascript details

The file platform.js provides the global Google API object called `gapi` and the auth2 module.  The auth2 module must be explicitly loaded into gapi with the `gapi.load` method before it's used.  This method provides an optional callback for when/if the module is loaded successfully.

    gapi.load("auth2", callback);

Once the module is loaded, it must be initialized with the sign-in client ID (see above).  The client ID may either be provided as [an option to the init method](https://developers.google.com/identity/sign-in/web/reference#gapiauth2initwzxhzdk20paramswzxhzdk21) or in [a meta tag in the document](https://developers.google.com/identity/sign-in/web/sign-in#specify_your_apps_client_id).  The init function returns a GoogleAuth object.

    gauth = gapi.auth2.init(options);

A logical initialization flow would be to have the initialization in the load callback.

    gapi.load("auth2", function() { gapi.auth2.init(); });

The GoogleAuth object may also be obtained any time after it's initialized using the [`getAuthInstance`](https://developers.google.com/identity/sign-in/web/reference#gapiauth2getauthinstance) method.

    gauth = gapi.auth2.getAuthInstance();

At this point we can find out whether Joe is or isn't signed in to website X.

    if (gauth.isSignedIn.get()) { alert("User is signed in, and I debug with alerts."); }

If he isn't signed in, we can try to sign him in.  This is done with `GoogleAuth.signIn`.  In most cases, we should wait for him to click a button (e.g. "Sign In with Google") before doing so.  There are [a few options](https://developers.google.com/identity/sign-in/web/reference#googleauthsigninwzxhzdk72optionswzxhzdk73) that we can ignore for now.

    gauth.SignIn(options)

This function returns [a Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), making it easy to do stuff when it finishes.

    gauth.SignIn().then(function() {
        alert("We're in!");
    }, function() {
        alert("You FAILED!");
    });

There is also this thing called a GoogleUser object, which we can get after Joe has signed in.  (We can also get it before he's signed in, but it would be useless.)  The GoogleUser object reveals Joe's name, email address and profile picture if he has one.

    user = gauth.currentUser.get();
    profile = user.getBasicProfile();
    alert(profile.getName() + " " + profile.getEmail());

Finally, use can use `GoogleAuth.signOut` to sign Joe out of website X.

    gauth.signOut();

Although this signs him out, it doesn't forget about him.  The GoogleUser object is still available and still has his ID.

    user = gauth.currentUser.get();
    alert(user.getId());
    alert(user.isSignedIn());    // false

And we can sign him in again using the grant method.  This behaves slightly differently from `GoogleAuth.signIn` in that it doesn't give Joe a chance to choose a different account.

    user.grant();

According to the documentation, `GoogleUser.signIn` is equivalent to `GoogleUser.grant`, but my Firebug tells me that there's in fact no method called signIn on the GoogleUser object.  Bad documentation.


### What usually happens in practice

Google provides [a handy shortcut](https://developers.google.com/identity/sign-in/web/sign-in#add_a_google_sign-in_button) so that anyone can add Google sign-in to their page without knowing any javascript.  As soon as platform.js is loaded, it checks for the existence of a `div.g-signin2` (&lt;div class="g-signin2"&gt;) and springs into action if found.

It loads the auth2 module and puts a standard sign-in button in the div (clobbering what was there, watch out), and wires up the button so that when you press it, it logs you in.  It can also call a function of your choosing on sign-in, passing in Joe's GoogleUser object.

The button appears to be rendered with [`gapi.signin2.render`](https://developers.google.com/identity/sign-in/web/reference#gapisignin2renderwzxhzdk114idwzxhzdk115_wzxhzdk116optionswzxhzdk117), and specifying options like this works as expected:

    <div class="g-signin2" data-onsuccess="onSignIn" data-longtitle="true" data-width="200"></div>

One caution here is that data-longtitle is a string value that gets cast to a boolean, and, as such, the string "false" will get cast to true.  The way to make longtitle false is set data-longtitle=&quot;&quot; (or to omit it entirely, as false is the default).


### Involving the server for website X

Chances are website X will want to know about Joe's sign-in on the server side.  This will require some javascript to send a token to the server, which the server can then decode to get Joe's information.

The token can be obtained from the GoogleUser object.

    token = user.getAuthResponse().id_token

The token is signed to prevent spoofing, and it's up to the server code to [verify its integrity](https://developers.google.com/identity/sign-in/web/backend-auth#verify-the-integrity-of-the-id-token).


### Scopes

Scopes are the permissions that we're asking Joe to give to website X.  The default for sign-in is scopes profile and email, and we get Joe's name, email, and picture if available.  _(Nerdy detail: the name and picture are not part of any scope; see below.)_

With the default settings, Joe will see the a request for the following permissions the first time he attempts to sign in to website X:

1. Know who you are on Google
2. View your email address
3. View your basic profile info

If other permissions (e.g. calendar) are needed, they can be added with the `scope` option in GoogleAuth.init, the `scope` option in GoogleAuth.signIn, or the `data-scope` tag on div.g-signin2.  Most scopes are expressed as urls, e.g. &quot;https:&#47;&#47;www.googleapis.com/auth/calendar.readonly".

Here is [a list of all known scopes](https://developers.google.com/identity/protocols/googlescopes).


### Fewer Scopes

The default scopes can be turned off with by sending `fetch_basic_profile: false` as one of the options to GoogleAuth.init.  (Note that this precludes using the auto magic button.)  In this case, at least one scope must be explicitly specified with the scope parameter.

The two scopes that are included by default are `profile` and `email`.  Adding the `profile` scope does nothing.  It only provides Joe's 21-digit ID, which we get anyway.  Adding the `email` scope gives us Joe's email address, but the only way to get it is to decode `id_token`, as `GoogleUser.getBasicProfile` only works when `fetch_basic_profile` is true.

Strangely, requesting only the `email` scope causes permission requests #1 and #2 to be displayed, even though we only get the email access.  Requesting only the `profile` scope causes only #3 to be displayed.  Requesting both `email` and `profile` displays #2 and #3 but not #1.  I have yet to understand the logic behind this behavior.

Also strange is the fact that neither the `profile` nor `email` scope provides Joe's name and picture.  As far as I can tell, the only way to get his name and picture is to stick to the default `fetch_basic_profile: true`.
