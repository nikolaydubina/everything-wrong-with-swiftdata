# Everything Wrong with SwiftData

_nuances, missing features, strange API, things to watchout, and wishlist_

- SwiftData ModelContext is not Sendable, which makes it impossible to make async repositories (e.g. actors) that use SwiftData
- SwiftData does NOT automatically remove stale models inserted into modelContext, manual deletion is necessary

## Query

- SwiftData cannot query dynamic properties (e.g. using Bidning and other dynamic data would not work) (TODO: verify this)
- SwiftData cannot query from inside ObservableObject, only from View query is possible (might be possible if pass modelcontext though, but not sure if notifications will work)


### query single ent by id in view

SwiftData can query single item by ID passed into View is possible, but not convenient. construct query in init method. now because of SwiftData your view needs init method.
```swift
struct BookmarkButton: View {
    @Environment(\.modelContext) private var modelContext

    var productID: ProductID

    @Query private var allBookmarks: [ProductBookmark]

    init(productID: ProductID) {
        self.productID = productID
        _allBookmarks = Query(filter: #Predicate { $0.productID.rawValue == productID.rawValue })
    }
```

## Model

- SwiftData Model is not Sendable

## DataStore

- SwiftData DataStore fetch method is sync, which renders it impossible to make HTTP calls
- SwiftData DataStore fetch can detect which type (Model) is being requested to fetch, what is actually fetched is dynamic Swift language predicate that can be encoded to string

## iCloud

- SwiftData iCloud does not support unique IDs for objects
