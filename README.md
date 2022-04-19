# SMS Import / Export

SMS Import / Export is a simple Android app that imports and exports SMS and MMS messages and call logs from and to JSON files. Root is not required.

[<img src="https://fdroid.gitlab.io/artwork/badge/get-it-on.png"
     alt="Get it on F-Droid"
     height="80">](https://f-droid.org/packages/com.github.tmo1.sms_ie/)

## Installation

SMS Import / Export is available from [Github](https://github.com/tmo1/sms-ie). The repository can be cloned and built locally, from the command line (e.g., by issuing `gradlew assembleDebug` in the root directory of the project) or within Android Studio. Prebuilt APK packages can be downloaded from the [Releases page](https://github.com/tmo1/sms-ie/releases).

## Usage

 - Export messages: Click the `Export Messages` button, then select an export destination.
 - Import messages: Click the `Import Messages` button, then select an import source.
 - Export call log: Click the `Export Call Log` button, then select an export destination.
 - Import call log: Click the `Import Call Log` button, then select an import source.

These operations may take some time for large numbers of messages or calls. The app will report the total number of SMS and MMS messages or calls exported or imported, and the elapsed time, upon successful conclusion.

By default, binary MMS data (such as images and videos) are exported. The user can choose to exclude them, which will often result in a file that is much smaller and more easily browsable by humans. (The setting is currently ignored on import.)

### Permissions

To export messages, SMS Import / Export must be granted permission to read SMSs and Contacts (the need for the latter is explained below). The app will ask for these permissions on startup, if it does not already have them.

To import messages, SMS Import / Export must be the default messaging app. This is due to [an Android design decision](https://android-developers.googleblog.com/2013/10/getting-your-sms-apps-ready-for-kitkat.html).

**Warning:** While an app is the default messaging app, it takes full responsibility for handling incoming SMS and MMS messages, and if does not store them, they will be lost. SMS Import / Export ignores incoming messages, so in order to avoid losing such messages, the device it is running on should be disconnected from the network (by putting it into airplane mode, or similar means) before the app is made the default messaging app, and only reconnected to the network after a proper messaging app is made the default.

To export call logs, SMS Import / Export must be granted permission to read Call Logs and Contacts (the need for the latter is explained below). Currently, the app does not ask for the former permission, and it must be granted by the user on his own initiative.

To import call logs, SMS Import / Export must be granted permission to read and write Call Logs.

### Contacts

SMS and MMS messages include phone numbers ("addresses") but not the names of the communicating parties. The contact information displayed by Android is generated by cross-referencing phone numbers with the device's Contacts database. When exporting messages, SMS Import / Export does this cross-referencing in order to include the contact names in its output; this is why permission to read Contacts in necessary. When importing, included contact names are ignored, since the app is (at least currently) not in the business of adding to or modifying the Android Contacts database. The best way to maintain the association of messages with contacts is to simply use Android's built in Contacts export / import functionality to transfer contacts to the device into which SMS Import / Export is importing messages. Contacts cross-referencing is performed for call log export as well, despite the fact that call log metadata will often already include the contact name; see below for a discussion of this point.

## JSON Structure

Following is the structure of the JSON currently exported by SMS Import / Export; this is subject to change in future versions of the app.

### Messages

The exported JSON is an array of JSON objects representing messages, SMSs followed by MMSs. Each JSON message object contains a series of tag-value pairs taken directly from Android's internal message data / metadata structures, documented in the Android API Reference: [SMS](https://developer.android.com/reference/android/provider/Telephony.TextBasedSmsColumns), [MMS](https://developer.android.com/reference/android/provider/Telephony.BaseMmsColumns). In addition, SMS Import / Export adds some other tag-value pairs and child JSON objects, as described below.

#### SMS Messages

In SMS messages, the value of `type` specifies (among other things) the direction of the message: the two most common values are `1`, denoting "inbox" (i.e., received), and `2`, denoting "sent".

SMS messages contain a single `address` tag; depending on the message direction, this is either the sender or receiver address. SMS Import / Export attempts to look up the address in the Android Contacts database. If this is successful, a tag-value pair of the form `"display_name": "Alice"` is added to the SMS message object.

#### MMS Messages

MMS message objects have the following additions to the tag-value pairs of their internal Android MMS representation:

 - A tag-value pair of the form `"sender_address": { ... }`
 
 - A tag-value pair of the form `"recipient_addresses": [ { ... }, { ... } ]`. The child JSON objects associated with `sender_address` and `recipient_addresses` contain a series of tag-value pairs taken directly from Android's internal MMS address structure, documented [here](https://developer.android.com/reference/android/provider/Telephony.Mms.Addr), plus possibly a single added tag-value pair of the form `"display_name": "Alice"`, as with SMS messages.
 
 - A tag-value pair of the form `"parts": [ { ... }, { ... }]`, where the child JSON objects contain a series of tag-value pairs taken directly from Android's internal MMS part structure, documented [here](https://developer.android.com/reference/android/provider/Telephony.Mms.Part), plus, for parts containing binary data (assuming binary data inclusion is checked), a tag-value pair of the form `"binary_data": "<Base64 encoded binary data>"`.

### Call Logs

The exported JSON is an array of JSON objects representing calls. Each JSON call object contains a series of tag-value pairs taken directly from Android's internal call metadata structures, documented in the [Android API Reference](https://developer.android.com/reference/android/provider/CallLog.Calls). In addition, SMS Import / Export will try to add a `display-name` tag, as with SMS and MMS messages. The call logs may already have a `CACHED_NAME` (`name`) field, but the app will still try to add a `display-name`, since [the documentation of the `CACHED_NAME` field](https://developer.android.com/reference/android/provider/CallLog.Calls#CACHED_NAME) states:

> The cached name associated with the phone number, if it exists.
>
> This value is typically filled in by the dialer app for the caching purpose, so it's not guaranteed to be present, and may not be current if the contact information associated with this number has changed.

## Limitations

Currently, no deduplication is done. For example, if messages are exported and then immediately reimported, the device will then contain two copies of every message.

## sms-db

SMS Import / Export is a sibling project to [sms-db](https://github.com/tmo1/sms-db), a Linux tool to build an SQLite database out of collections of SMS and MMS messages in various formats. sms-db can import JSON files created by SMS Import / Export, and it can export its database to JSON files that can be imported by SMS Import / Export.

## Contributors

The primary author of SMS Import / Export is [Thomas More](https://github.com/tmo1). The following individuals have contributed to the app:

 - [sxwxs](https://github.com/sxwxs): call log export support
 - [vbh (Bindu)](https://github.com/vbh): call log import support
 - [nautilusx](https://github.com/nautilusx): initial German translation
 - [AntoninCurtit](https://github.com/AntoninCurtit): French translation (and assistance with the German one)
 - [baitmooth](https://github.com/baitmooth): additions to German translation
 
## Background

Coming from a procedural, command line interface, synchronous, Linux, Perl and Python background, the development of SMS Import / Export served as a crash course in object-oriented, graphical user interface, asynchronous, Android, Kotlin programming, and consequently entailed a fair amount of amateurishness and cargo cult programming. After much work and learning, however, the app does seem to function correctly and effectively.

## Bugs, Feature Requests, and Other Issues

Bugs, feature requests, and other issues can be filed at [the SMS Import / Export issue tracker](https://github.com/tmo1/sms-ie/issues). When reporting any problem with the app, please specify the version of the app used for export and / or import, as applicable. When reporting a crash, particularly a reproducible one, please attach a logcat. Instructions for doing so (with increasing level of detail) can be found [here](https://wiki.lineageos.org/how-to/logcat), [here](https://f-droid.org/en/docs/Getting_logcat_messages_after_crash/), and [here](https://www.xda-developers.com/guide-sending-a-logcat-to-help-debug-your-favorite-app/).

## License

SMS Import / Export is free / open source software, released under the terms of the [GNU GPLv3](https://choosealicense.com/licenses/gpl-3.0/)
