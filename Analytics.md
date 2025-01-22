## Analytics


## Set users id
`Analytics.setUserID(UUID().uuidString)`
## Set a property
`Analytics.setUserProperty(newSelectedSeason?.rawValue, forName: "favorite_season")  // User property allows us to set a value for the anayltics eg: Index of season chosen `
## Log an event
`Analytics.logEvent("greetings", parameters: nil) // User property allows us to set a value for the anayltics eg: Index of season chosenThs allows us to see the event name inside analytics `
