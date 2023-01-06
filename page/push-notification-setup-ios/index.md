---
title: "Push Notification Setup - iOS-   Archived"
date: "2015-08-04"
---

## Prerequisites

- Must have an iOS developer account
- In order to configure brainCloud to send notifications to your iOS app, you will need to provide a p12 certificate file. [Follow these steps](http://www.mobiloud.com/help/knowledge-base/how-to-export-push-notification-certificate-p12/) to create a p12 file for your app.

## brainCloud Portal Configuration

- Log into the brainCloud portal
- Navigate to "Notifications | Settings" in the design tab for your app  
    [](/apidocs/wp-content/uploads/2016/09/brainCloud_iosNotif_1.jpg)
- Click on the edit button for the Apple notification settings  
    [](/apidocs/wp-content/uploads/2016/09/brainCloud_iosNotif_2.jpg)
- Upload your p12 file and enter the password that was used to create the p12 file into the dialog
- Notice that once you've done that, the type of certificate shows up (production/sandbox) as well as the expiry date  
    [](/apidocs/wp-content/uploads/2016/09/brainCloud_iosNotif_3.jpg)

**Unity Code Example**

using UnityEngine;
using UnityEngine.iOS;

public class App : MonoBehaviour
{
    private BrainCloudWrapper \_bc;
    private string \_wrapperName = "default";
    
    bool tokenSent;
    void Start()
    {
        // Unity
        GameObject go = new GameObject();
        \_bc = go.AddComponent<BrainCloudWrapper>();
        \_bc.WrapperName = \_wrapperName; // optionally set a wrapper-name
        \_bc.Init(); // extra data, such as: \_appId, \_secret and \_appVersion, is taken from the brainCloud Unity Plugin.
        DontDestroyOnLoad(go); // keep the brainCloud game object through scene changes
     
        
        // Authenticate your user with brainCloud 
        \_bc.AuthenticateAnonymous((response, cbObject) =>
            {
                Debug.Log(string.Format("\[Auth Success\] {0}", response));
                
            },
            (status, code, error, cbObject) =>
            {
                Debug.Log(string.Format("\[Auth Failed\] {0}  {1}  {2}", status, code, error));
            });
        
        
        tokenSent = false;
        UnityEngine.iOS.NotificationServices.RegisterForNotifications(
            NotificationType.Alert |
            NotificationType.Badge |
            NotificationType.Sound);
    }

    void Update()
    {
        
        if (!tokenSent)
        {
            byte\[\] token = UnityEngine.iOS.NotificationServices.deviceToken;
            
            // When token exists, and brainCloud is Authenticated
            if (token != null && \_bc.Client.Authenticated)
            {
                // Send the token to brainCloud
                \_bc.PushNotificationService.RegisterPushNotificationDeviceToken(
                    token, 
                    (response, cbObject) =>
                    {
                        Debug.Log(string.Format("\[Register Success\] {0}", response));
                    },    ((status, code, error, cbObject) =>
                    {
                        Debug.Log(string.Format("\[Register Failed\] {0}  {1}  {2}", status, code, error));
                    }));
                
                tokenSent = true;
            }
        }
    }
}
