.. _manual_connected_applications:

Connecting other applications
=============================

Other applications can work with your wger data on your behalf: a meal tracker,
an AI assistant that logs your sets while you talk to it. You never give them
your password. Instead you log in on wger once, agree to what the application
may do, and wger hands it a key of its own and which you can take away again at
any time.

This is the opposite direction of logging in to wger *with* a Google or GitHub
account. Here it is wger that vouches for you, towards someone else.

Not every wger instance offers this. If you run your own and the pages below
are missing, the OAuth2 provider is switched off; see
:doc:`/administration/oauth2_provider`.


Giving an application access
----------------------------

You start in the application, not in wger: it sends you to a wger page that
names it and lists what it is asking for. Read that list. It is the only point
in the whole process where you decide, and the application gets exactly what it
says there:

**View your user ID**
  Which wger account this is. Every application asks for this.

**View your training, nutrition and body data**
  Read access to everything: routines, logs, meals, weight, measurements.

**Add and change your training, nutrition and body data**
  Write access to the same. An application with this permission can create,
  change and delete entries, not limited to the things it added itself.

If you agree, you are sent back to the application and it can start working.

.. note::

   An application that runs somewhere else sees your data somewhere else. With
   an AI assistant in particular, whatever it reads from your account travels
   to the service that runs the model. Only connect applications you would be
   comfortable handing that data to.


Taking it back
--------------

Go to *Settings* → *Connected applications*. Every application you have given
access to is listed there with what it may do, and a **Disconnect** button.

Disconnecting takes effect immediately. The application cannot get back in on its
own: it has to ask again, and you will see the same page you saw the first
time. So disconnecting something you are unsure about costs you nothing but the
click it takes to reconnect.

An application you stop using disappears from the list by itself once its key
runs out, which the page tells you the date of.
