IOS Push Notification Steps
===========================
Simple IOS APNs Test project with SNS integration.

Create IOS Project
-------------------
1. Create a new project in XCode.
    1. Choose **Single View Application**.
    2. Enter **APNSSampleApp** for **Product Name**.
    3. Enter **com.example** for **Organization Identifier**.
    4. Leave **Language** to **Swift**.
2. Replace **AppDelegate.swift** file content with the following:

    ```swift
     
    //
    //  AppDelegate.swift
    //  SNSAPNSSwiftSample
    //

    import UIKit
    import UserNotifications

    @UIApplicationMain
    class AppDelegate: UIResponder, UIApplicationDelegate {

        var window: UIWindow?


        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
            // Override point for customization after application launch.
            
            let center = UNUserNotificationCenter.current()
            // Request permission to display alerts and play sounds.
            center.requestAuthorization(options: [.alert, .sound, .badge])
            { (granted, error) in
                // Enable or disable features based on authorization.
                print("Permission granted: \(granted)")
                // 1. Check if permission granted
                guard granted else { return }
                // 2. Attempt registration for remote notifications on the main thread
                DispatchQueue.main.async {
                    UIApplication.shared.registerForRemoteNotifications()
                }
            }
            
            return true
        }
        
        func application(_ application: UIApplication,
                         didRegisterForRemoteNotificationsWithDeviceToken
            deviceToken: Data) {
            
            let token = (deviceToken as NSData).description.replacingOccurrences(of: "<", with: "").replacingOccurrences(of: ">", with: "").replacingOccurrences(of: " ", with: "")
            print("Successfully registered for push notifications.")
            print("Device token: \(token)")
        }
        
        func application(_ application: UIApplication,
                         didFailToRegisterForRemoteNotificationsWithError
            error: Error) {
            
            // Try again later.
            print("Failed to register for push notifications.")
            print("\(error), \(error.localizedDescription)")
        }
        
        func application(
            _ application: UIApplication,
            didReceiveRemoteNotification userInfo: [AnyHashable: Any],
            fetchCompletionHandler completionHandler:
            @escaping (UIBackgroundFetchResult) -> Void) {
            
            if (application.applicationState == .active) {
                let alert = UIAlertController(title: "Notification Received",
                                              message: userInfo.description,
                                              preferredStyle: .alert)
                alert.addAction(UIAlertAction(title: "Ok", style: .default, handler: nil))
                
                UIApplication.shared.keyWindow?.rootViewController?.present(
                    alert, animated: true, completion:nil)
            }
        }

        func applicationWillResignActive(_ application: UIApplication) {

        }

        func applicationDidEnterBackground(_ application: UIApplication) {

        }

        func applicationWillEnterForeground(_ application: UIApplication) {

        }

        func applicationDidBecomeActive(_ application: UIApplication) {

        }

        func applicationWillTerminate(_ application: UIApplication) {

        }

    }
    ```
    
Register App Id in iTunes Connect
---------------------------------
1. In Xcode, go to project setting -> **Capabilities** tab, Switch **On** Push Notifications.
2. Connect your iOS device to your computer, and let Xcode add your device to the device list by click **Fix Issue**
3. Open iTunes Connect by visiting [here](https://developer.apple.com/account/ios/identifiers/bundle/bundleList.action).
4. In App IDs, find and click **XC com example APNSSampleApp**
5. Click **Edit**.
6. For **Push Notifications** under **Development SSL Certificate** click **Create Certificate**.
7. Click **Continue** to get to **Upload CSR** page.
8. Generate a new CSR using Keychain:
    1. Open **Keychain Access**.
    2. From **Keychain Access** menu -> **Certificate Assistant** sub menu click **Request a Certificate From a Certificate Authority...**
    3. Enter your email address, **APNSSampleApp Development** in the Common Name field.
    4. Select **Saved to disk** option.
    5. Click **Continue** and **Save**.
9. Back in upload CSR page in iTunes Connect click **Choose file**, select the saved CSR and upload.
10. Click **Generate** button.
11. After certificate is generated click **Download** button to download.
12. Save it as **APNSSampleAppDevelopment.cer**.
13. Double click **APNSSampleAppDevelopment.cer** to add to Keychain.
14. In **My Certificates** section, right click the added certificate in Keychain and click **Export**.
15. Save as **APNSSampleAppDevelopment.p12**.
16. **DO NOT ENTER A PASSWORD** when prompted to secure the exported certificate.

Get device Token
----------------
1. Run the app in xCode and select your device as the target to run on.
2. Tap OK to allow push notifications as the application starts. Notifications can also be turned On from device settings -> notifications.
3. As the application starts it will register with Apple and will log device token in xCode console which we will need for sending notification to the device.

Publishing Notification using AWS SNS
-------------------------------------
Advantage of using SNS is scalability which is required when publishing tens of millions of notifications in a very short time and abstracts interaction with different push services behind a unified API.

1. Create a new platform application in SNS console -> **Side Menu - Mobile - Push notifications**.
2. Enter **APNSSampleApp** for the name.
3. Select **Apple iOS/VoIP/Mac** from the **Push notification platform** drop down menu.
4. Select **Used for development in sandbox** as the generated certificate is for development
5. Select **iOS push certificate** from the **Push certificate type** drop down menu.
6. Click **Choose file** button.
7. Select **APNSSampleAppDevelopment.p12** file that we exported from Keychain.
8. Click **Load Credentials from File** to populate **Certificate** and **Private Key** boxes.
9. Click **Create platform application** button.
10. Click on the new application name to enter.
11. Click **Create application endpoint** button.
12. Paste your **Device Token** in the **Device token** field.
13. Enter optional data in **User Data** field.
14. Click **Create application endpoint** button.
15. Select the newly added endpoint from the list.
16. Click **Publish message** button.
17. Enter the following in the **Message body to send to the endpoint** box and click **Publish message** button.

    ```json
    {
    "APNS_SANDBOX":"{\"aps\":{\"alert\":\"Test message from SNS console.\", \"title\": \"APNSSampleApp\"}}"
    }
    ```        
    
