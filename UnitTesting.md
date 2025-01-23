## Unit Testing

### How to do unit testing

On the viewModel/ system (file) under test
1.	Look at the view model you would like to test
2.	Identify its dependencies
3.	Identify if you want to mock the manager or the repository
In the mock 
1.	Create an integer to count how many times it returns a result
2.	Create an object of the type the repository or the manager is going to return
In the test in setup
1.	Create an instance of the viewmodel you want to test and call it sut 
2.	Create instances of your dependencies
3.	Inject the instances of the dependencies 
In the test in teardown
1.	Set all dependencies and instances to nil
Writing the tests
1.	Start with setting all the variables in the Given section
2.	Call the function you want to mock from the viewmodel
3.	Asert the result you would like to receive
If there is a dispatchQueue.main we need to use XCTestExpectation with a fulfill after the assert and wait(for: [expectation], timeout: 1)


Example of test with DispatchQueue 
class RootViewModel_Tests: XCTestCase {
    
    private var sut: RootViewModel!
    private var mockAppUpdateManager: MockAppUpdateManager!
    
    override func setUp() {
        sut = RootViewModel()
        
        // Init Dependency
        mockAppUpdateManager = MockAppUpdateManager()
        
        // Inject Dependency
        DependencyValues[\.appUpdateManager] = mockAppUpdateManager
    }
    
    override func tearDown() {
        sut = nil
        mockAppUpdateManager = nil
    }
    
    func test_noUpdate_showForceUpgradeScreenFalse() async {
        // 1. Given
        let expectation = XCTestExpectation(description: "App update status fetched and applied")
        mockAppUpdateManager.fetchAppUpdateResult = AppUpdateStatus(upgradeStatusId: .noUpdate, message: "No update required")
        // 2. When
        await sut.fetchAppUpdateStatus()
        DispatchQueue.main.async {
            // 3. Then
            XCTAssertFalse(self.sut.showForceUpgradeScreen)
            expectation.fulfill()
        }
        wait(for: [expectation], timeout: 1)
    }
    
    
    func test_soft_showForceUpgradeScreenTrue() async {
        // 1. Given
        let expectation = XCTestExpectation(description: "App update status fetched and applied")
        mockAppUpdateManager.fetchAppUpdateResult = AppUpdateStatus(upgradeStatusId: .soft, message: "Soft update required")
        // 2. When
        await sut.fetchAppUpdateStatus()
        DispatchQueue.main.async {
            // 3. Then
            XCTAssertTrue(self.sut.showForceUpgradeScreen)
            expectation.fulfill()
        }
        wait(for: [expectation], timeout: 1)
    }
    
    func test_hard_showForceUpgradeScreenTrue() async {
        // 1. Given
        let expectation = XCTestExpectation(description: "App update status fetched and applied")
        mockAppUpdateManager.fetchAppUpdateResult = AppUpdateStatus(upgradeStatusId: .force, message: "Hard update required")
        // 2. When
        await sut.fetchAppUpdateStatus()
        DispatchQueue.main.async {
            // 3. Then
            XCTAssertTrue(self.sut.showForceUpgradeScreen)
            expectation.fulfill()
        }
        wait(for: [expectation], timeout: 1)
    }
}




![image](https://github.com/user-attachments/assets/4f37811e-939d-4fb6-9a72-f0a79ed4ade8)


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
