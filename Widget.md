## Widgets

Only avialble in IOs 17.0 and above

1. Add a WidgetBundle and import WidgetKit

```swift
import WidgetKit
import SwiftUI
@main
struct TrackerWidgetBundle: WidgetBundle {
    var body: some Widget {
        StatusWidget() // View you want to show as a widget
    }
}
```

2. Every widget has a unique kind, a string that you choose. You use this string to identify your widget when reloading its timeline with WidgetCenter.
3. The timeline provider is an object that determines the timeline for refreshing your widget. Providing future dates for updating your widget allows the system to optimize the refresh process.
4. The content closure contains the SwiftUI views that WidgetKit needs to render the widget. When WidgetKit invokes the content closure, it passes a timeline entry created by the widget provider’s getSnapshot(in:completion:) or getTimeline(in:completion:) method.

### Set up the WidgetConfiguration and give it a kind (identify what should be reloaded when timeline reaches futureDate and a provider)
``` swift
import WidgetKit
import SwiftUI

struct StatusWidget: Widget {
    static let kind: String = "TrackerStatusWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(
            kind: StatusWidget.kind,
            provider: StatusWidgetProvider()
        ) { entry in
                StatusWidgetEntryView(entry: entry)
                    .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("Vehicle Status")
        .description("Status of vehicle and location")
        .supportedFamilies([.systemMedium]) // Support only medium size
    }
}
```

1. Create a view that takes in an entry of type StatusWidgetProvider.Entry where Entry is the model for the widget
2. Create @environment variable for widgetFamily to change the fonts

### Display the entry data

``` swift

import WidgetKit
import SwiftUI

struct StatusWidgetEntryView: View {
    var entry: StatusWidgetProvider.Entry
    
    @Environment(\.widgetFamily) private var widgetFamily

    var body: some View {
        Group {
            if widgetFamily == .systemMedium {
                if !entry.userLoggedIn {
                    Text("Please login to view your vehicle status")
                } else if entry.needToSelectVehicle {
                    VStack(alignment: .center) {
                        Text("Please select your vehicle")
                            .padding(.bottom, 5)
                        Text("Long press this widget and tap on 'Edit Widget'")
                            .multilineTextAlignment(.center)
                    }
                } else {
                    VStack(alignment: .leading) {
                        HStack(alignment: .top) {
                            VehicleStatusView(basicStatus: entry.vehicleStatus,
                                              customAddressIcon: entry.customAddressIcon,
                                              speed: Int(round(entry.vehicleSpeed)))
}

```

## Create dummy data for entry
``` swift
import WidgetKit
import SwiftUI

// The Entry is the model we use for the widget

struct StatusEntry: TimelineEntry, Codable {
    // This is the date that the widget will reload its view
    let date: Date
    
    // Custom (This is for displaying wehn we are going to reload the widget again in the widget UI) (Degugging purposes)
    var nextUpdateDate: Date
    
    // Vehicle Info
    let vehicleId: Int
    let vehicleRegistration: String
    let vehicleStatus: String
    let vehicleSpeed: Double
    let vehicleLocation: String
    let customAddressIcon: Int
    let rtcDate: Date?
    let alias: String
    
    // Account Info
    let userLoggedIn: Bool
    let needToSelectVehicle: Bool
}

extension StatusEntry {
    /// Data when user is logged in but no vehicle has been selected for the widget yet)
    static func loggedInNeedToSelectVehiclePlaceholder() -> StatusEntry {
        StatusEntry(date: Date(),
                    nextUpdateDate: Date(),
                    vehicleId: 1,
                    vehicleRegistration: "",
                    vehicleStatus: "",
                    vehicleSpeed: 0.0,
                    vehicleLocation: "",
                    customAddressIcon: 1,
                    rtcDate: Date(),
                    alias: "",
                    userLoggedIn: true,
                    needToSelectVehicle: true)
    }
    /// Data when user is logged out (In this case we display user must login to see data)
    static func loggedOutPlaceholder() -> StatusEntry {
        StatusEntry(date: Date(),
                    nextUpdateDate: Date(),
                    vehicleId: 1,
                    vehicleRegistration: "",
                    vehicleStatus: "",
                    vehicleSpeed: 0.0,
                    vehicleLocation: "",
                    customAddressIcon: 1,
                    rtcDate: Date(),
                    alias: "",
                    userLoggedIn: false,
                    needToSelectVehicle: false)
    }
    /// Data when widget has no data and busy loading data for widget
    static func placeholder() -> StatusEntry {
        StatusEntry(date: Date(),
                    nextUpdateDate: Date(),
                    vehicleId: 1,
                    vehicleRegistration: "Loading...",
                    vehicleStatus: "Loading...",
                    vehicleSpeed: 0.0,
                    vehicleLocation: "Loading...",
                    customAddressIcon: 1,
                    rtcDate: Date(),
                    alias: "",
                    userLoggedIn: true,
                    needToSelectVehicle: false)
    }
    /// Data for the widget preview in gallery when adding widget
    static func galleryPlaceHolder() -> StatusEntry {
        StatusEntry(date: Date(),
                    nextUpdateDate: Date(),
                    vehicleId: 1,
                    vehicleRegistration: "AABBCCGP",
                    vehicleStatus: "Moving",
                    vehicleSpeed: 40.0,
                    vehicleLocation: "Street address",
                    customAddressIcon: 1,
                    rtcDate: Date(),
                    alias: "",
                    userLoggedIn: true,
                    needToSelectVehicle: false)
    }
    /// Actual data for the widget to display (user is logged in and selected vehicle)
    static func entry(vehicleStatus: VehicleStatus, vehicleResponse: VehicleResponse? = nil) -> StatusEntry {
        StatusEntry(date: Date(),
                    nextUpdateDate: Date(),
                    vehicleId: vehicleStatus.vehicleId,
                    vehicleRegistration: vehicleResponse?.registration ?? "",
                    vehicleStatus: vehicleStatus.vehicleBasicStatus,
                    vehicleSpeed: vehicleStatus.motion?.speed ?? 0.0,
                    vehicleLocation: vehicleStatus.address?.addressString ?? "",
                    customAddressIcon: 1,
                    rtcDate: vehicleStatus.lastUpdatedAt, // TODO: Pieter - This is not the date we use in the app // we use the Vehicle object date. Need to change or see what todo here.
                    alias: vehicleResponse?.alias ?? "",
                    userLoggedIn: true,
                    needToSelectVehicle: false)
    }
}
```


### Timeline entry provider

1. Conform the struct to use AppIntentTimelineProvider
2. Set a widget preview when theres no data for that timeline
3. Set a widget snapshot which is shown when they setup the widget in their gallery
4. Set up a timeline that gets called on every update cycle
5. Define a helper method to fetch StatusEntry data async and remember once it is completed say .resume or it will hang the app

   // This updates every 15 minutes

``` swift
import WidgetKit
import SwiftUI

struct StatusWidgetProvider: TimelineProvider {
    @Inject(\.apiClient) private var apiClient
    @Inject(\.apiAuthenticator) private var apiAuthenticator
    @Inject(\.userDefaultsCache) private var userDefaultsCache
    @Inject(\.fileManagerCache) private var fileManagerCache

    init() {
        Logging.widget.log("init")
        WidgetExtConfig.configureAppLogging()
        WidgetExtConfig.configureServerConfig()
    }

    // Placeholder widget data when no timeline is available
    func placeholder(in context: Context) -> StatusEntry {
        Logging.widget.log("placeholder()")
        return StatusEntry.placeholder()
    }

    // Preview widget data for the widget gallery
    func getSnapshot(in context: Context, completion: @escaping (StatusEntry) -> Void) {
        Logging.widget.log("getSnapshot()")
        completion(StatusEntry.galleryPlaceHolder())
    }

    // Fetch and provide a timeline for the widget
    func getTimeline(in context: Context, completion: @escaping (Timeline<StatusEntry>) -> Void) {
        Logging.widget.log("getTimeline() - Start")
        
        // Increment the refresh count for debugging
        incrementWidgetRefreshCount()

        // Check if the user is logged in
        guard isLoggedIn() else {
            let currentDate = Date()
            let nextUpdate = Calendar.current.date(byAdding: .minute, value: 15, to: currentDate)!
            Logging.widget.log("getTimeline() - User not logged in")
            completion(Timeline(entries: [StatusEntry.loggedOutPlaceholder()], policy: .after(nextUpdate)))
            return
        }

        // Fetch the selected vehicle ID from UserDefaults or another storage mechanism
        let selectedVehicleId = getSelectedVehicleId()
        guard selectedVehicleId != "0" else {
            let currentDate = Date()
            let nextUpdate = Calendar.current.date(byAdding: .minute, value: 15, to: currentDate)!
            Logging.widget.log("getTimeline() - No vehicle selected")
            completion(Timeline(entries: [StatusEntry.loggedInNeedToSelectVehiclePlaceholder()], policy: .after(nextUpdate)))
            return
        }

        // Fetch widget data asynchronously
        Task {
            var entry = await fetchEntryAsync(vehicleId: selectedVehicleId)
            var updateIntervalMin = 2
            if entry.vehicleStatus == "IgnitionOff" {
                updateIntervalMin = 15
            }
            
            let currentDate = Date()
            let nextUpdate = Calendar.current.date(byAdding: .minute, value: updateIntervalMin, to: currentDate)!
            entry.nextUpdateDate = nextUpdate
            updateWidgetLastUpdatedAtDate(date: currentDate)
            updateWidgetNextUpdatedAtDate(date: nextUpdate)
            Logging.widget.log("getTimeline() - Finished - currentDate: \(currentDate), nextUpdate: \(nextUpdate), in \(updateIntervalMin) min")
            
            // Provide the timeline
            completion(Timeline(entries: [entry], policy: .after(nextUpdate)))
        }
    }

    // Helper to get selected vehicle ID
    private func getSelectedVehicleId() -> String {
        // Fetch the vehicle ID from UserDefaults or a similar mechanism
        userDefaultsCache.string(forKey: .selectedVehicleIdKey) ?? "0"
    }
}

```

``` swift
// Helper methods to fetch new data for the widget

extension StatusWidgetProvider {
    private func isLoggedIn() -> Bool {
        apiAuthenticator.currentToken != nil
    }

    private func fetchEntryAsync(vehicleId: String) async -> StatusEntry {
        await withCheckedContinuation { continuation in
            fetchEntry(vehicleId: vehicleId) { entry in
                continuation.resume(returning: entry)
            }
        }
    }

    private func fetchEntry(vehicleId: String, completion: @escaping (StatusEntry) -> Void) {
        Logging.widget.log("Fetching new snapshot for vehicleId: \(vehicleId)")
        let id = Int(vehicleId) ?? 0
        let request = GetVehicleStatus(vehicleId: id)
        apiClient.executeClosure(request: request, cache: .loadOnly) { (result: Result<VehicleStatus, any Error>) in
            switch result {
            case .success(let response):
                Logging.widget.log("Finished fetching snapshot - Success")
                if let vehicles: Vehicles = fileManagerCache.load(from: .vehicles),
                   let vehicleResponse = vehicles.value.first(where: { $0.id == id }) {
                    completion(StatusEntry.entry(vehicleStatus: response, vehicleResponse: vehicleResponse))
                } else {
                    completion(StatusEntry.entry(vehicleStatus: response))
                }
            case .failure(let error):
                Logging.widget.error("Finished fetching snapshot - Failure: \(error)")
                completion(StatusEntry.placeholder())
            }
        }
    }
}
   ```

### Specify what widget we want to update

``` swift
  static func reloadWidgets() {
        Logging.widget.log("Manual Reload Widget Extensions")
        WidgetCenter.shared.reloadTimelines(ofKind: "TrackerStatusWidget")
    }

```
