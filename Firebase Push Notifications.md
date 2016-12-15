# Firebase push notifications

Table of contents:

1. [General](#general)
2. [Setup](#setup)
3. [Development approaches](#development-approaches)
4. [Choosing right development approach](#choosing-development-approach)

## General
In this chapter we won't talk about specific implementation because everything is well described in Firebase documentation. We will explain few ideas for push notifications implementations and theirs PROS and CONS.

There are many solutions which are providing us push notifications. We are using Firebase because of simplicity, reliability and it's free.

Firebase Cloud Messaging handles the routing and delivery to targeted devices.

Largest size of one message is 4KB.

Google provides connection servers for HTTP and XMPP protocols but we are using HTTP.
It is possible to send push notification to only one device, group of devices and topics.

Before development, read chapters about [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/), and [app server development](https://firebase.google.com/docs/cloud-messaging/server).

## Setup
1. DevOps has to create new Firebase project.

2. Server key from Firebase project settings should be copied to Vault. Server key authorizes your app server for access to Google services. Don't include the server key anywhere in your client code.

3. For development we are using [fcm gem](https://github.com/spacialdb/fcm). Fcm gem has methods for sending requests to [Firebase](https://firebase.google.com/docs/cloud-messaging/send-message).

4. Choose one development approach and you can start developing.

## Development Approaches

Before development starts we need to know which approaches exist.

We will explain three approaches:
  * Approach based on one device id
  * Approach based on multiple device ids
  * Approach based on topics
    * With persisted topic names in our DB
    * With non persisted topic names in our DB


1. Approach based on one device id
 * If user can have only one device, device_id can be stored in User model. When user installs mobile application he will get device_id from Firebase.

 * Create API endpoint at which mobile team can send device_id after login. After every login they have to send you device_id because user can login in different device.

 * Create API endpoint at which mobile team can delete device_id after logout. After every logout from device, mobile team will delete device_id from User through API.

 * CONS
    * User can get notification only at last logged in device.

    * If user A login through his own phone, and then user A login and logout through user B phone(because user A left phone at home). All push notifications sent to user A in between will not be received because user A hasn't device_id in DB and user A has to login again through his phone.

  * PROS
    * Easy to build.

2. Approach based on multiple device ids

  * We need 1:N relationships between User and UserDevice models.

  * Create same API endpoints for creating and deleting device_ids like in previous example.
  * PROS
    * User can get push notifications at multiple devices.
  * CONS
    * We need to maintain user device_ids(create, delete).

3. Approach based on topics

  * Topic is group of one or more device_ids saved in Firebase by specific name. Name can be anything that matches the regular expression: `[a-zA-Z0-9-_.~%]+`

  * Imagine you are building app like Reddit.


  * Persisted topics
    * In this approach we have stored topic names in DB. For example in every subreddit record.
    * We can have one topic for every subreddit.
    * We can't create new topic from server side and because of that mobile client has to create topics. They can do this explicit after subreddit creation, or implicit after subscription. (Thats how Firebase works when topic doesn't exist. If topic doesn't exist in Firebase, it will be created and user will be subscribed after creation)
    * At every login, mobile app fetches subscribed subreddits from our API and they subscribe user to topic/topics on Firebase.
    * At every logout mobile app unsubscribe user(only current device) from topic/topics on Firebase.
    * CONS
      * We are duplicating topic names in our DB and Firebase.


  * Non Persisted topics
    * We can make convention for topic names with mobile developers.
    * Every topic name is parameterized subreddit name.
    (Firebase Push Notifications -> firebase-push-notifications)
    * PROS:
      * With this approach we don't need to store topic names in DB.
      * We don't need topic_id field in DB because mobile team can get them through subscribed subreddits names.
      * Easy to build.
    * CONS:
      * It doesn't seems that Firebase has topic limits but maybe in future they will restrict this like Amazon SNS.

## Choosing right development approach
When we are choosing between development approaches there are few questions for which we need answer. If we know answer at all questions, then we can choose development approach which is best for our application.

1. How many devices user will be using?
  * One device: Any approach
  * Multiple devices: Second or third approach
2. Are notifications grouped by any rules in our application or we always need to send notification to one specific user or to all users together?
  * It depends on number of user devices:
    * One device: Any approach
    * Multiple devices: Second or third approach
3. Do we need control for every users device and currently active device or we are ok with sending notification to all user devices.
 * Second approach is best for full control push notification systems.

Note: Every application is different and please choose carefully.
