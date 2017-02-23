# Firebase push notifications

Table of contents:

1. [General](#general)
2. [Setup](#setup)
3. [Development approaches](#development-approaches)
4. [Choosing right development approach](#choosing-development-approach)

## General
In this chapter we will explain few approaches for push notifications implementations and theirs PROS and CONS.

There are many services which provides us with push notifications, but we are using Firebase because of its simplicity, reliability and because it's free.

Firebase Cloud Messaging handles the routing and delivery of push notifications to targeted devices.

Largest size of one message is 4KB.

It is possible to send push notification to only one device, group of devices or to topics.

Before development, read chapters about [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/), and [app server development](https://firebase.google.com/docs/cloud-messaging/server).

## Setup
1. DevOps has to create new Firebase project.

2. Server key from Firebase project settings should be copied to Vault. Server key authorizes your app server on Google services.

3. Add [fcm gem](https://github.com/spacialdb/fcm) to your project. Fcm gem has methods for sending requests to [Firebase](https://firebase.google.com/docs/cloud-messaging/send-message).

## Development Approaches
There are three approaches for using Firebase:

  * one device id
  * multiple device ids
  * topics
    * With persisted topics
    * With non persisted topics


1. Approach based on one device id
 * Using this approach means adding a new column directly to a User model.
 * On each login this column will be overwritten.
 * This approach is not recommended.

 * PROS
    * Easy to build.

 * CONS
    * User can get notification only on last logged in device.

2. Approach based on multiple device ids

  * A new table to store user device_ids needs to be implemented.

  * PROS
    * User can get push notifications at multiple devices.

  * CONS
    * We need to maintain user device_ids(create, delete).

3. Approach based on topics

  * Topic is a named group of one or more device_ids stored in Firebase. Topic name can be anything that matches this regular expression: `[a-zA-Z0-9-_.~%]+`

  * Persisted topics
    * In this approach we need a way to create topic names and expose them through the API.
    * If Firebase doesn't have a topic, it will be created on users first subscription.
    * It is mobile app job to handle topic subscriptions.

    * CONS
      * Topic names are being duplicated in Firebase as well as our backend.

  * Non Persisted topics
    * We can agree on a convention for topic names with mobile developers.
    * Every topic name is parameterized. (Firebase Push Notifications -> firebase-push-notifications)

    * PROS:
      * With this approach we don't need to store topic names in backend.
      * Easy to build.

    * CONS:
      * It doesn't seems that Firebase has limits on topics, but maybe in future they will restrict this like Amazon SNS does.

## Choosing right development approach
Even though every application is different, our recommendation is a topic based approach.

## Examples
* [Code examples](https://gist.github.com/fkuster/9cc5e19a59dc5bf11a35acc383ada01e)
