# Firebase push notifications

We used to send push notifications directly to APNS or GCM from our code. We were maintaining device_ids, OS types, GCM/APNS certificates and sometimes this was really tricky.

We knew that an easier solution exist and then we discovered the Firebase. Firebase is a service which does all hard work for us. There is no certificates at server side and we don't need to maintain OS types, because the Firebase knows which notification goes to the APNS and which to the GCM. Basically, Firebase handles the routing and delivery of push notifications to targeted devices. Firebase also has a dashboard with the statistics and mobile developers can use this dashboard for testing purposes.

There are many services which provide us with push notifications, but we are using Firebase because of its simplicity and reliability.

## General
In this chapter we will explain a few approaches for push notifications implementations and theirs PROS and CONS.

It is possible to send push notification to only one device, group of devices or to topics.

Before development, read chapters about [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/), and [app server development](https://firebase.google.com/docs/cloud-messaging/server).

## Development Approaches
There are three approaches for using Firebase:

  * storing one device id per user
  * storing multiple device ids per user
  * topics
    * With persisted topics
    * With non persisted topics


1. Approach based on one device id per user
 * Using this approach means adding a new column `device_id` directly to a User model.
 * On each login this column will be overwritten.
 * On each logout this column should be deleted.

 * PROS
    * Easy to build.

 * CONS
    * User can get notification only on last logged in device.

 * This approach is not recommended because it is normal that users have multiple devices.

2. Approach based on multiple device ids per user

  * A new table to store user device_ids needs to be implemented.
  * Index on device_id is mandatory

  * PROS
    * User can get push notifications at multiple devices.

  * CONS
    * We need to maintain user device_ids(create, delete).

  * Sender service example
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

  * API sessions controller example

    ```ruby
    module Api
      module V1
        class SessionsController < AuthorizedController
          skip_before_action :authenticate_user!, only: [:create]

          version 1

          def create
            # login logic goes here
            add_user_device
            expose current_user
          end

          def destroy
            # logout logic goes here
            device.destroy if device.present?
            expose current_user
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
      end
    end
    ```

3. Approach based on topics

  * Topic is a named group of one or more device_ids stored in Firebase. Topic name can be anything that matches this regular expression: `[a-zA-Z0-9-_.~%]+`

  * Persisted topics
    * In this approach we need a way to create topic names and expose them through the API.
    * If Firebase doesn't have a topic, it will be created on users first subscription.

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
  * Sender service example
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

## Choosing right development approach
Even though every application is different, our recommendation is a topic based approach.

## Setup
1. DevOps has to create new Firebase project.

2. Server key from Firebase project settings should be copied to Vault. Server key authorizes your app server on Google services.

3. Add [fcm gem](https://github.com/spacialdb/fcm) to your project. The Fcm gem serves as an SDK for communicating with the [Firebase](https://firebase.google.com/docs/cloud-messaging/send-message).
