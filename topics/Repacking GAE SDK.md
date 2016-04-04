Repacking GAE SDK
===

April 4, 2016

I dabbled with [Google App Engine](https://cloud.google.com/appengine/) lately, and I find the size of the SDK to be too large for my poor internets. It stands at around 40 MB, and comes with a number of libraries which I do not use. Since I have access to a small Linux instance with better bandwidth, I devised a small script to repack the SDK.

```sh
#!/bin/sh

CURRENT_DIR=$( cd "$(dirname "${BASH_SOURCE}")" ; pwd -P )
cd "$CURRENT_DIR"

echo "Clearing old files"
rm -rf ./google_appengine

echo "Getting new files"
version=$(curl -s https://cloud.google.com/appengine/downloads | grep -oP "google_appengine_(.+?)\.zip" | head -n 1)
wget -q "https://storage.googleapis.com/appengine-sdks/featured/$version" -O gae.zip

echo "Unzipping"
unzip -q gae.zip
rm gae.zip

echo "Removing Django"
rm -rf ./google_appengine/lib/django-*

echo "Compressing"

if [ -x "$(command -v 7za)" ]; then
	rm -f gae.7z
	7za a -mx9 gae.7z ./google_appengine > /dev/null
else
	rm -f gae.tar.gz
	tar czf gae.tar.gz ./google_appengine
fi

echo "Done"
```

In short, the script fetches the latest SDK version from the [downloads page](https://cloud.google.com/appengine/downloads), and retrieves the SDK zip file. Since I do not use the Django library, I removed them. Then, either `7za` or `tar` is used to repackage the SDK. `7za` has significantly better compression rates, but since it might not be available on the system, `tar` is used as a fallback.

As a comparison, here's a short table with version 1.9.35.

| | `.zip` | Unzipped | `tar czf` | `7za a -mx9` |
| --- | --- | --- | --- | --- |
| Original | 39.0 MB | 178.0 MB | 27.0 MB | 9.1 MB |
| Django removed | | 38.0 MB | 7.3 MB | 4.7 MB |

From 39.0 MB down to 4.7 MB, its a significant change. `tar` is also a lot faster than `7za`, but I can wait a few extra seconds for better compression. It can also be seen that Django makes a large part of the SDK. If the project requires Django, it would be nice, but otherwise, its a bit of an overkill.
