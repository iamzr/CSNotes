# How long does it take to start up a spark session?

You should have set up you environment.
I'll be using a spark environment starting with one master node and one worker node.

In order to time how long it'll take to start up a spark session, we need to create a script that will that a timestamp and start up a spark session, and then once the session has been started up will do another timestamp.
The difference will be the startup time.

Now you can't just simply run this script you'd have to submit the job to the spark master node.
Then once "inside" spark we'll be able to run our script.

Note: sending our script to the master node does not in itself create a spark session. 