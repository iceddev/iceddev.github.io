At Iced Dev, our main business is consulting for Node.js and building
great web applications using it with HTML5 technologies. Our favorite
hosting platform is [Nodester](http://nodester.com "Nodester") ([@Nodester](http://twitter.com/nodester "Nodester")) because it is the only
Open-Source PaaS for Node.js and maintained by a great group of
developers. Recently, Nodester
redesigned their website and made it extremely easy to use their API
using [apigee’s](http://apigee.com "apigee") API Explorer.  This is very helpful for people using
Windows because curl is no longer required to request an invite, but
Window’s users may still have trouble using Nodester because Git is used
to push your project to the Nodester platform. For our Node.js
development, we typically use [Cloud9 IDE](http://c9.io "Cloud9 IDE") because it is very easy to
debug our Node.js apps before we deploy them to production.
 Unfortunately, Nodester is not a platform that is supported by Cloud9’s
deploy feature (We have no idea why this is!!! Cloud9, please make this
happen!). Thankfully, Cloud9 uses Git and makes all the commands
available to the user.  Using the new Nodester API Explorer and Cloud9’s
Git implementation, I am going to walk you through publishing your first
Node.js app to Nodester from ~~any~~ <span>**every**</span> operating
system with a web browser.


## Set up your Cloud9 account.

1.  Go to [http://c9.io/](http://c9.io)
2.  Choose a username
3.  Enter a valid email (they send you an activation email)
4.  Re-enter your email for confirmation
5.  Click the “Sign Me Up” button
6.  Check your email for your confirmation email
7.  Follow the activation link in your email
8.  Choose your password (following any password requirements C9 has)
9.  Re-type the password for confirmation
10. Click Activate
11. You will be redirected to your Cloud9 Dashboard (Keep this tab open for now)

## Request a Nodester Registration Coupon.

1.  Open (in a new tab) [http://nodester.com/](http://nodester.com)
2.  Once on Nodester.com, Request your Registration Coupon by clicking
    the blue button labeled “Request Registration Coupon”
3.  Enter your email address in the box labeled “Email” and click Send
4.  Wait patiently to receive your Nodester Coupon (many more are being
    sent out daily)
5.  Once you receive your Nodester Coupon, you will be able to sign up
    for an account at [http://nodester.com/api.html\#explorer](http://nodester.com/api.html\#explorer "Nodester API Explorer")

## Set up your Nodester account

1.  Expand the “User” option
2.  Click “CreateUser”
3.  The API Explorer will populate the URL
4.  To the right of the URL input, click the little gear button
    (![Apigee Gear Icon](https://apigee.com/images/gear_icon.png "Gear Icon"))
5.  Clicking the button will bring up the Advanced Options dialogue box
6.  Enter the Nodester Coupon received in the email in the “value” box
    next to “coupon”
7.  Enter your email in the “value” box next to “email”
8.  Enter a password in the “value” box next to “password”
9.  **VERY IMPORTANT:**The RSA Key you enter must be the RSA Key
    provided by Cloud9 IDE
    -   On the Cloud9 Dashboard, click “Show Your SSH Key” on the right
        side, in the “Account Settings” list
    -   Click “Copy” to copy your RSA Key to the clipboard
    -   Paste that RSA Key into the Nodester “Advanced Options” dialogue
        in the “value” box next to “rsakey”

10. Enter a username in the “value” box next to “user”
11. Leave all the other options as defaults
12. Click “OK” to save the options and dismiss the dialogue
13. Click the “POST” button to submit your API call
    -   Your Response should look something like:

            HTTP/1.1 200 OK
            Content-Length:
            20
            Content-Type:
            application/json; charset=utf-8
            Connection:
            close
            X-Powered-By:
            Express{
            "status": "success"
            }

    -   If your response does not have the “status” of “success”, you
        have to repeat the above steps again (most likely because
        something was mistyped, that username is taken, or something
        else funky happened).

14. Now that you have a Nodester account, it’s time to create your first
    Nodester app!!!!

## Create a Nodester App

There are 2 options for creating a Nodester app with the API, I will
walk you through the Nodester Admin Panel (The option I like better) and
then I will walk you through using the Nodester API Explorer.

### Nodester Admin Panel

1.  Since you have a Nodester account, you can access the Admin Panel by
    entering your username and password at the top of the page.
    -   As a new user, your will get an error message on the first page
        you see in the Admin Panel, but that is only because it is
        looking for your apps and you don’t have any yet.

2.  Click “Create New App” in the menu on the left
3.  Enter the name of the app you wish to create (This is also the
    subdomain that your app will be accessed with, e.g.
    appname.nodester.com)
4.  Enter the file name that you wish to be your startup file (This file
    will most likely be where your Node.js app is configured to listen
    on your specific port)
    -   **Personal preference:**I like to use `server.js` as my start
        file because I have encountered issues using `app.js` as the
        start file with Nodester.

5.  Once your Nodester app is created, you will see something similar
    to: [Nodester Admin Panel Clipping](![Nodester Admin Panel Clipping](http://iceddev.com/wp-content/uploads/2011/12/nodester-admin-panel.jpg))
    -   The port is the port that your app listens on (this will need to
        be hardcoded into your app) and the app-status is false because
        there is no code inside the app yet.

### Nodester API Explorer
1.  If you would rather use the API Explorer, the steps are below:
2.  Expand the “App” option
3.  Click “CreateApp”
4.  On the left side of the URL input box, click the lock icon (![Apigee
    Padlock Icon](https://apigee.com/images/padlock_icon.png
    "Padlock Icon")) and select “Basic HTTP”
5.  Enter your Nodester username and password in the dialogue that opens
6.  Open the “Advanced Options” dialogue
7.  Fill in the values for “appname” and “start”
8.  Click “OK”
9.  Click “POST”
10. The response you should get looks like:

        HTTP/1.1 200 OK
        Content-Length:
        182
        Content-Type:
        application/json
        Connection:
        close
        X-Powered-By:
        Express

        {
          "status": "success",
          "port": 13092,
          "gitrepo": "git@nodester.com:/node/git/nodester-deploy/4939-840ce8a201f03c7d27852a5bde15b55e.git",
          "start": "server.js",
          "running": false,
          "pid": "unknown"
        }

11. Let’s get this app into Cloud9, so we can add some code and get it
    running!

## Import Nodester App into Cloud9

1.  Click “Info” and copy the “gitrepo” value from the dialogue box that
    opens
2.  In your Cloud9 Dashboard, click the plus button that it points to
    while saying “Create a Project Here”
3.  Choose the “Clone from url” option and paste your “gitrepo” value
    from Nodester into the box, then click the “Checkout” button
4.  Cloud9 will perform the git clone operation and (once finished) you
    will be able to open the project within Cloud9 by selecting the
    project and clicking “Start Editing”
5.  Now that you have your project cloned into Cloud9, let’s add some
    code!

## Add “Hello World” Code

1.  When your project opens, we need to create the start file specified
    when creating your Nodester app (I used `server.js`) by clicking the
    plus icon to open an untitled document.
2.  For this tutorial, we will just get this app running the Nodejs.org
    “Hello World” code with some small tweaks to get it to run on
    Nodester and still debug on Cloud9
    -   Cloud9 requires you to use `process.env.C9_PORT`as your port for
        Node’s `.listen()`method and Nodester requires you to use a port
        specified to your app (They will have this environmental
        variable working soon), so I use this modified code for my
        “Hello World” app:

            var port = process.env.C9_PORT || 13087; /* Change 13087 to the port specified for your app */

            var http = require('http');
            http.createServer(function (req, res) {
              res.writeHead(200, {'Content-Type': 'text/plain'});
              res.end('Hello World\n');
            }).listen(port);

3.  Now, you want to save the code in the untitled document as the
    filename you specified as your start file when creating your
    Nodester app.
4.  Once saved, clicking the “Run” button will bring up a dialogue box
    for a run configuration, where you can name your configuration and
    specify which file it is to run as the start file (server.js for
    me).  You can then click “Run” on that dialogue box.
5.  In the “Output” tab of your Cloud9 console, you will see a tip with
    a clickable link, when click, will launch the webpage for your app
    and you should see “Hello World” printed to the screen.
6.  Now that you have your app working in Cloud9, let’s get it onto
    Nodester!

## Pushing your App to Nodester

1.  Pushing your app to Nodester requires a little knowledge of Git
    (even though you don’t install it on your local machine, Cloud9 is
    still using it behind the scenes).
2.  In the Cloud9 Command Line (at the bottom of the screen), we around
    going to want to issue the command (either type or copy-paste):

        git add .

    to add all files to the Git repo.

3.  Next, we want to issue the command:

        git commit -m 'initial commit'

    to make our initial commit to the Git repo.

4.  The last command we need to issue is

        git push origin master

    to push our changes to the Nodester Git repo.

5.  Nodester will report on the progress of the push and the restarting
    of your app with the committed changes
6.  If everything went as expected, your app will now be online!

## Test Your Nodester App

1.  Direct your web browser to YOUR\_APPNAME.nodester.com (my test app
    was c9deploy1.nodester.com) to make sure your app is running.
2.  You can stop and start your app from the Nodester Admin Panel if
    something isn’t working correctly.
3.  After all of this is setup/configured, the only 3 commands you need
    to remember are the Git commands to add, commit, and push your
    changes to your Nodester apps!!

## That’s All Folks!

I hope this post helps some people (especially ones developing on
Windows or ChromeOS) with the difficult task of getting your apps hosted
on the Nodester platform using the equally awesome Cloud9 IDE.