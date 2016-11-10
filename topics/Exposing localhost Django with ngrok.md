Exposing localhost Django with ngrok
===

November 10, 2016

This is going to be a short one.

Firstly, [ngrok](https://ngrok.com/) is an amazing free service which allows you to expose your local server to the internet. As a simple example, I can expose my development Django server to a tester, and check the logs when an issue happens. (This would be implying that the existing workflow is pretty bad, but yeah.)

ngrok is [available for all platforms](https://ngrok.com/download), and all it takes is to unzip it to a path, boot a command prompt, and run it. It is recommended to register for an account, even though if the account is only on the Free tier. You will get an `authtoken`, which allows you to monitor extra stuff through their web interface.

Alright, so Django. I already have it running with `./manage.py runserver`, so by default it runs on port 8000. All it takes is to expose this through ngrok.

```sh
./ngrok http 8000
```

The command prompt should show a CLI interface, which also provides the current URL. This URL is accessible through the internet, so you can share it with anyone across the globe. Do note that the URL changes at every ngrok restart. Also, do note that ngrok has a [global infrastructure](https://ngrok.com/docs#global). Pick the closest available region, and pass it as an argument, or better yet, save it on the [YAML setting file](https://ngrok.com/docs#config). Here's an example.

```yaml
# path of the file should be ~/.ngrok2/ngrok.yml
authtoken: {your authtoken}
region: ap  # this is for asia pacific
```

In Django, if you are using `django.contrib.sites` in `INSTALLED_APPS`, Django will depend on the `HOST` header in order to serve the right website. On localhost, the header would have been `localhost:8000`. But through the ngrok URL, the header is the same as the given URL, which means Django will be unable to serve the right website.

Luckily, ngrok allows you to [change the `HOST` header](https://ngrok.com/faq#virtual-hosts) with the `-host-header` argument.

```sh
./ngrok http -host-header="localhost:8000" 8000
# of course, you can add -region here too
```

Finally, set Django to [allow the use of `X_FORWARDED_HOST`](https://docs.djangoproject.com/en/dev/ref/settings/#std:setting-USE_X_FORWARDED_HOST), and everything should be working. Share the ngrok URL and have fun.

---

#### References

- https://ngrok.com/
- https://ngrok.com/docs#global
- https://ngrok.com/docs#config
- https://ngrok.com/faq#virtual-hosts
- https://docs.djangoproject.com/en/dev/ref/settings/#std:setting-USE_X_FORWARDED_HOST
