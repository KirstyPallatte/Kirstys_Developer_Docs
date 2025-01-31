# Migrate from NavigationLink to NavigationStack

1. NavigationView changes to NavigationStack
2. NavigationLink(isActive) that includes a boolean changes to Button {} label: {} so we can perform an action
3. .navigationDestination(isPresented) is like isActive. If the bool for isPresented chnages to false it pops back to the view when the navigation started
4. NavigationLink(destination: ) changes to NavigationLink {} label: {} with a matching .navigationDestination(for: )

## Example of 2 - Need to perform navigation from onTap
```swift

// Old way
       NavigationLink(destination: ResultView(description: "contactUs.querysubmitted.message"),
                               isActive: $viewModel.showsSuccessView) {
                    Button("contactUs.sendEmailButton.label".localized(), action: viewModel.submitSupportTicket)
                }.disabled(!viewModel.sendQueryValidated)

// New way

 Button {
                    viewModel.submitSupportTicket()
                } label: {
                    Text("contactUs.sendEmailButton.label".localized())
                }
                .disabled(!viewModel.sendQueryValidated)

```

## Example of 3 - Need to perform navigation ontap and set a variable so we need a button
```swift

// Pass isRootActive as a binidng to all child views and set to false when want to pop back to the root

// Old way
struct ContactSupportView: View {
    @Binding var isRootViewActive: Bool

// Make the widget tappable to navigate to another view
 Widget(infoTabItem.title) {
                        Image(infoTabItem.image)
                            .resizable()
                           
                    }
                    .background(
                        NavigationLink(destination: infoTabItem.destination($isRootViewActive),
                                       isActive: $isRootViewActive,
                                       label: { EmptyView() }))
                    .onTapGesture {
// Set this to pop to the root
                        isRootViewActive = true
                        isSelected = infoTabItem.rawValue
                    }
}

// New way
struct ContactSupportView: View {
 @State private var isContactUsPresented = false

// Pass isContactUsPresented as a binidng to all child views and set to false when want to pop back to the root

 Button {
     isContactUsPresented = true
     } label: {
     InfoTabCell(item: infoTabItem, viewModel: viewModel)
     }

.navigationDestination(isPresented: $isContactUsPresented) {
                ContactSupportView(viewModel: ContactSupportViewModel(), isPresented: $isContactUsPresented)
}

```
## Example of 4 - Need to perform navigation ontap and not set a variable

```swift

// Old way
      Widget(infoTabItem.title) {
                        Image(infoTabItem.image)
                            .resizable()
                    }
                    .background(
                        NavigationLink(destination: infoTabItem.destination(),
                                       tag: infoTabItem.rawValue,
                                       selection: $isSelected
                                      ) {
                                          EmptyView()
                                      })
                    .frame(maxWidth: 500, alignment: .center)

// New way

    NavigationLink(value: infoTabItem) {
                            InfoTabCell(item: infoTabItem, viewModel: viewModel)
    }
   .navigationDestination(for: InfoTabItems.self) { infoTabItem in
                InfoTabDestination(item: infoTabItem)
            }

```
