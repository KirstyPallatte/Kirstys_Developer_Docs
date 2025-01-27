## Swift Testing

http://leocoout.medium.com/become-a-unit-test-master-84f4fa276deb
https://leocoout.medium.com/welcome-swift-testing-goodbye-xctest-7501b7a5b304

### Given, when then

1. GIVEN: it's when we define the starting data and set up our objects;
2. WHEN: It's the key action; we need to call the method under test with the given parameters;
3. THEN: here we verify the system output and see if it behaves expectedly.

## Pro
* Swift Testing runs tests in parallel by default, for sync and async tests, while XCTest only supports parallelization using multiple processes each running one test at a time.

### How to do Swift Testing
1. import Testing
2. Declare function as @Test
3. Use #expect to do the assertions
4. No more setUp and tearDown, only init and deinit

```swift 
import Testing

init() {

}

deinit() {

}

@Test myFirstTest() {
  #expect(1 == 1) 
}
   ```

```swift
import Testing

final class JobOfferViewModelTests {
  private let applicationSpy = ApplicationSpy()
  private lazy var sut: JobOfferViewModel = {
    // Implementation hidden for brevity
  }()
    
   @Test callPassenger_givenPassengerNumber_shouldOpenCorrectURL() {
      sut.jobDataToBeReturned = .fixture(passengerContact: "1234-5678")
      sut.callPassenger()
      #expect(applicationSpy.openedURL == URL(string: "tel://1234-5678"))
   }
}
```

### What is a fixture
 1. You do not need to write all the parameters every time, just the ones you are going to use. (Specifies defualt values for the initializier)

```swift

struct Person {
    let name: String
    let age: Int
}

extension Person {
    static func fixture(name: String = "", age: Int = 0) -> Self {
        .init(name: name, age: age)
    }
}

@Test func testA() {
    let person: Person = .fixture(age: 18)
    #expect(person.age, 18)
}

@Test func testB() {
    let person: Person = .fixture(name: "Leonardo")
     #expect(person.name, "Leonardo")
}

```

### Traits
* Give descriptive names to tests instead of showing the function name

```swift
@Test("Given correct user number and password, should return success")
func validateLogin() {...}
```

### Tag trait
A type representing a tag that can be applied to a test. We can create new tags by extending the Tag object and applying the @Tag macro:

```swift
@Test(.tags(.jobBidding))

private extension Tag {
  @Tag static var jobBidding: Self
}

```
* Tags provide semantic information for a test that can be shared with any number of other tests across test suites, source files, and even test targets. They can be filtered in the left-side menu, run separately from the other cases, or even filtered from the Insights menu after a test run.

### Pros of tags

1. Helps you analyze results across test targets
2. Quickly include or exclude tags from your test plan
3. See insights about each tag

### Toggle trait
* Enable or disable your test based on conditions. By doing this, you specify a runtime-evaluated condition that skips your test if the condition is met:

```swift
@Test(.disabled()) // Works ✅
func routeToPaymentScreen() {}

@Test(.enabled(if: FeatureFlag.isCallPassengerEnabled)) // Works ✅
func callPassenger() {}

Given a scenario where you know a unit test is failing, instead of commenting out or disabling using the .disabled() trait, prefer using the withKnownIssue.

@Test func thisFunctionShouldFail() async {
    withKnownIssue {
      sut.thisFunctionShouldFail()
      #expect(routeTriggerSpy.values == [.pop(animated: false)])
   }
}
 ```

### TimeLimit trait
Set a maximum time limit for a test:

```swift
@Test(.timeLimit(.minutes(3))) 
func callPassenger() {}

@Test(.timeLimit(.seconds(1))) 
func callPassenger() {}
```

### Parametrized Tests
* Swift Testing automatically splits it up into separate test cases, one per argument.

```swift
  @Test(
  "Given different payment type, should display correct alert",
  arguments: [PaymentType.creditCard, .payNow, .nets, .onBoard] // Arguments
)
func displayAlertMessage(paymentType: PaymentType) { // Function that receives the argument
  let sut = PaymentAdditionalItemViewModel(
        payment: .fixture(type: paymentType)
    )  
  #expect(sut.payment.description == paymentType.description)
}
```

2. You can also pass arguments of different types

```swift
@Test(arguments: [
  (TaxiRideType.metered, Driver.fixture(name: "Joel")),
  (TaxiRideType.normal, Driver.fixture(name: "Chong"))
])
```

3. Careful! For multiple arguments, Swift Testing will run your testing function for every combination of elements, which may increase the amount of testing for each new item added to one of the parameters.

### Testing Async Code

```swift
@Test func fetchJobDetails() async { // Mark function as async
    jobServiceSpy.getJobExpectedResult = getJobExpectedResult
    await sut.onTapBidButton() // Await for the function
    #expect(jobServiceSpy.jobNo == "12345")
    #expect(routeTriggerSpy.values == [...])
}
```

### Validating Error is Thrown

```swift
@Test("When trying to pay without internet, should display noInternet error") 
func payment() throws {
  #expect(throws: ErrorEnvelope.noInternet.self) {
    try sut.doPaymentUseCase()
  }       
}

```

### Validating error is NOT thrown

```swift
@Test("When trying to pay with internet, should not throw errors") 
func payment() throws {
  #expect(throws: Never.self) {
    try sut.doPaymentUseCase()
  }       
}
```

### Testing how many times a function was called
* The Confirmation object has a built-in counter that can be used by passing the expectedCount parameter.

  ```swift
@Test func fetchJobDetails() async {
  jobServiceSpy.serviceToBeReturned = .success(.fixture(jobNo: "0"))

  await confirmation(expectedCount: 1) { incrementCounter in
     jobServiceSpy.serviceCounterHandler = { incrementCounter() } // Trigger this counter every time the service is called 
     await sut.fetchJobDetails()
  }

  #expect(jobServiceSpy.jobDetails.jobNo == "12345")
  #expect(sut.jobData?.jobNo == "0")
}
  ```

### Dealing with Optionals

* XCTFail was replaced byIssue.record()
* XCTUnwrap was replaced by #require

```swift
// Prefer using this
@Test uploadTrip() { 
  let tripData = try #require(sut.uploadTripData)
  #expect(tripData.jobNo == "12345")
}
```
