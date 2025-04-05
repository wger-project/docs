Changelog
=========

2.4 - IN DEVELOPMENT
--------------------

🔧 Upgrade steps from 2.3
~~~~~~~~~~~~~~~~~~~~~~~~~
* Update python libraries ``pip3 install -r requirements.txt``
* Migrate database ``python manage.py migrate``
* Update CSS and JS libraries ``yarn install``
* Compile the CSS ``yarn build:css:sass``
* Update static files (only production): ``python3 manage.py collectstatic``

🚀 Features
~~~~~~~~~~~
*



2.3
----

**2025-04-05**


🔧 Upgrade steps from 2.2
~~~~~~~~~~~~~~~~~~~~~~~~~
* Update python libraries ``pip3 install -r requirements.txt``
* Migrate database ``python manage.py migrate``
* Update CSS and JS libraries ``yarn install``
* Compile the CSS ``yarn build:css:sass``
* Update static files (only production): ``python3 manage.py collectstatic``

🚀 Features
~~~~~~~~~~~

* Axes behind reverse proxy by :github:user:`@bbkz <bbkz>` in :github:pull:`1521`
* Fix wrong default setting in docker image by :github:user:`@bbkz <bbkz>` in :github:pull:`1531`
* Add sync method for ingredients by :github:user:`@rolandgeider <rolandgeider>` in :github:pull:`1546`
* Add progress bar to load-online-fixtures by :github:user:`@ebwinters <ebwinters>` in :github:pull:`1561`
* Move to pyproject.toml by :github:user:`@rolandgeider <rolandgeider>` in :github:pull:`1565`
* Add full text search by :github:user:`@rolandgeider <rolandgeider>` in :github:pull:`1594`
* put building/obtaining image instructions before trying to run it. by :github:user:`@Dieterbe <Dieterbe>` in :github:pull:`1655`
* add support for fiber goal by :github:user:`@Dieterbe <Dieterbe>` in :github:pull:`1664`
* Add myself to the Polish translators  by :github:user:`@Maniues <Maniues>` in :github:pull:`1661`
* fix calendar accordion by :github:user:`@bbkz <bbkz>` in :github:pull:`1681`
* Refactor product/ingredient import by :github:user:`@rolandgeider <rolandgeider>` in :github:pull:`1666`
* Export prometheus metrics by :github:user:`@rolandgeider <rolandgeider>` in :github:pull:`1685`
* Fixed a typo by :github:user:`@JLaField <JLaField>` in :github:pull:`1695`
* Allow deactivating the language filter when searching for ingredients and exercises by :github:user:`@rolandgeider <rolandgeider>` in :github:pull:`1687`
* docker image: support loading exercise videos, fixtures, nutrition info by :github:user:`@Dieterbe <Dieterbe>` in :github:pull:`1746`
* Allow env specification of Redis connection SSL parameters by :github:user:`@taylor-fuller <taylor-fuller>` in :github:pull:`1751`
* Improve docker image by :github:user:`@rolandgeider <rolandgeider>` in :github:pull:`1786`
* give dummy meals names by :github:user:`@Dieterbe <Dieterbe>` in :github:pull:`1795`
* Update django.po by :github:user:`@victorbmlabs <victorbmlabs>` in :github:pull:`1798`
* Dynamically update WeightUnit from user preferences when creating workout by :github:user:`@kmoy1 <kmoy1>` in :github:pull:`1807`
* Rework the calendar page by :github:user:`@rolandgeider <rolandgeider>` in :github:pull:`1824`
* make __main__ use pyinvoke ``Program`` as entrypoint by :github:user:`@eyJhb <eyJhb>` in :github:pull:`1833`
* Fix incorrectly placed <h1> tag and replace it with an <h4> tag for the "Members List" DataTable. by :github:user:`@navyjosh <navyjosh>` in :github:pull:`1843`
* Fix/registration by :github:user:`@Maralai <Maralai>` in :github:pull:`1855`
* fixes #1278 by :github:user:`@blsouthcott <blsouthcott>` in :github:pull:`1365`
* Add Management Command for Async Ingredient Synchronization by :github:user:`@crypto-a <crypto-a>` in :github:pull:`1876`
* Add language filter to sync-ingredients management command. by :github:user:`@navyjosh <navyjosh>` in :github:pull:`1875`
* Fixes for adding language filter to sync-ingredients management command. by :github:user:`@scrapcode <scrapcode>` in :github:pull:`1894`
* Clean apt temporary files in the base Docker image by :github:user:`@PeterDaveHello <PeterDaveHello>` in :github:pull:`1906`
* Flexible routines by :github:user:`@rolandgeider <rolandgeider>` in :github:pull:`1827`


2.2
---
**2023-11-06**

https://github.com/wger-project/wger/releases/tag/2.2

🔧 Upgrade steps from 2.1
~~~~~~~~~~~~~~~~~~~~~~~~~

* Update python libraries ``pip3 install -r requirements.txt``
* Migrate database ``python manage.py migrate``
* Update CSS and JS libraries ``yarn install``
* Compile the CSS ``yarn build:css:sass``
* Update static files (only production): ``python3 manage.py collectstatic``
* Load new permissions ``python3 manage.py loaddata groups.json categories.json``
* Read the section on celery in this documentation on how to set it up. While at the
  moment this is not needed and only provides quality of life features, in the future
  this might change



🚀 Features
~~~~~~~~~~~

* Improvements to the nutritional plan handling. Users don't have to setup a detailed
  plan with meals anymore, instead they can just log their meals `#817`_
* Allow users to set goals for their nutritional plans. This basically works
  like the sum of the individual meals, but is simpler and easier to setup `#1003`_
* Implemented nutrition page with react
* Added general measurements tracking to the web application `#875`_
* Added JWT authentication to the REST API (thanks `@RohanKaran`_!) `#1047`_
* Added images to ingredients. This can now be shown in the nutritional plan,
  the autocompleter, etc. `#653`_
* Add a celery queue for longer running or periodic tasks. At the moment this is only used
  to keep the exercises in sync and download the ingredient images, but other features
  are planned `#1174`_
* When scanning a product, fetch the data from the live OFF server if it is not
  found locally `#1012`_, `#1348`_
* Added brute protection against brute force login attacks (thanks `@RohanKaran`_!) `#1096`_
* Reworked the landing page (thanks `@12people`_!) `#1112`_
* Allow to set the minimum account age for users to contribute exercises (thanks `@mohammadrafigh`_!) `#1187`_
* Document the API with openAPI, redoc and all the goodies that come from it
  (better online docs, being able to generate clients, etc.) `#1127`_
* Allow searching exercises and ingredient in English in addition to the user's
  currently selected language `#1238`_
* More flexible user (sub)locale switching. This specially affected English users
  that would be shown dates in US format `#1245`_
* Add a deletion log for exercises. This allows exercises to be marked as deleted by
  the system. Alternatively a replacement can be set so that when other instances
  sync the exercises logs and routines are correctly updated `#1237`_
* Show all the authors of an exercise and any of its child items (translations,
  images, videos, etc.) `#1137`_
* Allow users to give meals a description (thanks `@mohammadrafigh`_!) `#822`_
* Added style field (Foto, 3D, etc.) to exercise image (thanks `@LucasSD`_!) `#822`_
* Added exercise edit history (thanks `@ImTheTom`_!) `#1082`_
* Added JWT authentication to rest API (thanks `@RohanKaran`_!) `#1134`_
* Add support for sub-locales in the application such as en-gb  `#1275`_
* Moved some parts of routine management to react `#1328`_

🧰 Maintenance
~~~~~~~~~~~~~~

* Improvements to the Open Food Facts product importer. The setup has been simplified
  with a docker compose, making the process much more streamlined. `#1505`_
* `#1137`_  (thanks `@AdamPetik`_!)
* Show last modified datetime of exercises in the API `#1387`_
* Better handling of exercises without translations `#1319`_
* Split the dummy generator into individual files `#919`_
* Update bootstrap to current version `#1109`_
* Update django to current version `#1110`_
* Bettler handling of exercises UUIDs (thanks `@Gr8ayu`_) `#675`_
* Add foreign key to meals on log (thanks `@Alig1493`_) `#842`_
* Make URL for media, static and login redirect configurable (thanks `@novalis111`_) `#1020`_
* Configure django axes (thanks `@RohanKaran`_) `#1143`_
* Add tzdate package to docker base image (thanks `@bbkz`_) `#1408`_


🐛 Bug Fixes
~~~~~~~~~~~~

* Fix issue with django axes and mobile app `#1163`_
* Correctly format decimal places in numbers according to the user's locale `#1402`_
* Fix issue when a user tried to register with an existing email via the app (thanks `@JayanthBontha`_!) `#1459`_
* Fix bug in the demo entries generator (thanks `@JayanthBontha`_!) `#1278`_
* Fix issue with password reset links and expired tokens (thanks `@RohanKaran`_!) `#1154`_
* Fix issue with password reset links and expired tokens (thanks `@RohanKaran`_!) `#1287`_
* Fix issue that prevented users from resetting their password (thanks `@RohanKaran`_!) `#1154`_
* Fix import error (thanks `@sophiamartelli`_!) `#986`_
* Use either TLS or SSL for emails (thanks `@bbkz`_!) `#1514`_
* Fix bug in the link used in the password reset link `#1320`_
* Fix bug in the weight log chart `#1308`_


.. _#1514 : https://github.com/wger-project/wger/issues/1514
.. _#1505 : https://github.com/wger-project/wger/issues/1505
.. _#1459 : https://github.com/wger-project/wger/issues/1459
.. _#1408 : https://github.com/wger-project/wger/issues/1408
.. _#1402 : https://github.com/wger-project/wger/issues/1402
.. _#1387 : https://github.com/wger-project/wger/issues/1387
.. _#1348 : https://github.com/wger-project/wger/issues/1348
.. _#1328 : https://github.com/wger-project/wger/issues/1328
.. _#1320 : https://github.com/wger-project/wger/issues/1320
.. _#1319 : https://github.com/wger-project/wger/issues/1319
.. _#1308 : https://github.com/wger-project/wger/issues/1308
.. _#1287 : https://github.com/wger-project/wger/issues/1287
.. _#1278 : https://github.com/wger-project/wger/issues/1278
.. _#1275 : https://github.com/wger-project/wger/issues/1275
.. _#1245 : https://github.com/wger-project/wger/issues/1245
.. _#1238 : https://github.com/wger-project/wger/issues/1238
.. _#1237 : https://github.com/wger-project/wger/issues/1237
.. _#1187: https://github.com/wger-project/wger/issues/1187
.. _#1174: https://github.com/wger-project/wger/issues/1174
.. _#1163: https://github.com/wger-project/wger/issues/1163
.. _#1154: https://github.com/wger-project/wger/issues/1154
.. _#1143: https://github.com/wger-project/wger/issues/1143
.. _#1137: https://github.com/wger-project/wger/issues/1137
.. _#1134: https://github.com/wger-project/wger/issues/1134
.. _#1127: https://github.com/wger-project/wger/issues/1127
.. _#1112: https://github.com/wger-project/wger/issues/1112
.. _#1110: https://github.com/wger-project/wger/issues/1110
.. _#1109: https://github.com/wger-project/wger/issues/1109
.. _#1096: https://github.com/wger-project/wger/issues/1096
.. _#1082 : https://github.com/wger-project/wger/issues/1082
.. _#1047: https://github.com/wger-project/wger/issues/1047
.. _#1020 : https://github.com/wger-project/wger/issues/1020
.. _#1012: https://github.com/wger-project/wger/issues/1012
.. _#1003: https://github.com/wger-project/wger/issues/1003
.. _#986: https://github.com/wger-project/wger/issues/986
.. _#919: https://github.com/wger-project/wger/issues/919
.. _#875: https://github.com/wger-project/wger/issues/875
.. _#842: https://github.com/wger-project/wger/issues/842
.. _#822: https://github.com/wger-project/wger/issues/822
.. _#817: https://github.com/wger-project/wger/issues/817
.. _#774: https://github.com/wger-project/wger/issues/774
.. _#675: https://github.com/wger-project/wger/issues/675
.. _#653: https://github.com/wger-project/wger/issues/653
.. _@12people: https://github.com/12people
.. _@RohanKaran: https://github.com/RohanKaran
.. _@AdamPetik: https://github.com/AdamPetik
.. _@mohammadrafigh: https://github.com/mohammadrafigh
.. _@JayanthBontha : https://github.com/JayanthBontha
.. _@Gr8ayu: https://github.com/Gr8ayu
.. _@LucasSD: https://github.com/LucasSD
.. _@Alig1493: https://github.com/Alig1493
.. _@sophiamartelli: https://github.com/sophiamartelli
.. _@novalis111: https://github.com/novalis111
.. _@RohanKaran: https://github.com/RohanKaran
.. _@bbkz: https://github.com/bbkz


2.1
---
**2022-10-11**

Upgrade steps from 2.0:

* Install ffmpeg if you want to upload videos (consult documentation).
* Update python libraries ``pip3 install -r requirements.txt``
* To sync the new exercises:

  * Run migrations ``python3 manage.py migrate``
  * delete all exercises not in use ``python manage.py delete-unused-exercises``
    (this will delete all exercises that are currently in the database but are
    not part of any workout, log, etc. You will be promted before the script does anything)
  * get the new exercises ``python manage.py sync-exercises`` (Also note that if
    you don’t perform these steps and directly run a regular sync the worst
    that can happen is that you might have some duplicate exercises in your
    installation)
  * get the new images ``python manage.py download-exercise-images``
  * get the new videos ``python manage.py download-exercise-videos`` (please
    note that this needs more space)

* Update CSS and JS libraries ``yarn install``
* Compile the CSS ``yarn build:css:sass``
* Update static files (only production): ``python3 manage.py collectstatic``
* Load new permissions ``python3 manage.py loaddata groups.json categories.json``


🚀 Features:

* The exercise database has undergone a huge cleanup, combining duplicates and
  translations, deleting stubs, etc. Refreshed the UI for the exercise overview,
  detail view and contribution page. It is now easier (or at all possible) to
  submit, correct and translate the exercises. `#1120`_
* New gallery where users can upload pictures to track their progress `#572`_
* Exercises can now have videos. Also many thanks to Goulart for providing 150 videos `#970`_
  and releasing them under the CC-BY-SA license.
* Add templates / centrally managed workouts (thanks `@qwert45hi`_) `#639`_
* Add comment filed to set for user notes `#702`_
* Custom measurements such as biceps size or body fat `#133`_
* Add picture type to exercise images (thanks `@LucasSD`_) `#589`_
* Add optional relation from nutritional diary to meal (thanks `@Alig1493`_) `#819`_
* Muscles now have a "common" name, besides their names in Latin (thanks `@ImTheTom`_) `#1041`_
* Allow to add nutritional plan diary entries for other dates (thanks `@ImTheTom`_) `#520`_

🐛 Bug Fixes:

* Adding a new workout day no longer needs to be saved twice (thanks `@ImTheTom`_) `#974`_

🧰 Maintenance:

* Exercise API response is now cached (thanks `@ImTheTom`_) `#1033`_

* Changes to the REST API:

  + /exercisebaseinfo/
    - New endpoint to get exercise information grouped by the base exercise
  + /language/
    - Also expose the language ID
  + /exerciseimage/
    - ``exercise`` was renamed to  ``exercise_base`` (was pointing there anyway)
    - New field ``style``
  + /workout/
    - ``comment`` was renamed to name
    - field ``description`` was added, for longer descriptions
  + /set/
    - field ``comment`` added, for user notes
  + /nutritiondiary/
    - field ``meal`` added, optional reference to meal
  + /min-app-version/
    - New endpoint indicating minimum required version for flutter app
* `#666`_, `#667`_, `#656`_ (thanks `@jackmulligan-ire`_), `#716`_

.. _#133: https://github.com/wger-project/wger/issues/133
.. _#520: https://github.com/wger-project/wger/issues/520
.. _#572: https://github.com/wger-project/wger/issues/572
.. _#589: https://github.com/wger-project/wger/issues/589
.. _#639: https://github.com/wger-project/wger/issues/639
.. _#656: https://github.com/wger-project/wger/issues/656
.. _#666: https://github.com/wger-project/wger/issues/666
.. _#667: https://github.com/wger-project/wger/issues/667
.. _#702: https://github.com/wger-project/wger/issues/702
.. _#716: https://github.com/wger-project/wger/issues/716
.. _#819: https://github.com/wger-project/wger/issues/819
.. _#970: https://github.com/wger-project/wger/issues/970
.. _#974: https://github.com/wger-project/wger/issues/974
.. _#1033: https://github.com/wger-project/wger/issues/1033
.. _#1120: https://github.com/wger-project/wger/pull/1120
.. _#1041: https://github.com/wger-project/wger/pull/1041

.. _@Alig1493: https://github.com/Alig1493
.. _@LucasSD: https://github.com/LucasSD
.. _@qwert45hi: https://github.com/qwert45hi
.. _@ImTheTom: https://github.com/ImTheTom
.. _@jackmulligan-ire: https://github.com/jackmulligan-ire


2.0
--------------------
**2021-05-01**

Upgrade steps from 1.9:

* Update python libraries ``pip3 install -r requirements.txt``
* Install ``yarn`` and ``sass`` (e.g. ``sudo npm install -g yarn sass``)
* Update CSS and JS libraries ``yarn install``
* Compile the CSS ``yarn build:css:sass``
* Run migrations ``python3 manage.py migrate``
* Update data ``python3 manage.py loaddata licenses.json languages.json language_config.json``
* Load new ingredients (note that this will overwrite any ingredients that you
  might have added) ``wger load-online-fixtures``
* Update static files (only production): ``python3 manage.py collectstatic``
* Subcommands for ``wger`` now use dashes in their names (i.e. create-settings
  instead of create_settings)


🚀 Features:

* Add nutrition diary to log the daily calories actually taken `#284`_, `#501`_
  and `#506`_ (thanks `@WalkingPizza`_ and `@oconnelc`_)
* Support for reps-in-reserve (RiR) in workout plans and logs `#479`_
  (thanks `@SkyNetIndustry`_)
* Improved user experience, on desktop and mobile `#337`_
* Around 70000 new ingredients with Open Food Facts import with more to come `#422`_
  (thanks `@harlenesamra`_, `@nikithamurikinati`_ and `@jcho1`_)
* Group common exercise information such as muscles, etc. for more easy translations,
  data management, etc. `#448`_ (thanks `@nikithamurikinati`_, `@harlenesamra`_,
  `@jcho17`_, `@vaheeshta`_ and `@jeevikaghosh`_)
* Group similar exercises such as wide grip, reverse, etc. `#555`_
  (thanks `@ryowright`_)
* Improved info endpoints for exercises and ingredients, this saves additional
  API calls `#411`_
* Show BMI on weight graph `#462`_ (thanks `@Svn-Sp`_)
* Allow user to edit and delete body weight entries `#478`_ (thanks `@beingbiplov`_)
* Show kJoules as well as kcal in nutritional plan `#568`_  (thanks `@nopinter`_ and `@derekli17`_)
* Check name similarity when adding exercises to avoid duplicates `#551`_
  (thanks `@lydiaxing`_, `@eq8913`_, `@Hita-K`_)
* Return the muscle background images in the REST API `#547`_ (thanks `@gengkev`_)


🐛 Bug Fixes:

* `#368`_, `#379`_, `#426`_ (thanks `@austin-leung`_), `#499`_, `#505`_, `#504`_,
  `#511`_, `#516`_, `#522`_, `#554`_ and `#560`_ (thanks `@sandilsranasinghe`_),
  `#564`_, `#565`_, `#615`_, `#560`_ (thanks `@bradsk88`_), `#617`_ (thanks `@Sidrah-Madiha`_),
  `#636`_, `#640`_, `#642`_, `#648`_, `#650`_


🧰 Maintenance:

* Moved translations to weblate `#266`_
* Improved docker and docker-compose images `#340`_
* Updated many libraries to the last version (bootstrap, font awesome, etc.)
* Use yarn to download CSS/JS libraries
* Improvements to documentation (e.g. `#494`_)
* Improved cache handling `#246`_ (thanks `@louiCoder`_)
* Others: `#450`_ (thanks `@Rkamath2`_), `#631`_ (thanks `@harlenesamra`_), `#664`_ (thanks `@calvinrw`_),

.. _@Svn-Sp: https://github.com/Svn-Sp
.. _@louiCoder: https://github.com/louiCoder
.. _@WalkingPizza: https://github.com/WalkingPizza
.. _@oconnelc: https://github.com/oconnelc
.. _@beingbiplov: https://github.com/beingbiplov
.. _@sandilsranasinghe: https://github.com/sandilsranasinghe
.. _@Rkamath2: https://github.com/Rkamath2
.. _@SkyNetIndustry: https://github.com/SkyNetIndustry
.. _@ryowright: https://github.com/ryowright
.. _@austin-leung: https://github.com/austin-leung
.. _@harlenesamra: https://github.com/harlenesamra
.. _@lydiaxing: https://github.com/lydiaxing
.. _@eq8913: https://github.com/eq8913
.. _@Hita-K: https://github.com/Hita-K
.. _@derekli17: https://github.com/derekli17
.. _@nopinter: https://github.com/nopinter
.. _@gengkev: https://github.com/gengkev
.. _@nikithamurikinati: https://github.com/nikithamurikinati
.. _@jcho1: https://github.com/jcho1
.. _@jcho17: https://github.com/jcho17
.. _@vaheeshta: https://github.com/vaheeshta
.. _@jeevikaghosh: https://github.com/jeevikaghosh
.. _@bradsk88: https://github.com/bradsk88
.. _@Sidrah-Madiha: https://github.com/Sidrah-Madiha
.. _@calvinrw: https://github.com/calvinrw


.. _#246: https://github.com/wger-project/wger/issues/246
.. _#266: https://github.com/wger-project/wger/issues/266
.. _#284: https://github.com/wger-project/wger/issues/284
.. _#337: https://github.com/wger-project/wger/issues/337
.. _#340: https://github.com/wger-project/wger/issues/340
.. _#368: https://github.com/wger-project/wger/issues/368
.. _#379: https://github.com/wger-project/wger/issues/379
.. _#411: https://github.com/wger-project/wger/issues/411
.. _#422: https://github.com/wger-project/wger/issues/422
.. _#426: https://github.com/wger-project/wger/issues/426
.. _#448: https://github.com/wger-project/wger/issues/448
.. _#450: https://github.com/wger-project/wger/issues/450
.. _#462: https://github.com/wger-project/wger/issues/462
.. _#478: https://github.com/wger-project/wger/issues/478
.. _#479: https://github.com/wger-project/wger/issues/479
.. _#494: https://github.com/wger-project/wger/issues/494
.. _#499: https://github.com/wger-project/wger/issues/499
.. _#501: https://github.com/wger-project/wger/issues/501
.. _#504: https://github.com/wger-project/wger/issues/504
.. _#505: https://github.com/wger-project/wger/issues/505
.. _#506: https://github.com/wger-project/wger/issues/506
.. _#511: https://github.com/wger-project/wger/issues/511
.. _#516: https://github.com/wger-project/wger/issues/516
.. _#522: https://github.com/wger-project/wger/issues/522
.. _#547: https://github.com/wger-project/wger/issues/547
.. _#550: https://github.com/wger-project/wger/issues/550
.. _#551: https://github.com/wger-project/wger/issues/551
.. _#554: https://github.com/wger-project/wger/issues/554
.. _#555: https://github.com/wger-project/wger/issues/555
.. _#560: https://github.com/wger-project/wger/issues/560
.. _#564: https://github.com/wger-project/wger/issues/564
.. _#565: https://github.com/wger-project/wger/issues/565
.. _#568: https://github.com/wger-project/wger/issues/568
.. _#615: https://github.com/wger-project/wger/issues/615
.. _#617: https://github.com/wger-project/wger/issues/617
.. _#631: https://github.com/wger-project/wger/issues/631
.. _#636: https://github.com/wger-project/wger/issues/636
.. _#640: https://github.com/wger-project/wger/issues/640
.. _#642: https://github.com/wger-project/wger/issues/642
.. _#648: https://github.com/wger-project/wger/issues/648
.. _#650: https://github.com/wger-project/wger/issues/650
.. _#664: https://github.com/wger-project/wger/issues/664



1.9
---
**2020-06-29**

Upgrade steps from 1.8:

* Django update to 3.x: ``pip install -r requirements.txt``
* Database upgrade: ``python manage.py migrate``
* Update static files (only production): ``python manage.py collectstatic``

New features:

* Allow users to enter their birthdate instead of just the age (thanks `@dtopal`_) `#332`_
* Work to ensure that mobile templates are used when appropriate
* Added optional S3 static asset hosting.
* Drop Python 2 support.
* Replaced django-mobile with django-user_agent (and some custom code)
  This isn't as slick as django-mobile was, but it unblocks possible Django 2.x support.
* Update many dependencies to current versions.

Improvements:

* Improve the look of weight graph (thanks `@alokhan`_) `#381`_
* Added password validation rules for more security
* Exercise image downloader checks only accepted exercises (thanks `@gmmoraes`_) `#363`_
* Use a native data type for the exercises' UUID (thanks `@gmmoraes`_) `#364`_
* Increase speed of testsuite by performing the tests in parallel (thanks `@Mbarak-Mbigo`_) `wger_vulcan/#6`_
* Update screen when adding an exercise to the workout while using set slider (thanks `@gmmoraes`_) `#374`_
* Work to slim docker image
  * Download images at startup - If `DOWNLOAD_IMGS` environmental variable is set to `TRUE`
  * Uninstall dev packages
* Update Ubuntu version used in docker container.
* Fixed a handful of hard coded static path references to use `static` taglib
* Updated tinymce theme for v5.

Other improvements and bugfixes: `#336`_, `#359`_,`#386`_, `#443`_

.. _@gmmoraes: https://github.com/gmmoraes
.. _@Mbarak-Mbigo: https://github.com/Mbarak-Mbigo
.. _@dtopal: https://github.com/dtopal

.. _wger_vulcan/#6: https://github.com/andela/wger_vulcan/pull/6

.. _#332: https://github.com/wger-project/wger/issues/332
.. _#336: https://github.com/wger-project/wger/issues/336
.. _#359: https://github.com/wger-project/wger/issues/359
.. _#363: https://github.com/wger-project/wger/issues/363
.. _#364: https://github.com/wger-project/wger/issues/364
.. _#374: https://github.com/wger-project/wger/issues/374
.. _#381: https://github.com/wger-project/wger/issues/381
.. _#386: https://github.com/wger-project/wger/issues/386
.. _#443: https://github.com/wger-project/wger/issues/443


1.8
---
**2017-04-05**

.. warning ::
   There have been some changes to the installation procedure. Calling 'invoke'
   on its own has been deprecated, you should use the 'wger' command (which
   accepts the same options). Also, some of these commands have been renamed:

   * ``start_wger`` to ``wger``
   * ``bootstrap_wger`` to ``bootstrap``

Upgrade steps from 1.7:

* Django update to 1.9: ``pip install -r requirements.txt``
* Database upgrade: ``python manage.py migrate``
* Reset cache: ``python manage.py clear-cache --clear-all``
* Due to changes in the JS package management, you have to delete
  wger/core/static/bower_components and do a ``python manage.py bower install``
* Update static files (only production): ``python manage.py collectstatic``
* Load new the languages fixtures as well as their configuration
  ``python manage.py loaddata languages`` and
  ``python manage.py loaddata language_config``
* New config option in settings.py: ``WGER_SETTINGS['TWITTER']``. Set this if
  your instance has its own twitter account.

New languages:

* Norwegian (many thanks to Kjetil Elde `@w00p`_ `#304`_)
* French (many thanks to all translators)

New features:

* Big ingredient list in Dutch, many thanks to alphafitness.club!
* Add repetition (minutes, kilometer, etc.) and weight options (kg, lb, plates, until failure) to sets `#216`_ and `#217`_
* Allow administrators to deactivate the guest user account `#330`_
* Add option to show the gym name in the header instead of the application name, part of `#214`_
* Exercise names are now capitalized, making them more consistent `#232`_
* Much improved landing page (thanks `@DeveloperMal`_) `#307`_
* Add extended PDF options to schedules as well (thanks `@alelevinas`_ ) `#272`_
* Show trained secondary muscles in workout view (thanks `@alokhan`_ ) `#282`_
* Use the metricsgraphics library to more easily draw charts `#188`_
* Removed persona (browserID) as a login option, the service is being discontinued `#331`_

Improvements:

* Check and enforce style guide for JS files `#317`_ (`@petervanderdoes`_)
* BMI calculator now works with pounds as well (thanks `@petervanderdoes`_) `#318`_
* Give feedback when autocompleter didn't find any results `#293`_
* Make exercise names links to their detail page in training log pages `#350`_
* Better GUI consistency in modal dialogs (thanks `@jstoebel`_ ) `#274`_
* Cache is cleared when editing muscles (thanks `@RyanSept`_ `@pythonGeek`_  ) `#260`_
* Fields in workout log form are no longer required, making it possible to only log weight for certain exercises `#334`_
* New, more verbose, API endpoint for exercises, (thanks `@andela-bmwenda`_)
* The dashboard page was improved and made more user friendly `#201`_ (partly)
* Replace jquery UI's autocompleter and sortable this reduces the size of JS and CSS `#78`_ and `#79`_
* Update to D3js v4 `#314`_, `#302`_
* Remove hard-coded CC licence from documentation and website `#247`_

Other improvements and bugfixes:     `#25`_, `#243`_, `#279`_, `#275`_, `#270`_,
`#258`_, `#257`_, `#263`_, `#269`_, `#296`_, `#297`_, `#303`_, `#311`_, `#312`_,
`#313`_, `#322`_, `#324`_, `#325`_


.. _#25: https://github.com/wger-project/wger/issues/25
.. _#78: https://github.com/wger-project/wger/issues/78
.. _#79: https://github.com/wger-project/wger/issues/79
.. _#188: https://github.com/wger-project/wger/issues/188
.. _#201: https://github.com/wger-project/wger/issues/201
.. _#214: https://github.com/wger-project/wger/issues/214
.. _#216: https://github.com/wger-project/wger/issues/216
.. _#217: https://github.com/wger-project/wger/issues/217
.. _#232: https://github.com/wger-project/wger/issues/232
.. _#243: https://github.com/wger-project/wger/issues/243
.. _#248: https://github.com/wger-project/wger/issues/248
.. _#247: https://github.com/wger-project/wger/issues/247
.. _#260: https://github.com/wger-project/wger/issues/260
.. _#263: https://github.com/wger-project/wger/issues/263
.. _#269: https://github.com/wger-project/wger/issues/269
.. _#272: https://github.com/wger-project/wger/issues/272
.. _#274: https://github.com/wger-project/wger/issues/274
.. _#282: https://github.com/wger-project/wger/issues/282
.. _#293: https://github.com/wger-project/wger/issues/293
.. _#296: https://github.com/wger-project/wger/issues/296
.. _#297: https://github.com/wger-project/wger/issues/297
.. _#302: https://github.com/wger-project/wger/issues/302
.. _#303: https://github.com/wger-project/wger/issues/303
.. _#304: https://github.com/wger-project/wger/issues/304
.. _#307: https://github.com/wger-project/wger/issues/307
.. _#311: https://github.com/wger-project/wger/issues/311
.. _#312: https://github.com/wger-project/wger/issues/312
.. _#313: https://github.com/wger-project/wger/issues/313
.. _#314: https://github.com/wger-project/wger/issues/314
.. _#317: https://github.com/wger-project/wger/issues/317
.. _#318: https://github.com/wger-project/wger/issues/318
.. _#322: https://github.com/wger-project/wger/issues/322
.. _#324: https://github.com/wger-project/wger/issues/324
.. _#325: https://github.com/wger-project/wger/issues/325
.. _#330: https://github.com/wger-project/wger/issues/330
.. _#331: https://github.com/wger-project/wger/issues/331
.. _#334: https://github.com/wger-project/wger/issues/334
.. _#350: https://github.com/wger-project/wger/issues/350
.. _@petervanderdoes: https://github.com/petervanderdoes
.. _@DeveloperMal: https://github.com/DeveloperMal
.. _@alelevinas: https://github.com/alelevinas
.. _@jstoebel: https://github.com/jstoebel
.. _@alokhan: https://github.com/alokhan
.. _@w00p: https://github.com/w00p
.. _@andela-bmwenda: https://github.com/andela-bmwenda
.. _@RyanSept: https://github.com/RyanSept
.. _@pythonGeek: https://github.com/pythonGeek



1.7
---
**2016-02-28**

New translations:

* Czech (many thanks to Tomáš Z.!)
* Swedish (many thanks to ywecur!)


New features:

* Workout PDF can now print the exercises' images and comments `#261`_
* Allow login with username or email (thanks `@warchildmd`_) #164`_
* Correctly use user weight when calculating nutritional plans' calories (thanks `@r-hughes`_) `#210`_
* Fix problem with datepicker `#192`_
* Order of exercises in supersets is not reverted anymore `#229`_
* Improvements to the gym management:

  * Allow to add contracts to members
  * Visual consistency for lists and actions
  * Vastly reduce the number of database queries in gym member list `#144`_
  * Global list of users for installation `#212`_
  * Allow administrators to restrict user registration `#220`_
  * Refactored and improved code, among others `#208`_
  * Allow gym managers to reset a member's password `#186`_

* Better rendering of some form elements `#244`_
* Improved GUI consistency `#149`_
* Docker images for easier installation `#181`_
* Use hostname for submitted exercises (thanks `@jamessimas`_) `#159`_
* Download js libraries with bowerjs (thanks `@tranbenny`_) `#126`_
* Improved and more flexible management commands `#184`_
* Fixed error when importin weight entries from CSV (thanks `@r-hughes`_) `#204`_
* Fixed problems when building and installing the application on Windows (thanks `@romansp`_) `#197`_
* Fixed potential Denial Of Service attack (thanks `@r-hughes`_) `#238`_
* Dummy data generator can not create nutrition plans (thanks `@cthare`_) `#241`_


Other improvements and bugfixes: `#279`_, `#275`_, `#270`_, `#258`_, `#257`_


.. _#126: https://github.com/wger-project/wger/issues/126
.. _#144: https://github.com/wger-project/wger/issues/144
.. _#149: https://github.com/wger-project/wger/issues/149
.. _#159: https://github.com/wger-project/wger/issues/159
.. _#164: https://github.com/wger-project/wger/issues/164
.. _#181: https://github.com/wger-project/wger/issues/181
.. _#184: https://github.com/wger-project/wger/issues/184
.. _#186: https://github.com/wger-project/wger/issues/186
.. _#192: https://github.com/wger-project/wger/issues/192
.. _#197: https://github.com/wger-project/wger/issues/197
.. _#204: https://github.com/wger-project/wger/issues/204
.. _#208: https://github.com/wger-project/wger/issues/208
.. _#210: https://github.com/wger-project/wger/issues/210
.. _#212: https://github.com/wger-project/wger/issues/212
.. _#229: https://github.com/wger-project/wger/issues/229
.. _#220: https://github.com/wger-project/wger/issues/220
.. _#238: https://github.com/wger-project/wger/issues/238
.. _#241: https://github.com/wger-project/wger/issues/241
.. _#244: https://github.com/wger-project/wger/issues/244
.. _#257: https://github.com/wger-project/wger/issues/257
.. _#258: https://github.com/wger-project/wger/issues/258
.. _#261: https://github.com/wger-project/wger/issues/261
.. _#270: https://github.com/wger-project/wger/issues/270
.. _#275: https://github.com/wger-project/wger/issues/275
.. _#279: https://github.com/wger-project/wger/issues/279
.. _@jamessimas: https://github.com/jamessimas
.. _@r-hughes: https://github.com/r-hughes
.. _@romansp: https://github.com/romansp
.. _@cthare: https://github.com/cthare
.. _@warchildmd: https://github.com/warchildmd
.. _@tranbenny: https://github.com/tranbenny


1.6.1
-----
**2015-07-25**

Bugfix release


1.6
---
**2015-07-25**

New translations:

* Greek (many thanks to Mark Nicolaou!)

New features:

* Save planed weight along with the repetitions `#119`_
* Improvements to the workout calendar `#98`_
* Allow external access to workouts and other pages to allow for sharing `#102`_, `#124`_
* Email reminder to regularly enter (body) weight entries `#115`_
* Allow users to submit corrections to exercises
* Add day detail view in workout calendar `#103`_
* Fix bug where the exercises added to a superset did not remain sorted `#89`_
* Reduce the size of generated HTML code `#125`_
* Allow users to copy shared workouts from others `#127`_
* Added breadbrumbs, to make navigation easier `#101`_
* Add option to delete workout sessions and their logs `#156`_
* Improve installation, development and maintenance documentation `#114`_

Other improvements and bugfixes:
`#99`_, `#100`_, `#106`_, `#108`_, `#110`_, `#117`_, `#118`_, `#128`_, `#131`_,
`#135`_, `#145`_, `#155`_



.. _#89: https://github.com/wger-project/wger/issues/89
.. _#98: https://github.com/wger-project/wger/issues/98
.. _#99: https://github.com/wger-project/wger/issues/99
.. _#100: https://github.com/wger-project/wger/issues/100
.. _#101: https://github.com/wger-project/wger/issues/101
.. _#102: https://github.com/wger-project/wger/issues/102
.. _#103: https://github.com/wger-project/wger/issues/103
.. _#106: https://github.com/wger-project/wger/issues/106
.. _#108: https://github.com/wger-project/wger/issues/108
.. _#110: https://github.com/wger-project/wger/issues/110
.. _#114: https://github.com/wger-project/wger/issues/114
.. _#115: https://github.com/wger-project/wger/issues/115
.. _#117: https://github.com/wger-project/wger/issues/117
.. _#118: https://github.com/wger-project/wger/issues/118
.. _#119: https://github.com/wger-project/wger/issues/119
.. _#124: https://github.com/wger-project/wger/issues/124
.. _#125: https://github.com/wger-project/wger/issues/125
.. _#127: https://github.com/wger-project/wger/issues/127
.. _#128: https://github.com/wger-project/wger/issues/128
.. _#131: https://github.com/wger-project/wger/issues/131
.. _#135: https://github.com/wger-project/wger/issues/135
.. _#145: https://github.com/wger-project/wger/issues/145
.. _#155: https://github.com/wger-project/wger/issues/155
.. _#156: https://github.com/wger-project/wger/issues/156


1.5
---
**2014-12-16**

New Translations:

* Dutch (many thanks to David Machiels!)
* Portuguese (many thanks to Jefferson Campos!) `#97`_


New features:

* Add support for gym management `#85`_

  * Gym managers can create and manage gyms
  * Trainers can see the gym's users and their routines

* Reduce the amount of CSS and JS libraries by using bootstrap as much as possible `#73`_
* Improvements to the REST API `#75`_

  * Add read-write access
  * Add live browsing of the API with django rest framework
  * Improve documentation
  * /api/v1 is marked deprecated

* Show exercise pictures in workout as well
* Detailed view of exercises and workouts in schedule `#86`_
* Support for both metric (kg) and imperial (lb) weight units `#105`_
* Allow the user to delete his account and data `#84`_
* Add contact field to feedback form
* Cleanup translation strings `#94`_
* Python 3 compatibility! `#68`_

Other improvements and bugfixes:
`#51`_, `#76`_, `#80`_, `#81`_, `#82`_, `#91`_, `#92`_, `#95`_, `#96`_


.. _#51: https://github.com/wger-project/wger/issues/51
.. _#68: https://github.com/wger-project/wger/issues/68
.. _#73: https://github.com/wger-project/wger/issues/73
.. _#75: https://github.com/wger-project/wger/issues/75
.. _#76: https://github.com/wger-project/wger/issues/76
.. _#80: https://github.com/wger-project/wger/issues/80
.. _#81: https://github.com/wger-project/wger/issues/81
.. _#82: https://github.com/wger-project/wger/issues/82
.. _#84: https://github.com/wger-project/wger/issues/84
.. _#85: https://github.com/wger-project/wger/issues/85
.. _#86: https://github.com/wger-project/wger/issues/86
.. _#91: https://github.com/wger-project/wger/issues/91
.. _#92: https://github.com/wger-project/wger/issues/92
.. _#94: https://github.com/wger-project/wger/issues/94
.. _#95: https://github.com/wger-project/wger/issues/95
.. _#96: https://github.com/wger-project/wger/issues/96
.. _#97: https://github.com/wger-project/wger/issues/97
.. _#105: https://github.com/wger-project/wger/issues/105


1.4
---

**2014-03-08**

New features and bugfixes:

  * Calendar view to more easily check workout logs
  * Add "gym mode" with timer to log the workout while at the gym
  * Add automatic email reminders for new workouts
  * New iCal export to add workouts and schedules e.g. to google calendar
  * New exercise overview, grouped by equipment
  * Add possibility to write comments and rate the workout
  * Simplify form for new exercises
  * Alternative PDF export of workout without table for entering logs
  * Unified way of specifying license of submitted content (exercises, etc.)



1.3
---

**2013-11-27**


New translations:

  * Bulgarian (many thanks to Lyuboslav Petrov!)
  * Russian (many thanks to Inna!)
  * Spanish

New features and bugfixes:

  * Mobile version of website
  * Add images to the exercises
  * Exercises now can list needed equipment (barbell, etc.)
  * BMI calculator
  * Daily calories calculator
  * New management utility for languages
  * Improved performance
  * RESTful API



1.2
---
**2013-05-19**

New features and bugfixes:

  * Added scheduling option for workouts.
  * Open all parts of website to all users, this is done by a custom middleware
  * Regular users can submit exercises and ingredients to be included in the general list
  * Add more 'human' units to ingredients like '1 cup' or '1 slice'
  * Add nutritional values calculator on the ingredient detail page
  * Several bugfixes
  * Usability improvements


1.1.1
-----
**2013-03-06**


New features and bugfixes:

  * Pin version of app django_browserid due to API changes in 0.8
  * Fix issue with tabs on exercise overview due to API changes in JQuery


1.1
---
**2013-02-23**

New features and bugfixes:

  * Better navigation bar
  * Added descriptions for the exercises (German)
  * New workout logbook, to keep track of your improvements
  * Import your weight logs from a spreadsheet (CSV-Import)
  * Better filtering for weight chart
  * Muscle overview with corresponding exercises
  * Add guest accounts by generating a temporary user
  * Description pages about the software
  * Easier installation process


1.0.3
-----
**2012-11-19**


New features and bugfixes:

  * Add option to copy (duplicate) workouts and nutritional plans
  * Login without an account with Mozilla's Persona (BrowserID)
  * Better AJAX handling of the modal dialogs, fewer page reloads and redirects
  * Expand the list of ingredients in German
  * Add pagination to the ingredient list
  * Improvements to user page:

    * Add a "reset password" link to the login page
    * Email is now user-editable

  * More natural lines in weight chart with cubic interpolation


1.0.2
-----
**2012-11-02**

Bugfix release


1.0.1
-----
**2012-11-02**


New features and bugfixes:

  * Fix issue with password change
  * Small improvements to UI
  * Categories editable/deletable from the exercise overview page
  * Exercise AJAX search groups by category
  * More tests!
  * Use generic views for editing, creating and deleting objects


1.0
---
**2012-10-16**

Initial release.

New features and bugfixes:

  * Workout manager
  * PDF output for logging progress
  * Initial data with the most popular exercises
  * Simple weight chart
  * Nutrition plan manager
  * Simple PDF output
  * Initial data with nutritional values from the USDA
