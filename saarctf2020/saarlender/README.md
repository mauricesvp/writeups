# saarlender

Saarlender was a web service of the saarCTF 2020.
The user was able to register and login, and create events and messages, both of which could be
public or private.


The backend consisted of many nginx configs, and a custom JS runtime environment.


# Flag Store 1
The first Flag Store was quite basic. Analysing the traffic after the network opened, we quickly
discovered that the checker created users with the username scheme <time_stamp_as_int>[:-2]. This was
quite important as the usernames were not obtainable otherwise. From there we could access the
messages and events of the users, in which the flags were stored.

This was the first exploit we had running of all services, with a couple of variations as well, as
there were multiple similar endpoints for the events and messages to get flags from:

For example:

    /events/<username>
    
    /events/raw/<username>
    
    /events/raw-pub/<username>
   etc.


# Flag Store 2
For the second Flag Store we had to parse the messages, as they contained a "from" and "to" field.
Here we noticed that the checker-created users sent each other messages. This way we could obtain
usernames which otherwise could not be leaked. The flags than could be found in the same places as
for Flag Store 1.


# Flag Store 3
Something very interesting we noticed when analysing the traffic was a /shell endpoint, which the
checker was using. Following the /shell?, one was able to send base64 encoded JavaScript code, which then got executed. However there were quite some limitations, as typical functions like include or system
were disabled. We did not manage to craft any payload which would get us additional flags, although
there was a config file which already hinted that RCE was possible. After the CTF the challenge
creator showed us an example payload: one was able to grep for additional messages (in the `users` directory) which were not
obtainable otherwise, which again contained flags.

# Fixes
We fixed Flag Store 1 and 2 by simply returning nothing for `load_events()`, which was called when accessing
`/events*`.

Regarding Flag Store 3 we did not patch anything, which lead to roughly two lost flags per
tick.

# Misc
Right in the beginning of the CTF a team member noticed the `auth.conf` file, which contained hardcoded credentials.
However we did not find this to be exploitable in any way.

Another sort of fun file was `next_level.py`, which would obfuscate `ngnix.config`.
