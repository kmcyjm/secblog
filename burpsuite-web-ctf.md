## Use BurpSuite to solve CTF challenge Secret Agent - Points: 200

>Here's a little website that hasn't fully been finished. But I heard google gets all your info anyway. http://2018shell.picoctf.com:53383

Firefox's developer tool doesn't work well in this case (haven't thoroughly tested yet), but the `Edit and Resend` feature seems doesn't do any help in this case, since before we can edit the `User-Agent` in the request, the request has already been sent to the server. Whereas in BurpSuite, it can pause the first request before it sent out, so that we can modify it before the server sees it.

### Steps

BurpSuite by default will start the proxy on `127.0.0.1:8080`. So configure your web browser to use it.(preferably Firefox, since it can set proxy on browser level, whereas Chrome use the system's proxy setting.)

- Navigate to `http://2018shell.picoctf.com:53383/`, and you might see Firefox instead trying to connect to `captiveportal.firefox.com`. To disable this feature, oen `about:config` in Firefox and set `network.captive-portal-service.enabled` to `false`.

- Here we go back to BurpSuite, and click `Drop` button, to drop the request sent to `captiveportal.firefox.com`, and we will see the next request is sending to `http://2018shell.picoctf.com:53383/`.

- Now we click `Forward`, the home page for this challenge will open up.

- Then we go ahead and click `Flag`, here is where the flag should appear if the `User-Agent` shows it is the Googlebot.

- Here, before clicking the `Forward` button, we need to go to the `Raw` pane below the `Forward` button, replace the string in  `User-Agent` with `Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)`. Basically we can pick any from the list of the ·full user agent string` in this [Google crawlers \(user agents\)](https://support.google.com/webmasters/answer/1061943?hl=en), and it will work.

Now after we hit `Forward`, the flag will be showing on the page.

![Happy](images/smileface.jpg?raw=true "Happy face")
