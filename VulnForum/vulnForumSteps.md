1. Subdomain Discovery
	- `dnsrecon -d vulnforum.co.uk -D ~/wordlists/subdomains.txt -t brt`
		- *0 Records Found*
	- Searching on [crt.sh](https://crt.sh/?q=vulnforum.co.uk)
		- Just found `*.auth.vulnforum.co.uk` but is shows nothing when accessed from the web
2. Content discovery on [www.vulnforum.co.uk](http://www.vulnforum.co.uk)
	- Found some endpoints![[Pasted image 20220529000558.png]]
	- Tried `/settings` but it redirected me to `/login`
3. Through surfing I found the username of a user that commented on a forum, so maybe we can do password bruteforcing on the login page.
4. While bruteforcing, I noticed the `POST` request that was sent to login and it contained a `method` parameter that was set to `local` and this showed when I changed it to `remote`.![[Pasted image 20220529052010.png]] *Found Flag 1*
5. Now, investigating this *URL* that was sent back from the server. It shows `Invalid Domain` when I visit it so may be it is a remote authentication server. Tried password bruteforcing through the remote auth server but same results as the local.
6. We need to investigate this `Server Error`. I tried enumerating passwords to the user `toby` with this `method=remote` parameter but it also failed.
7. Tried to `dig` this domain `xczz3rvw.auth.vulnforum.co.uk` and it showed that it is a `CNAME` to `vulnauth.co.uk`![[Pasted image 20220529233148.png]]
8. Visited `http://vulnauth.co.uk/` and it this HAPPENED!![[Pasted image 20220529233553.png]]*Found Flag 2*
9. This site only accepts Auth Domains that have not been yet registered. And since our `technical_msg` says that `Server \"http:\/\/xczz3rvw.auth.vulnforum.co.uk\/auth\" responded with a 404 error`, this might mean that this domain is not registered. I tried to register it with a testing email I have `nerdoarrr@gmail.com` and a password of `123456`, in case they send it a verification email but they did not. And then I was redirected to a `/complete` page.![[Pasted image 20220529235814.png]]
10. I visited `http://xczz3rvw.auth.vulnforum.co.uk` and now it redirects me to a `/login` that shows content after it was showing an `Invalid Domain` response.![[Pasted image 20220530000103.png]]
11. Now I logged on with the credentials I provided and it shows this.![[Pasted image 20220530000218.png]]
12. Played with this interface a little and tried to create a new `admin` user with `0` as Remote UUID and a password of `12345`, then when I logged on to the main website with `admin:12345` and `method=remote`, it responded with a `technical_msg`![[Pasted image 20220530001559.png]] So this might mean that something may happen if we have the UUID of the admin on the system.
13. Remember, we have the UUID of the user `toby` linked with its name when someone clicks on it `http://www.vulnforum.co.uk/user/1ac9c036aaf12a755084dc6a326ed7f5`. So the idea may be we create a user `toby` with our password of choice, say `1234`, and this UUID `1ac9c036aaf12a755084dc6a326ed7f5`? Then we login with `toby:1234` and see what happens
14. Voila! As we expected. `Login Successful`!![[Pasted image 20220530002108.png]]
15. Took the `token` in `set-cookie` response header and put it in the browser, and then I logged on to `toby`'s account! *(ATO)*![[Pasted image 20220530002526.png]]*Found Flag 3*
16. Found this user `john` so I changed his password with the same previous way *(Step 13)* and logged on to his account. It showed that `john` is the ADMIN!![[Pasted image 20220530020403.png]]
17. I surfed through the website being logged on as `toby` and found the this around the comment function![[Pasted image 20220530023102.png]] and by visiting this plugin code on [GitHub](https://github.com/code-for-sites/bbcode_plugin), it says that by providing \[script\]alert(true)\[/script\], it will produce `<script>alert(true)</script>` to the client. So, may be it is an XSS?![[Pasted image 20220530024821.png]]No. It shows the comment as is!![[Pasted image 20220530025618.png]]
18. I looked in the code and found that it replaces `(", <, >)` with their HTML variants `(&quote, &lt, &gt)` so this will prevent XSS! But then tried again adding an `<img>` tag using BBCode by writing \[img\]https://fileinfo.com/img/ss/xl/jpg_44.png\[/img\], it worked!![[Pasted image 20220530042815.png]]Now this may have an XSS, or a CSRF!
19. According to *(Step 16)*, the password of user `john` (the admin) is changed but doesn't allow the login process when `method=remote` (shows `"technical_msg":"Admins must logon locally"`), and when `method=local` (shows `"display_msg":"Invalid Username or Password"`). But we have a local password change function that makes a user change his password locally, and when I am logged in as `toby` and used the hash of user `john` it did not work so it must be validating with the `token` cookie![[Pasted image 20220603140605.png]]So here comes the CSRF!
20. So I tried this payload `[img]http%3a//www.vulnforum.co.uk/settings/password%3fpassword%3d123456%26hash%3d76887c0378ba2b80f17422fb0c0791c4[/img]` in the comment box but it did not work because the code of the BBCode plugin used has this validation for the `[img]` tag![[Pasted image 20220603141456.png]]So the link must end with one of these formats `.jpg, .jpeg, .gif, .png`.
21. Waited for `john` to see it and tried to login locally and Voila!![[Pasted image 20220603141901.png]]
22. Logged in as `john` and saw the confidential section that has the last flag.![[Pasted image 20220603142028.png]]*Found Flag 6*