App Engine application for the Udacity Fullstack Web Developer Requirements.
Project 4.

Andrew Moore
2016/02/25

## Products
- [App Engine][1]

## Language
- [Python][2]

## APIs
- [Google Cloud Endpoints][3]

## Description

This application when deployed using Google App Engine, allows the user to
securely log-in to create conference listings, conference session information,
and independent speaker information across conferences with a variety of
functions. Information is stored on Google Datastore.

At the moment, only the back-end operations are fully functional (eg: using
apis-explorer console), with limited front-end capabilities.

These operations can be accessed at:
- [https://apis-explorer.appspot.com/apis-explorer/?base=https://**name_of_conference_app**/_ah/api#p/] [4]

Functions include:

### Profile entities:
- A user may see information regarding their log-in profle (don't forget to
register your t-shirt size!). Authentication is through Oauth2 and Google.
- The following endpoints are available regarding profiles:

    getProfile

    saveProfile

### Conference entities:
- As a parent user, one can create and update conferences, with information regarding: name, city, select topics, start and end dates, maximum number of registrants and seats available.
- One can easily return information about the available conferences.
- As a logged-in user, the ability to register and deregister from 
conferences (and to see your list of registrations).
- The ability to query conferences regarding month, city and topic.
- The following endpoints are available regarding conferences:

    createConference

    getConference

    getConferencesCreated (eg, per logged-in user)

    queryConferences

    registerForConference

    unregisterFromConference

    updateConference

- An app engine cron job exists notifying users of conferences with fewer than 6 seats available. An announcement is sent to memcache regarding the limited availability. This end result can be manipulated using:
    
    getAnnouncement
    
    putAnnouncement


### Session entities:
- Sessions for each conference can only be created by the user parent of the 
conference itself and are defined as children entities to the parent 
conference.
- Session information includes: conference name, start time, duration, location, type of session (Lecture, Workshop, Symposium or NOT_SPECIFIED), 
session highlights, seats remaining for the session, and speaker information,
including speaker Last name for easy reference, in addition to keyed speaker
information for cross-reference.
- Multiple sessions can take place at the same time within a conference,
but given the available session properties, limits could easily be added 
(eg, with regard to location, speaker, or time constraints).
- The following endpoints are available regarding sessions:
    
    createSession
    
    getAvailableSessions (eg, return sessions with seats still available)**
    
    getConferenceSessions
    
    getConferenceSessionsByType (eg, limited to the four defined types)
    
    getConferenceSessionsExcludingType*
    
    getSession
    
    getSessionsByDate (eg, return sessions for a date, conference agnostic)**
    
    getSessionsBySpeaker
 
 - An app engine task is created upon session creation that notifies users
 (through an announcement in memcache) if the speaker of the most recent
 session is also giving any other talks at that conference. This end
 result can be seen using:
    
    getFeaturedSpeaker

- In addition a user may add or remove sessions from their wishlist (this
does not change the session availibility, although registration could easily
be implemented.)

    addSessionToWishlist

    deleteSessionInWishlist


* this endpoint solution has been created to minimize multiple inequality
queries using different properties (only one property may entertain an inequality query on app-engine and this query must also be the first query in
a series of queries) and relies on the fact that there are a limited number of
session types (including NON_SPECIFIED). The code provides a solution for selecting by type (in essence, eliminating the exclusion type using python tuples.). An example of a time inequality query is given in the comments as per project suggestions.

```
# use this space to filter all sessions using an inequality query
# eg, all sessions before 19:00
# sessionsAll = sessionsAll.filter(
#      Session.startTime < datetime(1970, 1, 1, 19, 0, 0, 0).time())

# now find the remaining sessions whose type you wish to exclude

sessionsLess = sessionsAll.filter(Session.typeOfSession==sessionType)

# using this information, remove these session from your query
sessions = tuple(x for x in sessionsAll if x not in sessionsLess)

```

** these are two additional queries provided as per project requirements.

### Speaker entities

- Speaker entities are defined as first and last names, with a websafe speaker
key passed to session creation and different queries.
- Speaker entities are created independently, and do not have a parent entity,
so that they are available across conferences and users, although a user
still needs to be logged-in to access this data.
- Many additional properties (eg, phone, email, etc) could be added to increase
the utility of this entity.
- The following endpoints are available regarding speakers:

    addSpeaker

    getSpeaker


### Setup Instructions for personalization

1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][5].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][6].)
1. (Optional) Generate your client library(ies) with [the endpoints tool][7].
1. Deploy your application.


[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://apis-explorer.appspot.com/apis-explorer/?base=https://conference-app-1222.appspot.com/_ah/api#p/
[5]: https://console.developers.google.com/
[6]: https://localhost:8080/
[7]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool
