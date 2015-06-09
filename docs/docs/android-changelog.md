[block:api-header]
{
  "type": "basic",
  "title": "Versions"
}
[/block]
## 2.2.0
  - Just a new release of the previous 1.3.5 version

## 1.3.5

**Features**

  The following device properties were made public for debug reasons:
  - device_id = TQDeviceInfo.getUdid()
  - tracking_id = TQGeotrigger.getTrackingId()
  - device_token = TQPush.getDeviceToken()
  - push_enabled = Boolean.valueOf(TQDeviceInfo.pushEnabled)
  - geopush_enabled = TQ.shared().checkTrackingState()
  - last_location_update = TQGeotrigger.getLastLocationUpdate()

## 1.3.4

**Bugfixes**
  - Most of the time the user's city was previously been sent to the API as 'unknown'

## 1.3.3

**Features**
 - Available new method `addCustomContent` to send any custom Map with user information to the API.

## 1.3.2
 
**Bugfixes**
  - In order to support applications with previous versions of the SDK, the implementation of error messages to the user is not *mandatory*, but still *recommended*.

## 1.3.1

**Features**
  - Interface `TQUserDialog` added and it's methods are called when the user's device current settings don't meet some of the requirements for the Geotrigger Service. 

## 1.3.0

Base version of the SDK.