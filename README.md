# MobileGestalt

Get iOS device UDID with public API

## How it works?

The library use \*.mobileconfig file to get device information. You can read the [documents](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/iPhoneOTAConfiguration/ConfigurationProfileExamples/ConfigurationProfileExamples.html) by Apple to learn about.

## What can we get?

- UDID
- IMEI
- ICCID (I can't got it but write in the documents by Apple)
- Products (Like: **iPhone8,3**)
- Version (Like: **14G60**)

## Can use in App Store?

> Sorry, I don't know.

## USAGE - QUICK

### 1. Install

1. Use CocoaPods `pod 'MobileGestalt'`
2. Use Source, drag _MobileGestalt_ to your project

Import

```objc
#import <MobileGestalt/MobileGestalt.h>
```

### 2. Add URL Scheme

Add an unique URLScheme to your _Info.plist_.

Such as: `mobilegestalt`

### 3. Create a session

```objc
MGSessionConfiguration *configuration = [MGSessionConfiguration defaultConfiguration];
configuration.port = 10418;
configuration.portOffset = 3;	// Use port 10418~10421

configuration.port = 0; //	Use random port
self.session = [MGSession sessionWithConfiguration:configuration];
```

### 4. Create a request

```objc
//	Create a custom request
MGRequest *request = [MGRequest request];
request.attributes = @[MGAttributeUDID, MGAttributeIMEI, MGAttributeICCID, MGAttributeVersion, MGAttributeProduct];
request.displayName = @"Title for Profile";
request.organization = @"Subtitle for Profile";
request.explain = @"Description for Profile";
request.identifier = @"com.unique.mobilegestalt";

//	Create a signed request in remote
MGRequest *request = [MGRequest requestWithMobileConfigURL:aRemoteURL];

//	Create a signed request in local
MGRequest *request = [MGRequest requestWithMobileConfigData:aNSData];

```

### 5. Send request

```objc
[self.session request:request completed:^(MGRequest *request, MGResponse *response, NSError *error) {
	if (error) {
		NSLog(@"%@", error);
	} else {
		NSLog(@"%@", response.data);
	}
}];
```

## SIGNED

https://github.com/JeremyAgost/Hancock/releases/tag/v1.2.1

use Hancock.1.2.1 to signed MDM config

# Create MDM file follow

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
	<dict>
		<key>PayloadContent</key>
		<dict>
			<key>URL</key>
			<string>http://127.0.0.1:10418/MobileGestalt/register</string> // url will be send after install provision on phone. consider change 127.0.0.1 to our host
			<key>DeviceAttributes</key>
			<array>
				<string>UDID</string>
				<string>IMEI</string>
				<string>ICCID</string>
				<string>VERSION</string>
				<string>PRODUCT</string>
			</array>
		</dict>
		<key>PayloadOrganization</key>
		<string>Mobile Gestalt</string>
		<key>PayloadDisplayName</key>
		<string>Mobile Gestalt Demo</string>
		<key>PayloadVersion</key>
		<integer>1</integer>
		<key>PayloadUUID</key>
		<string>3C4DC7D2-E475-3375-489C-0BB8D737A653</string>
		<key>PayloadIdentifier</key>
		<string>com.unique.mobilegestalt.register</string>
		<key>PayloadDescription</key>
		<string>Get device information</string>
		<key>PayloadType</key>
		<string>Profile Service</string>
	</dict>
</plist>

```

Host:
. GCDWebServer to create local host -> can use apache or nodeexpress

. requestWithMobileConfigURL => response data of signed MDM
http://127.0.0.1:10418/MobileGestalt//MobileGestalt/mdm.mobileconfig

. after install mobile profile will request
http://127.0.0.1:10418/MobileGestalt/register
Post data will contain plist file with payload => parser and send callback scheme to mobile app
