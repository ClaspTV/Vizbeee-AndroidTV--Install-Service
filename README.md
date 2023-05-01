# AndroidTV Second Screen Install (ASSIST) Service

This repository is an open source implementation of the AndroidTV Second Screen Install (ASSIST) Service.

# Overview

Mobile-to-TV discovery, install and launch of streaming apps enables rich cross-device experiences to increase viewer acquisition, engagement and monetization on TVs. The streaming industry has created open protocols and implementations to enable such mobile-to-TV interactions.  

**DIAL**

[DIAL](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGlhbC1tdWx0aXNjcmVlbi5vcmd8ZGlhbHxneDoyNzlmNzY3YWJlMmY1MjZl) protocol is an open protocol that is implemented in some Javascript and other TV platforms. DIAL enables TV app discovery and launch from mobile apps but leaves the TV app install capability as a TV platform specific implementation detail to the TV platform developers.

**ASSIST**

ASSIST is an open protocol for TV app discovery, launch and install on Android TVs. It *assists* existing open protocols like DIAL or proprietary protocols like Chromecast. The key feature of ASSIST is that it enables a secure and privacy-friendly way to initiate Android TV app install from mobile devices as part of casting flows when using DIAL or Chromecast protocols. ASSIST significantly increases the mobile-to-TV interactions by removing the biggest hurdle, i.e., automatic Android TV app install, during initial casting from a mobile app.

# Developer

## Overview

The repository provides the core ASSIST Service and enables it to be built as a (1) System Service and (2) Demo App. The System Service can be easily incorporated into existing Android OS builds (AOSP) on your TV platform. The Demo App enables simple demonstration of the ASSIST feature and also aids in its development with a simpler test cycle.

## Code Structure

The code is organized as:
* assist/ - 
* system-service/ -
* demo-app/ - 

## Demo App

### Notes

The Demo App has been designed for rapid testing of ASSIST service updates without having to build and deploy a System Service. In this mode, 

* ASSIST service runs as part of the demo app with a transparent UI. 
* The demo app must be run first for the ASSIST service to be discoverable and act on mobile commands.

### Build and Run

* Set the target to `demo-app`
* Select the Android TV device/simulator
* Hit the run `Run demo-app` button

## System Service

### Notes

The System Service is the production version that can be built and immediately use in custom Android TV platforms.

### AOSP Code Changes

You can see a version of the AOSP code changes with the ASSIST Service here: <XYZ-with-assist>

Steps to create Assist System Services

Step 1: Copy the mentioned ServiceManager classes from `system-service` module of ASSIST project to your AOSP File location:  `frameworks/base/core/java/android/app`  
Files to be Copied:  
  `AssistServiceManager.java`  
  `lAssistServiceManager.aidl`  
  `IAssistServiceManager.java`  

Step 2: Copy the mentioned Service class from `system-service` module of ASSIST project to your AOSP File location: `frameworks/base/services/core/java/com/android/server`  
File to be Copied:  
  `AssistService.Java`

Step 3: Modify the following files to register the System Service  
  
File: `frameworks/base/core/java/android/app/SystemServiceRegistry.java`  
```
//SystemServiceRegistry.java
registerService(Context.ASSIST_SERVICE, AssistServiceManager.class,
            new CachedServiceFetcher < AssistServiceManager > () {
                @Override
                public AssistServiceManager createService(ContextImpl ctx) throws ServiceNotFoundException {
                    IBinder binder;
                    if (ctx.getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.O) {
                        binder = ServiceManager.getServiceOrThrow(Context.ASSIST_SERVICE);
                    } else {
                        binder = ServiceManager.getService(Context.ASSIST_SERVICE);
                    }
                    return new AssistServiceManager(ctx, IAssistServiceManager.Stub.asInterface(binder));
                }
            });
  
```
File: `frameworks/base/core/java/android/content/Context.java` 
```
  //Context.java 
  //Add ASSIST_SEVICE 
   @StringDef(suffix = {
            "_SERVICE"
        }, value = {
+           ASSIST_SERVICE,
            ACCOUNT_SERVICE,
            ACTIVITY_SERVICE,
            ALARM_SERVICE,
            NOTIFICATION_SERVICE,
            ACCESSIBILITY_SERVICE,
            CAPTIONING_SERVICE,
        })
  
+  public static final String ASSIST_SERVICE = "assist"; 

  ```
  
File: `frameworks/base/services/java/com/android/server/SystemServer.java`
  ```
  //SystemServer.java
        private static final String ASSIST_SERVICE = "com.android.server.AssistService";

        //added in the startBootstrapServices() function 
        AssistService assistservice = null;
        try {
            traceBeginAndSlog("AssistService");
            assistservice = new AssistService(mSystemContext);
            ServiceManager.addService(Context.ASSIST_SERVICE, assistservice);
        } catch (Throwable e) {
            Slog.e(TAG, "Starting AssistService failed!!! ", e);
        }
        traceEnd();
        
  ```

Step 4: Build the AOSP and run it.

Step 5: Check If the System Service Is running using Logs.

### Build and Deploy

???
