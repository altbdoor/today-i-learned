Fun with Google Drive API v3
===

February 14, 2016

I started with looking for a file storage solution for some small files. I ended up learning a chunk of [Google Drive API](https://developers.google.com/drive/v3/web/about-sdk), and here is me trying to document what I have done.

First, I learned that Google Docs file can be exported as a text file. An [article from labnol.org](http://www.labnol.org/internet/direct-links-for-google-drive/28356/) showed how by adding extra parameters to the request.

```
https://docs.google.com/document/d/FILE_ID/export?format=txt
```

After some rounds of confusing circles with the documentation, I found a [small guide](http://stackoverflow.com/questions/20966397/good-tutorial-on-google-drive-sdk-and-oauth-2) on getting comfortable with OAuth and Google Drive API. It also points me to the [OAuth Playground](https://developers.google.com/oauthplayground/), which is the bedrock for all subsequent API calls. Since we will be using OAuth to interact with Google Drive API, we need to create an application with [Google Developers API Manager](https://console.developers.google.com/apis/). In short, the steps involved are:

- Go to API Manager, Overview page
- Enable **Drive API**
- Go to Credentials, create an **OAuth client ID**
- Set it as a **Web Application**, and give it a personal name

I was stuck for a moment, trying to authorize my own Google account to the new application. Once again, a [StackOverflow question](http://stackoverflow.com/questions/19766912/how-do-i-authorise-an-app-web-or-installed-without-user-intervention-canonic) came to the rescue. The key point here is how the question is **not using a Service Account**, since I wanted it to be bound to my account. To quote:

> NB2. This technique works well if you want a web app which access your own (and only your own) Drive account, without bothering to write the authorization code which would only ever be run once.

So what was missing was:

- Set **Authorized redirect URIs** as `https://developers.google.com/oauthplayground`
- Go to OAuth Playground
- Set OAuth Playground to use my application's OAuth credentials
  - Click Settings
  - Set **Access Type as offline**
  - Check **Use your own OAuth credentials**
  - Use **Client ID** and **Client Secret** from my application in Google Developer API Manager
- Authorize the Drive API, or simply use `https://www.googleapis.com/auth/drive`
- **Exchange authorization code for tokens**
- Note down the **Refresh token** and keep it somewhere safe
- Time to get down with the API

From here on, the **List possible operations** is extremely helpful if you wish to explore more. Uploading a file into Google Drive would count towards the disk space quota, which is unfavorable for me. Then I remembered that Google Docs files do not count against the quota. From another [StackOverflow question](http://stackoverflow.com/questions/18291928/how-to-create-a-public-google-doc-through-the-drive-api-php-client), I discovered the right mime type for it, which is `application/vnd.google-apps.document`. So a request is built with the OAuth Playground, and the first step of success is made.

```
POST /drive/v3/files HTTP/1.1
Host: www.googleapis.com
Content-length: 100
Content-type: application/json
Authorization: Bearer ACCESS_TOKEN

{
  "action": "create",
  "name": "test doc",
  "mimeType": "application/vnd.google-apps.document"
}

HTTP/1.1 200 OK
...
Content-type: application/json; charset=UTF-8

{
  "mimeType": "application/vnd.google-apps.document", 
  "kind": "drive#file", 
  "id": "FILE_ID", 
  "name": "test doc"
}
```

A new Google Doc with the name should be created at the root of the Google Drive. That would be very cluttered in the long run, but [documentations](https://developers.google.com/drive/v3/reference/files/create#request-body) come to the rescue again with the `parents: ["FOLDER_ID"]` parameter. The `FOLDER_ID` is easily obtainable when you browse into a folder through Google Drive (E.g. `https://drive.google.com/drive/folders/FOLDER_ID`).

Alright, so getting organized is good, but it does not help that these created files are empty. Luckily, [somebody did ask for a JavaScript solution](http://stackoverflow.com/questions/10317638/inserting-file-to-google-drive-through-api/), but the answer was for v2, when it is now v3. The last answer provided a much needed solution, and after some time hammering the playground, another request is built. Do note that a **custom Content-Type is required** for this.

```
POST /upload/drive/v3/files?uploadType=multipart HTTP/1.1
Host: www.googleapis.com
Content-length: 315
Content-type: multipart/related; boundary="===============1839147391=="
Authorization: Bearer ACCESS_TOKEN

--===============1839147391==
Content-type: application/json

{
  "parents": ["FOLDER_ID"],
  "name": "FILE_NAME",
  "mimeType": "application/vnd.google-apps.document"
}
--===============1839147391==
Content-type: text/plain

the red brown fox jumps
over the lazy dog

--===============1839147391==--

HTTP/1.1 200 OK
...
Content-type: application/json; charset=UTF-8

{
  "mimeType": "application/vnd.google-apps.document", 
  "kind": "drive#file", 
  "id": "FILE_ID", 
  "name": "FILE_NAME"
}
```

A peek into the file, and voila, all set. Double check with the extraction URL mentioned earlier (`https://docs.google.com/document/d/FILE_ID/export?format=txt`), and the same text file should pop out again.

So the next thing that comes to mind is, if I am able to serialize a file into text (base64 image comes to my mind first), save it into Google Drive via API, I can retrieve it again! Does it mean I have infinity disk space? Not exactly, with Google's limitations.

> These are the documents, spreadsheets, and presentation sizes you can store in Google Drive.
>
> - Documents: Up to 1.02 million characters. **If you convert a text document to Google Docs format, it can be up to 50 MB**.
>
> &mdash; [Google Drive Support](https://support.google.com/drive/answer/37603?hl=en)

<!-- -->
> Queries
>
> - requests/day : 1,000,000,000
> - **requests/100seconds/user : 1,000**
>
> &mdash; Google Developers Console, Drive API Quotas

The main concern would be 50 MB, I suppose. Since the files in question are assumed to be small, I suppose its sufficient. Assuming its base64 encoded (which means an increase of about 33% in file size), 37 MB would be cutting it close, at a result of 49.21 MB. Perhaps 30 or 35 MB would be a safer bet.

That would cover the basics, with insertion and extraction. But, be reminded that the **access token** will expire, which is why we need the **refresh token** noted down. Open up Step 2 in OAuth Playground, and notice how an access token can be refreshed.

```
POST /oauth2/v3/token HTTP/1.1
Host: www.googleapis.com
Content-length: 208
content-type: application/x-www-form-urlencoded
user-agent: google-oauth-playground

client_secret=CLIENT_SECRET&grant_type=refresh_token&refresh_token=REFRESH_TOKEN&client_id=CLIENT_ID

HTTP/1.1 200 OK
...
Content-type: application/json; charset=UTF-8

{
  "access_token": "NEW_ACCESS_TOKEN", 
  "token_type": "Bearer", 
  "expires_in": 3600
}
```

If we need to list down the files within the said folder, the [documentation](https://developers.google.com/drive/v3/web/search-parameters) has it all covered. As a short sample, here is how I get all the files created within the folder, and delete a file. Do note that they are URI encoded, so `=` is `%3d` and all that jazz.

```
HTTP Method: GET
https://www.googleapis.com/drive/v3/files?q=
  %27FOLDER_ID%27+in+parents+and+
  trashed+%3d+false+and+
  name+%3d+%27FILE_NAME%27

HTTP Method: DELETE
https://www.googleapis.com/drive/v3/files/FILE_ID
```

And that's about it. Think I've spent a little too much time on this on a Valentine's Day.

---

#### References

- https://developers.google.com/drive/v3/web/about-sdk
- http://www.labnol.org/internet/direct-links-for-google-drive/28356/
- http://stackoverflow.com/questions/20966397/good-tutorial-on-google-drive-sdk-and-oauth-2
- http://stackoverflow.com/questions/19766912/how-do-i-authorise-an-app-web-or-installed-without-user-intervention-canonic
- http://stackoverflow.com/questions/18291928/how-to-create-a-public-google-doc-through-the-drive-api-php-client
- http://stackoverflow.com/questions/11952169/create-new-google-drive-file-with-text-content-javascript
- https://developers.google.com/drive/v3/reference/files/create#request-body
- http://stackoverflow.com/questions/10317638/inserting-file-to-google-drive-through-api/
- https://support.google.com/drive/answer/37603?hl=en
- https://developers.google.com/drive/v3/web/search-parameters
