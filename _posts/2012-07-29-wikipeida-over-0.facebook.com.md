---
layout: default
---

While I was abroad and used a DNS tunnel to gain internet access an idea came
into my mind.

In Germany and some other countries you have completely free access to
*0.facebook.com* so why don't tunnel your traffic through this.
If this works you never have to pay for mobile internet access again.

Back at home  my first idea was to use the Linux TUN devices and tunnel to whole
TCP/IP traffic over private messages.
Like VPN over Facebook.
But after some testing and pinging through the tunnel it turns out that this
way of communication is very unstable and a lot of packages get lost.
This is because if you visit *0.facebook.com* only the last seven messages would
be available at the first page.
But the TCP handshake alone consumes already three messages.
So this was not the way of doing it.

The second attempt was to build a HTTP proxy so the request would be one message
and the response would be an other.
This decreased the amount of messages needed but it has an other disadvantage.
Everytime the client has to load all seven messages and this is very slow with
a standard mobile connection.

After this I thought that simply the private messages are the wrong medium.
I decided to uses notes instead and I am also not trying to tunnel the whole
traffic anymore.
Instead users could comment on the note and the server would do what the comment
says.
I thought Wikipedia would be the best website to try a proof of concept with.
So here it is.

Basically we need two commands one for request an article and one to reset the
whole note.
    
* **get:$title** to get the article with $title
* **reset** resets the note

The returned Wikipedia page must also be processed by the program because you
could not simply paste all HTML code into a Facebook note.
So
[this](https://github.com/koehlma/snippets/blob/master/facebook/wikipedia-bot/wikipedia-bot.py)
is the result.
It uses the Facebook Graph API to change the note and read the comments.
So if you want to use this you need an access token as well as Beautiful Soup 4
for Python 3 installed.
Then just start it with `python wikipedia-bot.py <access_token> <note_id>` to
wait for new comments on the note with *note_id*.
Try it by simply comment on your note, for example *"get:Knoppix"* to load the
Knoppix page of Wikipedia.

### Further Improvments
I think it would also be possible to setup a real HTTP proxy using notes.
It could use a bunch of notes.
For each request it comments the path to the website as well as the headers
with a note id where to find POST data if any.
The server would update the note with a base64 encoded version of the
requested site.
So you could browse the whole internet for free.
Maybe I will try this in the future but for now just using Wikipedia is enough
for me ;).





