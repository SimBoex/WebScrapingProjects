in this setting the key libraries are the following: (...)

however there some possible problems:
  HTTPError: 403 Client Error: Forbidden for url 
  this is often returned when a Cloudflare-protected website recognizes your traffic as automated and denies access to the content.
  
  #how to solve this?
  one approach is to set a Fake User Agent
  
  basically when a non-browser web client (as my bot) has a unique user agent that is quite different from the one a browser web client is using.So i need to manipulate this, in order to hide my bot.

But what is first of all a User Agent?
it is a string that is sent by the browser to the server, is located in the HTTP header and gives the info about the browser type as well as the operating system used. Why? well the web servers uses to render the content in a way that's compatible with the user's specifications.

still this is not enough in general, since there are other headers that are actually send when using a browser. However in most of the cases it's enough.
Otherwise i need to optimize request headers.

Now some remarks, there is another element used for detecting bots and this is the amount of requests made by the same client, so we need to rotate the headers if we need a lot of requests

here a way to get the headers and the cookies to fake the identity!
https://stackoverflow.com/a/74187673

