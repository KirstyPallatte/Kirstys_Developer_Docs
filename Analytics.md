## Analytics

- [Firebase Setup Documentation]([https://firebase.google.com/docs/ios/setup?authuser=0&hl=en](https://github.com/firebase/quickstart-ios)
  
## Set users id
```swift 
`Analytics.setUserID(UUID().uuidString)
```
## Set a property
/// Needs a value and a key
```swift 
Analytics.setUserProperty(newSelectedSeason?.rawValue, forName: "favorite_season")  // User property allows us to set a value for the anayltics eg: Index of season chosen
```
## Log an event
// Needs a string

```swift 
Analytics.logEvent("greetings", parameters: nil) // User property allows us to set a value for the anayltics eg: Index of season chosenThs allows us to see the event name inside analytics
```

### To make it easier 
* Create an enum

```swift
 enum Event: String, CaseIterable {
    case addExpenses = "AddExpenses"
 }

 // Create AnalyticsManager
   `    private func track(event: Event, params: [String: Any] = [:]) {
        if params.isEmpty {
            Logging.analytics.log("Log Event - event: \(event.rawValue)")
        } else {
            Logging.analytics.log("Log Event - event: \(event.rawValue), params: \(params)")
        }
        
        Analytics.logEvent(event.rawValue, parameters: params)
    }
```
