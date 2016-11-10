RubyGems SSL Fiasco
===

October 10, 2016

I maintain [jekyll-exe](https://github.com/altbdoor/jekyll-exe), which is basically Jekyll in an executable with OCRA. I do not really know how Ruby or Jekyll works in detail. After all, I only use it for simple static website stuff.

So a few days ago, I got to know that Jekyll is now v3.3.0, so I thought I'd try building it then. `gem install jekyll -v 3.3.0` and voila... Huh?

```
Unable to download data from https://rubygems.org/ - SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://api.rubygems.org/specs.4.8.gz)
```

Apparently, this is a common issue. RubyGems update their SSL certs from time to time, and users are expected to update their RubyGems. Now, this is all good practice, unless if you are stuck on Windows with RubyInstaller.

Oddly, I went to RubyGems' Google Group first instead of GitHub issues, but lucky for me, my answer was there. Someone had made a [similar complain](https://groups.google.com/forum/#!topic/rubygems-org/jcqpXXvOo0c) about this a few days ago, and was redirected to a documentation on [how to update the SSL certificates](http://guides.rubygems.org/ssl-certificate-update/).

So the "official" way to fix this is under the **Installing using update packages (NEW)** section, where you are supposed to download a RubyGems update for your version. Using `gem --version` reveals my local RubyGems version to be 2.2.5, and so I went to the [latest RubyGems 2.2.x GitHub release page](https://github.com/rubygems/rubygems/releases/tag/v2.2.5), but there was no `.gem` file. Wait, to begin with, how would updating 2.2.5 to 2.2.5 be an update?

This is rather frustrating. So I thought, forget it, my Ruby environment was only for building Jekyll anyways. I nuked it, and reinstalled Ruby 2.1 from [RubyInstaller](http://rubyinstaller.org/downloads/). If you are asking, why not take the latest Ruby version? Its because OCRA for building Jekyll does not support 2.2.x and above yet.

With Ruby 2.1.9 and the RubyDevKit all (re)setup, I tried to install Jekyll again, only to fail again. That's because RubyDevKit is still bundling RubyGems 2.2.5. With no where to turn to, I proceeded to the **Manual solution to SSL issue** section.

So I followed the steps closely, and tried to obtain the new `.pem` files. But, a wild [404](https://raw.githubusercontent.com/rubygems/rubygems/master/lib/rubygems/ssl_certs/AddTrustExternalCARoot-2048.pem) appeared.

\*sigh\*

What is up with the [master](https://github.com/rubygems/rubygems/tree/master/lib/rubygems/ssl_certs) now? There appears to be a little restructuring, and the `.pem` files are placed in their (I assume) respective folders. With that knowledge at hand, I proceeded to backup the existing local `.pem` files, and `curl`-ed the new certificates from master.

```sh
# this is Git Bash on Windows by the way
cd /c/Ruby21/lib/ruby/site_ruby/2.1.0/rubygems/ssl_certs/
mkdir backup_pem
mv ./*.pem backup_pem/
curl -OJL https://github.com/rubygems/rubygems/raw/master/lib/rubygems/ssl_certs/rubygems.org/AddTrustExternalCARoot.pem
curl -OJL https://github.com/rubygems/rubygems/raw/master/lib/rubygems/ssl_certs/index.rubygems.org/GlobalSignRootCA.pem
curl -OJL https://github.com/rubygems/rubygems/raw/master/lib/rubygems/ssl_certs/rubygems.global.ssl.fastly.net/DigiCertHighAssuranceEVRootCA.pem
```

I did not attempt to follow the new structure of the master branch, in fear that my old RubyGems would not understand. With that done, RubyGems is working once more. I was able to install Jekyll and its dependencies, and proceeded to build Jekyll v3.3.0. I also manage to run `gem update --system`, which updated my RubyGems to 2.6.7. I do not know if Ruby 2.1 is playing well with RubyGems 2.6.7, but for now I'll move my concern to the Appveyor CI.

I do wonder if I was supposed to update RubyGems to 2.6.7 directly then.

**Update:** RubyGems have updated their [guide](http://guides.rubygems.org/ssl-certificate-update/).

---

#### References

- https://groups.google.com/forum/#!topic/rubygems-org/jcqpXXvOo0c
- http://guides.rubygems.org/ssl-certificate-update/
- https://github.com/rubygems/rubygems/releases/tag/v2.2.5
