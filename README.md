This is the fourth project for "Full Stack Web Developer Nanodegree". 

In this project a cloud-based API server is developed to support conference organization application. The API supports the following funcationality: user authentication, user profiles, conference information and various manners in which to query data.

The website deployed on Google Cloud Plateform is : https://vital-keep-123617.appspot.com/
## Products
- [App Engine][1]

## Language
- [Python][2]

## APIs
- [Google Cloud Endpoints][3]

## Setup Instructions
1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host
   your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][4].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][5].)
1. (Optional) Generate your client library(ies) with [the endpoints tool][6].
1. Deploy your application.

This project mainly contains four following tasks.

## Task 1: Add Sessions to a Conference
1. Session and SessionForm are defined in models.py as follows:
   - Session class is created as `ndb.Model` and has the following attributes:
	```python
	class Session(ndb.Model):
		"""Session -- Session object"""
		name = ndb.StringProperty()
		highlights = ndb.StringProperty()
		speaker = ndb.StringProperty(required=True)  
		duration  = ndb.IntegerProperty() 
		typeOfSession  = ndb.StringProperty(repeated=True) 
		date = ndb.DateProperty()
		startTime = ndb.TimeProperty() 
		websafeConferenceKey =  ndb.StringProperty()
	```
   - SessionForm class is created as `messages.Message` and has the corresponding attributes in Session Model but sessi     onSafeKey is added. 	
	```python
	class SessionForm(messages.Message):
		"""SessionForm -- Session outbound form message"""
		name  = messages.StringField(1)
		highlights  = messages.StringField(2)
		speaker = messages.StringField(3)
		duration = messages.IntegerField(4)
		typeOfSession = messages.StringField(5, repeated=True)
		date  = messages.StringField(6) 
		startTime = messages.StringField(7) 
		sessionSafeKey  = messages.StringField(8)
		websafeConferenceKey  = messages.StringField(9)
	```
3. Speaker
	-  Defined as a string attribute in Session classes.
4. The following Endpoints methods are realized to manage sessions:
	- `getConferenceSessions(websafeConferenceKey)` -- Given a conference, return all sessions.
	- `getConferenceSessionsByType(websafeConferenceKey, typeOfSession)` -- Given a conference, return all sessions of a specified type (eg lecture, keynote, workshop)
	- `getSessionsBySpeaker(speaker)` -- Given a speaker, return all sessions given by this particular speaker, across all conferences
	- `createSession(SessionForm, websafeConferenceKey)` -- Open only to the organizer of the conference

## Task 2: Add Sessions to User Wishlist
1. Profile class
	-  Added a new attribute `sessionKeysInWishlist = ndb.StringProperty(repeated=True)` into Profile class.
	-  In `sessionKeysInWishlist`, the `urlsafe` of the sessions which the user is interested in are stored.

2. The following Endpoints methods are realized to manage sessions:
	- `addSessionToWishlist(SessionKey)` -- adds the session to the user's list of sessions they are interested in attending.
	- `getSessionsInWishlist()` -- query for all the sessions in a conference that the user is interested in.
	- `deleteSessionInWishlist()` -- delete sessions in wishlist.

## Task 3: Work on indexes and queries
1. Create indexes
2. Two additional queries are implemented and `ProfileForms` is designed to solve the problems:
	-  `getAttendeesByConference(websafeConferenceKey)` -- Given a conference, return all attendees.
	-  `getAttendeesBySession(sessionSafeKey)` -- Given a session, return all attendees.
3. Solve the following query related problem

	Question: Let’s say that you don't like workshops and you don't like sessions after 7 pm. How would you handle a query for all non-workshop sessions before 7 pm? What is the problem for implementing this query? What ways to solve it did you think of?

	Answer: Since google datastore APIs only support one inequality filter for one property in a query, we can not get the result in only one query since in this question two properties need to be filtered. But we can seperate the query into two by performing twice filterings.

## Task 4: Add a Task
When a new session is added, the speaker is checked if multiple sessions are held by this speaker. A new memcache entry is added with the session names.
1. Instead of adding a one-time task to check the featured speaker into `taskqueue`, this task is set as a `cron task` which is scheduled to check the featured speaker once each a hour.
2. `getFeaturedSpeaker()` method is realized to get the featured speaker.



[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://console.developers.google.com/
[5]: https://localhost:8080/
[6]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool
