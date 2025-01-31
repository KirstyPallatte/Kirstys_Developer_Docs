## View structure

1. Public variables at the top
2. Private variables at the bottom
3. NavigationStack where view begins with navigation
4. Navigation structure with all navigation items (toolbar, bar title, navigationDestination)
5. Page that contains all sub views
6. If you have a list define structure of cells
7. If the list navigates use destination struct

```swift
struct InfoTabView: View {
    
    @StateObject private var viewModel = InfoTabViewModel()
    @State private var isSelected: Int?
    private let screens = InfoTabItems.allCases
    @State private var isContactUsPresented = false
    
    var body: some View {
        NavigationStack {
            navigation
        }
    }
    
    var navigation: some View {
        page
            .navigationBarCustomFont(title: "info.navigation.title".localized())
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .topBarTrailing) {
                    SosButton()
                }
            }
            .navigationDestination(for: InfoTabItems.self) { infoTabItem in
                InfoTabDestination(item: infoTabItem)
            }
            .navigationDestination(isPresented: $isContactUsPresented) {
                ContactSupportView(viewModel: ContactSupportViewModel(), isPresented: $isContactUsPresented)
            }
    }
    
    var page: some View {
        ScrollView(.vertical, showsIndicators: false) {
            VStack {
                ForEach(screens, id: \.self) { infoTabItem in
                    if infoTabItem == .frequentlyAskedQuestions {
                        if viewModel.shouldShowFrequentlyAskedQuestions {
                            Button {
                                viewModel.openFrequentlyAskedQuestionURL()
                                viewModel.analytics.track(event: .viewFAQInfo)
                            } label: {
                                InfoTabCell(item: infoTabItem, viewModel: viewModel)
                            }
                        }
                    } else if infoTabItem == .contactUs {
                        Button {
                            isContactUsPresented = true
                        } label: {
                            InfoTabCell(item: infoTabItem, viewModel: viewModel)
                        }
                    } else {
                        NavigationLink(value: infoTabItem) {
                            InfoTabCell(item: infoTabItem, viewModel: viewModel)
                        }
                    }
                }
            }
        }
        .onAppear {
            viewModel.analytics.track(event: .viewInfoTab)
            viewModel.isLoading = true
            viewModel.fetchDynamicContent()
        }
    }
}

// Cell
fileprivate struct InfoTabCell: View {
    let item: InfoTabItems
    @ObservedObject var viewModel: InfoTabViewModel
    var body: some View {
        VStack(spacing: 0) {
            switch item {
            case .products, .aboutUs, .contactUs:
                Widget(item.title, state: .loaded) {
                    Image(item.image)
                        .resizable()
                        .scaledToFill()
                }
                .padding(EdgeInsets(top: 0, leading: .spacingS, bottom: .spacingXs, trailing: .spacingS))
                .frame(maxWidth: 500, alignment: .center)
            case .frequentlyAskedQuestions:
                Widget(viewModel.frequentlyAskedTitle, state: .loaded) {
                    Image(item.image)
                        .resizable()
                        .scaledToFill()
                }
                .padding(EdgeInsets(top: 0, leading: .spacingS, bottom: .spacingXs, trailing: .spacingS))
                .frame(maxWidth: 500, alignment: .center)
            }
        }
    }
}

// Destinations
fileprivate struct InfoTabDestination: View {
    let item: InfoTabItems
    var body: some View {
        switch item {
        case .contactUs:
            EmptyView()
        case .aboutUs:
            AboutUsView(viewModel: AboutUsViewModel())
        case .products:
            SplitView(
                master: { ProductsView(viewModel: ProductsViewModel()) },
                detail: { ProductsDetailView(product: nil) }
            )
            .preferredDisplayMode(.allVisible)
        case .frequentlyAskedQuestions:
            EmptyView()
        }
    }
}


```
