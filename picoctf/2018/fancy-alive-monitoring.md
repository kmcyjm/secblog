## fancy-alive-monitoring - Points: 400
>One of my school mate developed an alive monitoring tool. Can you get a flag from http://2018shell.picoctf.com:8587?

Note.
>The CTF server somehow blocked the outbound connection to non-standard ports. So on our own server, we need to listen on port 80, `nc -lvp 80`.

The main purpose of this blog post is to demonstrate how to get a webshell from a web server.

A few facts about Webshell,

- Webshell, as its name indicates, is to obtain a shell by means of a website.

  This diffs from how shell is usually accessed, which is either remotely via SSH, or log in directly in a server's console.

- A shell can be either a Linux shell, like Bash/Zsh, etc., or a command prompt from Windows.


Of course, above is my understanding of what a Webshell is, and what I believe the term "Webshell' should represent, not authoritative, but should be close.

A Webshell can be utilized to serve different purposes. It can be used to gain complete control of a web server; to search for sensetive information on the server; or to be used as a outpost to gather further information that can help to infiltrate other parts of the infrastructure that share the same network as the web server, etc. etc.

Ok, let us back to our main business.

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


