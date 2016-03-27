Conference Central App
======================
This project to extend the existing Conference Central application as per the given requirements is part of the Udacity Full Stack Nanodegree course. It is built upon the Google App Engine in Python.

Google App Engine App Name
--------------------------
conference-app-117522

Requirements
------------
- Python: `2.7.x`
- Google App Engine: `1.x`


Task 1: Add Sessions to a Conference
------------------------------------
For this task, 2 new classes were added - Session (NDB model) and SessionForm (ProtoRPC message).

The Session class is designed to have an ancestor relationship with its parent Conference object. This allows us to easily get the parent Conference object given a Session object.


Task 2: Add Sessions to User Wishlist
--------------------------------------
Couple of changes were made in the models for implementing these session wishlist API endpoints:

The websafeKey property was added to the SessionForm class to be able to get the datastore key for each Session which will need to be passed in to the api endpoints addSessionToWishlist(SessionKey) and
deleteSessionInWishlist(SessionKey)

A new repeated property named sessionKeysInWishlist was added to the Profile class to hold the list of all the session keys for session that were added to their wishlist.


Task 3: Work on indexes and queries
------------------------------------

Following 2 additional queries and its corresponding API endpoints were added:

1. **getWishlistSessionsBySpeaker(speaker):**  This gets the sessions in the current user's wishlist filtered by a specified speaker. This query could be useful get all sessions in the user's wishlist by the featured speaker for example. It was implemented by filtering the sessions in the wishlist by the specified speaker.

2. **getWishlistSessionsByType(typeofSession):** This gets the sessions in the current user's wishlist filtered by its type. This query could be userful for user to quickly see all the sessions they want to attend of a given type (e.g. workshop, forum, etc.) across all conferences. It was implemented by filtering the sessions in the wishlist by the specified type.

**Query Problem - All non-workshop sessions before 7 pm**

The problem with this query is that the Google Datastore does not allow inequality comparisons (<, <=, >, >=, !=) on more than one property in a single query. For this query, we have to apply a `NOT EQUAL TO` filter for the session type property and a `LESS THAN` filter for the session start time property, which is not allowed.

So there are 2 ways to implement this:

**Implementation 1:**
Change the `NOT EQUAL TO` filter for non-workshop sessions to an `IN` filter on the session types that are not workshop. The advantage of this is that this would perform both the filters in the query itself and only return the final limited results for Python/application code to process, hence making it more efficient and scalable, esp. when there are lots of sessions before 7pm. This is the solution that I have implemented.

**Implemenation 2:**
The other way is to apply only one inequality filter and then filter out the unwanted results in Python/application code. The advantage of this is that we do not have to worry about adding the new SessionType to the query, in case there is ever a new Session Type that gets added to the system. But the disadvantage is obviously that if the results returned by the just one filter are too many then Python/application code has to filter out a lot of results on the application side which is not very efficient. I implemented this approach as well but didn't decide to use it (I have the code commented out).

It is about tradeoff between these 2 approaches, and I chose #1 above because I believe that chances of there being many new SessionTypes after it is defined once is very very low, but changes of the number of sessions growing in the system is highly likely depending on how big a given conference is.


Task 4: Add a Task
-------------------
For this task, the `getFeaturedSpeaker()` endpoint method was added to get the current featured speaker. Everytime a new session is created and the speaker for that session has more than one sessions, that speaker is stored as a featured speaker in the memcache. Previous featured speaker would be overwritten. Also, the memcache entry is added via a background task implemented using the task queue feature in Google App Engine.


How to run and test the App
----------------------------
To start the app on localhost, run the Google App Engine Launcher, then add the Google App Engine project named `conference-app-117522` in the folder where you have checked out the project. After starting the project, it will get hosted at `http://localhost:<port>/` where the default value for `<port>` is typically `8080`

This project has also been deployed to the [Google App Engine cloud](https://conference-app-117522.appspot.com/):

You must login to the app using your Google account, then go to My Profile page, and update your Tee shirt size. This will ensure that your profile instance gets saved in the datastore.

In order to test the endpoints you can go to the [API explorer](https://apis-explorer.appspot.com/apis-explorer/?base=https://conference-app-117522.appspot.com/_ah/api#p/conference/v1/)

