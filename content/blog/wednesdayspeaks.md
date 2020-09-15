---
title: "Wednesday Speaks"
date: 2020-08-29T12:32:36+00:00
image : "images/wednesday_speaks_cover.png"
# author
author : ["Kelsey"]
# categories
categories: ["Projects"]
tags: ["flask", "python", "ngrok", "html", "css"]
# meta description
description: "Making a dog speak with Flask and Ngrok"
# save as draft
draft: false
---

There are many ways to be productive when you are faced with indefinite time stuck inside your home. Making a dog shout lyrics to WAP is usually not one of them.

This is Wednesday, our lovely noodle horse.

![wednesday](/images/wednesday_speaks_3.jpeg)

The idea for this project came from a friend who made a bot to speak out text that was sent by Facebook messenger to a Facebook page. I figured it would be a good *first ever coding project* to try and emulate something that already existed, and add some extra simple functionality. It turned out, as always, to be slightly more complex than I expected.

Facebook Faliures
--- 

There is a Python library called `fbmq` that allows one to interact with the Facebook Messenger Platform API.
To create a simple Facebook messenger bot, you need to set up a Facebook page, and create a Facebook for Developers account.

Unfortunately, Facebook has recently tightened security for their messenger bots.
Making the webhook authenticate and the functionality to work as intended is very fiddly. In the end I decided for my use case, it would be more efficient and open to customisation if I made a web application for users to input text into instead. It would also force me to understand web servers better, and start experimenting with html and css. All of which were really the best learnings from this project.

A Minumum Viable Product
---

The concept for the website app is fairly simple:

1. The user inputs text into a form on a website
2. The text is converted to speech using an API or library
3. The speech is stored as an mp3 file on my local computer
4. The mp3 file is read outloud by my operating system
5. My computer is connected by Bluetooth to a small portable speaker (attached to an unassuming dog)

As I'm most comfortable working in Python, I used `Flask`'s web framework. Flask is a lightweight 'WSGI', which (in language I understand) simply forwards requests from a webserver to my application 'backend'.

My directory for the minimum viable product looked a little like this:

    └── wednesdayspeaks/
        ├── main.py
        ├── templates/
        |   └── home.html
        └── static
            ├──image.jpg
            └── css/
                └── main.css

Where: `main.py` has all the juicy application bits to handle steps 2-4,
`home.html` provides the front-end for the user to input their text (step 1), and `main.css` provided some nice UI, without which you end up with an ugly app like so:

![wednesday_speaks_final1](/images/wednesday_speaks_0.png)

Here is the MVP version of `main.py`:

    from flask import Flask, render_template, request
    from gtts import gTTS
    import os

    # Config
    app = Flask(__name__)

    # Renders the generic homepage
    @app.route("/")
    def homepage():
        return render_template("home.html", title="HOME PAGE")

    # Kicks off logic when request made, renders message
    @app.route("/", methods=["POST"])
    def request_made():
        text = request.form["text"]
        message = ""
        if len(text) < 250 and len(text.split()) < 40:
            tts = gTTS(text=text, lang="en")
            tts.save("text.mp3")
            # to start the file from python
            os.system("start text.mp3")
            message = "thanks bbg"
        else:
            message = "Requested speech is too long. Please shorten your message."
        return render_template("home.html", title="speaktowen", message = message)

    if __name__ == "__main__":
        app.run()

There's a few things to note in this code snippet. Firstly, I've used Google's Text to Speech `gTTS` API to handle step 2 and 3. The `os.system` command handles step 4, which will be discussed in Deployment (spoiler: it's a nightmare to run in a cloud environment).

You will notice that Flask looks for the text input with `request.form['text']`, which references the form element and 'text' id in `home.html`:

    <!DOCTYPE html>
        <html lang="en" dir="ltr">
        <head>
            <meta charset="utf-8">
            <title>Talk to Wednesday</title>
        </head>
        <body>
            <h1> Make Wednesday Speak </h1>
            <img src="/static/image.jpg" alt="it is wednesday my dudes">
            <p>Type what you want Wednesday to say in the box below, then hit 'Submit'</p>
            <form method="POST">
            <div id= "inputs">
                <label for="text">Text:</label> <br>
                <input name="text"> <br>
                <input type="submit">
            </div>
            </form>
            {% if message %}
                <p>{{ message }}</p>
            {% endif %}
        </body>
        </html>

To get this running in a local environment, all that's needed is to run `main.py` from your terminal. Flask handles the rest. 

Deployment
---

Getting my little app functional in a local environment was enough to make me jump for joy. You have to celebrate the little wins in quarantine. 

Next up, it was time to get my app onto the big wide web for everyone to use. From some research, everyone seemed to rave about how simple and easy Google App Engine (GAE) is to get your app hosted on a serverless environment. Great!! I thought. However, I had overlooked the fact that a core part of my app is reading out the saved mp3 file on my local computer (step 4). All my implementation of GAE and Google Cloud Storage was capable of was reading out out from the user's browser, rather than Wednesday herself.

Finally I found a deployment method which would work for this use case: `ngrok`. Ngrok creates a public URL for your local web server. It's a great and very user-friendly method for testing.
It's as simple as installing a `flask-ngrok` package, or downloading and installing ngrok itself.

By this point, my app up on the web, and ready for trial by fire.

Testing and Iterating
---

I wasn't ready for what was unleashed on my app.

{{< youtube 7jxeSdbjboU >}}

Besides the general filth that Wednesday was forced to say, I was pleased with the app's performance, but learned about some key bugs and improvements from the testing phase.

#####  Wait what did she just say?? ##### 

Many times during testing, Wednesday would say something and we missed it because we were too loud.

I needed to implement a feature that tracked the history of requests made to the web app. For this, a database was needed. Luckily, Flask has good integration with SQLAlchemy through a package called `flask_sqlalchemy`. 

To set up the database, you need some config but it's all very simple and all can be run inside `main.py`. The database was spun up quickly, and to allow all users to see past requests, I just needed to add another html page in `templates`and a route in `main.py`:

    # Renders the request history page
    @app.route("/requests_log", methods=["GET","POST"])
    def get_table_data():
        data = db.session.query(
                Table.id, Table.username, Table.text_request).order_by(
                Table.id.desc()).limit(5)
        return render_template("requests_log.html",  data = data)

#####  Delayed speech #####

While `gTTS` creates realistic sounding speech, the request to the API takes around 3-4 seconds. This delay is frustrating when you're just trying to get a dog to say something in response to a conversation in real-time.

I needed to explore alternatives. I found a package called `pyttsx3` which works offline and is almost instantaneous. Sure, it sounds more like a robot, but it's almost more funny to have a robo-dog than a human dog.

##### Simultaneous requests ##### 

A couple of times during testing, users tried to make requests to the app at the same time. This caused the server to crash, as a loop in `pyttsx3` was already started. `pyttsx3` has some inbuilt engine threading and queue capabilities, but these wouldn't work very well in my app. So I used Python's threading and queue libraries in an external loop to enable the app to simply stop a new request interrupting an running one.

##### Malicious WAP attacks ##### 

Unfortunately my little app became a target for some skilled programmer friends, who created a bot to fire sections of songs at the URL repeatedly. This DOS attack culminated in Wednesday saying some particularly lurid lyrics and also the server being unaccessable to any other users.

To protect against any further such attacks, I utilised an IP address tracker called `Limiter` to cut off access to the website if a user made more than 3 requests in a minute. That'll teach em.

##### Mobile friendly UI ##### 

The UI I made in my original MVP was non-responsive and as most of the people using the app accessed it on mobile devices, this was not a great user experience. I added the following to `home.html` in the `<head>` tag to make the web version scale to mobile: `<meta name="viewport" content="width=device-width, initial-scale=1.0">`


##### Bluetooth cutting ##### 

An frustrating but easy to fix bug arose when the cheap mini bluetooth speaker I purchased for this project started to cut off the first word of each text request, due to it 'sleeping' to save battery between saying things. The fix was quick and dirty - add a word to each text string: `text = "um" + text`
This fixed the bug perfectly, and we would never hear the 'um' that prefaced the sentence.

##### Unacceptable requests ##### 

Finally, perhaps the worst bug arose when some of my housemates used the app to turn against its creator. Hearing Wednesday say things like 'I hate Kelsey' was entirely unacceptable. I implemented some custom rules to ensure any nasty comments were redirected to another housemate... 

    if "hate" in text:
        text = "I hate Anna"

##### Security ##### 

Even though ngrok supports tunnelling to a HTTPS url, there's always security concerns with exposing a local server. To combat this, it's possible to run the app on a virtual machine. I used VMBox to create a Windows machine, which to run smoothly required me to uninstall the Sims 4 on my physical machine (cry) but it's all worth it for that peace o'mind.

##### The final product ##### 

Here's some screengrabs of the final site in action.

![wednesday_speaks_final1](/images/wednesday_speaks_1.png)


![wednesday_speaks_final1](/images/wednesday_speaks_2.png)

This project was a great learning exercise. I filled some missing basic knowledge about website building, virtual machines and cloud platforms, while I also touched up technical skills in Flask, HTML and CSS. But most importantly, it was enjoyable (less so for Wednesday) and gave me a thirst to do more tinkering.