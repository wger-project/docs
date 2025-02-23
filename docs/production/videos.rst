.. _videos:

Videos
======

wger also supports adding videos to the exercises and are served locally like
the images.

If you just want to mirror/download the files from wger.de, just do::

 python manage.py download-exercise-videos

If you want to upload your own videos to your instance, it is recommended that
you install the ``ffmpeg`` package both in the system and in the virtualenv.
Without this, no validation can be performed during upload and the video metadata
wont be saved to the database. Due to the size of this package and its
dependencies, they are not installed by default::

    $ apt-get install ffmpeg
    $ pip install ffmpeg-python
