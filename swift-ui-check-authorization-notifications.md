# swift-ui-check-authorization-notifications

We can create a new class called Notification, and inside a static function called checkAuthorization. This function will be responsible to getting the user notification center and check the status, if is authorized or not, or if we need to request because is yet to be determined.

```swift
class PomodoroNotification {
    
    static func checkAuthorization(completion: @escaping(Bool) -> Void){
        let notificationCenter = UNUserNotificationCenter.current()
        notificationCenter.getNotificationSettings{settings in
            switch settings.authorizationStatus {
                case .authorized:
                    completion(true)
                case .notDetermined:
                notificationCenter.requestAuthorization(options: [.alert, .badge, .sound], completionHandler: {allowed, error in completion(allowed)})
                    
                default:
                    completion(false)
            }
        }
    }
```


After that, in our view we can use the scenePhase enviroment on change event to call our check autorization method


```swift

import SwiftUI

struct Notificationdemo: View {
    @State private var showWarning = false
    @Environment(\.scenePhase) var scenePhase
    var body: some View {
        VStack{
            Button("send not") {
                PomodoroNotification.scheduleNotification(seconds: 5, title: "Title", body: "Body")
            }
            
            if showWarning {
                VStack {
                    Text("Notifications are disabled")
                    Button("Enable"){
                        
                    }
                }
            }
        }
        .onChange(of: scenePhase) {
            if(scenePhase == .active){
                PomodoroNotification.checkAuthorization {
                        authorized in showWarning = !authorized
                    }
                }
            }
        }
}

#Preview {
    Notificationdemo()
}
```
