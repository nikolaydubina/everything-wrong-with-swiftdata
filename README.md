# Everything Wrong with SwiftData

_nuances, missing features, strange API, things to watchout, and wishlist_

## Migrations

- on schema updates, SwiftData crashes if schema is not compatible (automatically upgradable)
- chain of migration scripts (classes) is necessary
- schema that did not have migration at first, then started to use migraitno scripts may have problems. decision about migration scripts is required from the start

## ModelContext

- SwiftData ModelContext is not Sendable, which makes it impossible to make async repositories (e.g. actors) that use SwiftData
- SwiftData does not automatically remove stale models inserted into ModelContext, manual deletion is necessary

## Query

- SwiftData cannot query by array member in field
- SwiftData cannot query dynamic properties (e.g. using Bidning and other dynamic data would not work)
- SwiftData cannot query from inside ObservableObject, only from View query is possible (might be possible if pass modelcontext though, but not sure if notifications will work)

### Query single ent by ID in View

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

### Query custom Comparable

SwiftData requires to compare with `.rawValue`, even if custom types are Codable, Comparable, Hashable

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

- there is no way to tell if sync is complete
- there is no way to tell if sync is in-progress
- there is no way to tell when sync will start
- there is no way to trigger sync manually
- only status of iCloud available is CloudKit Container `CKAccountStatus`
- SwiftData iCloud does not support unique IDs for objects
- schema migration has to be triggered manually from Development environment and promoted to Production

## Encoding

- SwiftData does crashes on decoding Decimal whose parent struct is optional. SwiftData cannot decode back to optional parents data that is not-optional (Decimal) inside a struct. `Could not cast value of type 'Swift.Optional<Any>' to '__C.NSDecimal'`
- SwiftData does not recognize `enum CodingKeys` in Codable. (defining explicity encode and decode methods does not help)
- SwiftData can handle deeply nested structs, as long as they are Codable. Nested data will be serialized automatically.
