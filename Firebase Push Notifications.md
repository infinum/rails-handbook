We used to send push notifications directly to APNS or GCM from our code. We were maintaining device_ids, OS types, and GCM/APNS certificates. This approach was complicated and hard to maintain.

We knew that an easier solution must exist. And then we discovered Firebase.

Firebase is a service that does all the hard work for us. There is no need to store certificates on the server because Firebase handles them for us. Furthermore, we don't need to maintain OS types because Firebase knows which notification goes to APNS and which to GCM. Basically, Firebase handles the routing and delivery of push notifications to targeted devices. Firebase also has a dashboard with statistics which mobile developers can use for testing purposes.

There are many services (such as Amazon SNS) that provide push notifications, but we use Firebase because of its simplicity and reliability.

## General
In this chapter, we will explain a few approaches to implementing push notifications and give the pros and cons of each.

Before starting development, read chapters about [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/) and [app server development](https://firebase.google.com/docs/cloud-messaging/server).

## Development approaches
There are several approaches to sending push notifications using Firebase:
  * sending directly to device_ids
    * storing one device id per user
    * storing multiple device ids per user
  * sending to topics
    * with persisted topics
    * with non-persisted topics
  * sending to device groups

### Approach based on sending notifications directly to device_ids
#### Having one device id per user
* Using this approach means adding a new `device_id` column directly to a user model.
 * This column will be overwritten on each login.
 * This column should be deleted on each logout.

 * PROS
    * Easy to build.

 * CONS
    * The user can get notifications only on the last device they logged in.
    * Mobile developers need to be reminded to do a proper logout API call when the user logs out. They often forget to do this.

 * Usually, a user has multiple devices, so this approach is not recommended.

#### Having multiple device ids per user

  * A new table for storing user device_ids needs to be implemented.
  * An index on device_id is mandatory

  * PROS
    * The user can get push notifications on multiple devices.

  * CONS
    * We need to maintain the user's device_ids (create, delete).

#### API sessions controller example
  ```ruby
    class SessionsController < ApiController

      def create
        # login logic goes here
        add_user_device
        # render user
      end

      def destroy
        # logout logic goes here
        device.destroy if device.present?
        # render user
      end

      private

      def add_user_device
        if device.exists?
          device.update(user: current_user)
        else
          current_user.devices.create(device_id: params[:device_id])
        end
      end

      def device
        @device ||= Device.find_by(device_id: params[:device_id])
      end
    end
  ```

#### Sender service example (works for both approaches using device_ids)
  ```ruby
    module FirebaseCloudMessaging
      class UserNotificationSender

        attr_reader :message, :user_device_ids

        # Firebase works with up to 1000 device_ids per call
        MAX_USER_IDS_PER_CALL = 1000

        def initialize(user_device_ids, message)
          @user_device_ids = user_device_ids
          @message = message
        end

        def call
          user_device_ids.each_slice(MAX_USER_IDS_PER_CALL) do |device_ids|
            fcm_client.send(device_ids, options)
          end
        end

        private

        def options
          {
            priority: 'high',
            data: {
              message: message
            },
            notification: {
              body: message,
              sound: 'default'
            }
          }
        end

        def fcm_client
          @fcm_client ||= FCM.new(Rails.application.secrets.fcm['server_api_key'])
        end
      end
    end
  ```

### Approach based on sending notifications to topics

  * Topic messaging is best suited for content which is often sent to a group of users, for example, all users subscribed to weather information, some subreddits, etc.

  * Based on the publish/subscribe model, FCM topic messaging allows you to send a message to multiple devices that have opted into a particular topic. You compose topic messages as needed, and FCM handles routing and delivering the message reliably to the right devices.

  * Topic messaging supports unlimited topics and subscriptions for each app.

  * Persisted topics
    * With this approach, we need a way to create topic names and expose them through the API.
    * If Firebase doesn't have a topic, it will be created when the user subscribes for the first time.

    * CONS
      * Topic names are being duplicated in Firebase as well as in our backend.

  * Non-persisted topics
    * We can agree on a convention for topic names with mobile developers.
    * Every topic name is parameterized (Firebase Push Notifications -> firebase-push-notifications).

    * PROS:
      * With this approach, we don't need to store topic names in the backend.
      * Easy to build.

    * CONS:
      * Currently, it doesn't seem that Firebase has any limits on topics, but they might restrict this in the future just like Amazon SNS.

#### Sender service example
  ```ruby
    module FirebaseCloudMessaging
      class UserNotificationSender
        attr_reader :message, :topic

        def initialize(topic, message)
          @topic = topic
          @message = message
        end

        def call
          fcm_client.send_to_topic(topic, options)
        end

        private

        def options
          {
            priority: 'high',
            data: {
              message: message
            },
            notification: {
              body: message,
              sound: 'default'
            }
          }
        end

        def fcm_client
          @fcm_client ||= FCM.new(Rails.application.secrets.fcm['server_api_key'])
        end
      end
    end
  ```

  * Firebase recently added [support](https://firebase.google.com/docs/cloud-messaging/admin/manage-topic-subscriptions)
  for managing topic subscriptions via a server. With this feature, we can have full control over subscriptions if necessary.

### Approach based on sending notifications to device groups
  * With device group messaging, you can send a single message to multiple instances of an app running on devices belonging to a group. Typically, a "group" refers to a set of different devices that belong to a single user. All devices in a group share a common notification key, which is the token that FCM uses to fan out messages to all devices in the group.

  * If you need to send messages to multiple devices per user, consider device group messaging for those cases of use.

  * The maximum number of members allowed per a notification key is 20.
  * We haven't used device groups, but you can read more about this [approach]((https://firebase.google.com/docs/cloud-messaging/android/device-group)).

## Choosing the right development approach
Since we are rather new at using Firebase, please talk to someone from the team before choosing a development approach.

## Setup
1. DevOps has to create a new Firebase project.

2. The server key from the Firebase project settings should be copied to Vault. It authorizes your app server on Google services.

3. Add the [FCM gem](https://github.com/spacialdb/fcm) to your project. The FCM gem serves as an SDK for communication with [Firebase](https://firebase.google.com/docs/cloud-messaging/send-message).
