## Unit Testing

### How to do unit testing

```swift
import Foundation
@testable import Tracker // Name of your target

1. Find the class you want to test
2. Go into the class and see at the top what data it needs (Dependencies)
final class ExampleTests: XCTestCase {
    // What you are testing
    private var sut: ExpenseViewModel!

    // Dependencies you have inside ExpenseViewModel that you need to test
    private var mockExpensesRepository: MockExpenseRepository!
    private var mockUserDefaults: MockUserDefaultsCache!
}

    override func setUp() {
// Instantiate the class you want to test
        sut = ExpenseViewModel()
        // Initialise Dependency
        mockExpensesRepository = MockExpenseRepository()
        mockUserDefaults = MockUserDefaultsCache()
        // Inject Dependency
        DependencyValues[\.expensesRepository] = mockExpensesRepository
        // Inject Dependency
        DependencyValues[\.userDefaultsCache] = mockUserDefaults
    }
    
    override func tearDown() {
        // Deinitialise
        sut = nil
        mockExpensesRepository = nil
        mockUserDefaults = nil
    }
```

### Combine unit test
```swift
// test_whatYouTesting_WhatYouExpect
    func test_DateSections_ReturnsCorrectValue() {
        // 1. Given
        sut.expenseList = []
        let mockDate = "2020-08-01T22:00:00Z".toDate() ?? Date()
        mockExpensesRepository.fetchExpenseResults = Just(ExpensesResponse(value: VehicleExpense.examples, count: 1))
            .setFailureType(to: Error.self)
            .eraseToAnyPublisher()
        mockExpensesRepository.fetchExpenseTypeResult = Just([FetchExpenseTypeResponseValue(id: 12, expenseType: "93 Petrol")])
            .setFailureType(to: Error.self)
            .eraseToAnyPublisher()
        // 2. When
        sut.fetchExpenses()
        // 3. Then
        XCTAssertEqual(sut.dateSections.first?.month, mockDate.month)
    }
```

### Asyn/Await unit test
```swift
// test_whatYouTesting_WhatYouExpect
   func test_noUpdate_showForceUpgradeScreenFalse() async {
        let expectation = XCTestExpectation(description: "App update status fetched and applied")
        // 1. Given
        mockAppUpdateRepository.fetchAppUpdateResult = AppUpdateStatus.exampleNoUpdate
        // When
        await sut.fetchAppUpdateStatus()
        DispatchQueue.main.async {
        // 3. Then
            XCTAssertFalse(self.sut.showForceUpgradeScreen)
            expectation.fulfill()
        }
        await fulfillment(of: [expectation], timeout: 1.0)
    }
```

### How to mock repositories

```swift
import Foundation
@testable import Tracker

// Conform it to the actual repository protocol so that we have all the fucntions to mock
class MockAppUpdateStatusRepository: AppUpdateStatusRepositoryType {

// Create a counter to detemrine how many times this is called
    var fetchAppUpdateStatusDetailsCallCount = 0
// Create a variable to set with example data
    var fetchAppUpdateResult: AppUpdateStatus?

    func fetchAppUpdateStatusDetails() async -> AppUpdateStatus {
        fetchAppUpdateStatusDetailsCallCount += 1
// If the data is not nil assign it to the result 
        if let fetchAppUpdateResult {
            return fetchAppUpdateResult
        }
        return AppUpdateStatus(upgradeStatusId: .noUpdate, message: "")
    }
}

```

### How to mock user defualts 

```swift
import Foundation
@testable import Tracker

class MockUserDefaultsCache: UserDefaultsCacheType { // Conform to the protocol

// Define a dictionary to store key value pairs
private var inMemoryCache: [String: Any] = [:]

// Define a counter to see how many times the call is made
 var objectCallCount = 0

    func object(key: String, suite: UserDefaultsCacheSuite = .standard) -> Any? {
        objectCallCount += 1 // Increment the counter
        return inMemoryCache.safeValue(forKey: key)
    }
}
```
