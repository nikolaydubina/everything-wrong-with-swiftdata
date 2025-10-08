# Everything Wrong with SwiftData

_nuances, missing features, strange API, things to watchout, and wishlist_

## Development & Production

Development SwiftData builds are not equivalent to Production builds.

Following works in Development, but crashes on start in Production.

```swift
import SwiftData
import SwiftUI

struct DynamicQuery<Model: PersistentModel, Content: View>: View {
    let descriptor: FetchDescriptor<Model>
    let content: ([Model]) -> Content

    @Query private var results: [Model]

    init(descriptor: FetchDescriptor<Model>, @ViewBuilder content: @escaping ([Model]) -> Content) {
        self.descriptor = descriptor
        self.content = content
        _results = Query(descriptor)
    }

    var body: some View {
        content(results)
    }
}
```

## Migrations

- on schema updates, SwiftData crashes if schema is not compatible (automatically upgradable)
- chain of migration scripts (classes) is necessary
- schema that did not have migration at first, then started to use migraitno scripts may have problems. decision about migration scripts is required from the start

## ModelContext

- SwiftData ModelContext is not Sendable, which makes it impossible to make async repositories (e.g. actors) that use SwiftData
- SwiftData does not automatically remove stale models inserted into ModelContext, manual deletion is necessary

## Query

- SwiftData cannot query in JSON encoded Data (exposed via cached private var and getters)
- SwiftData cannot query by array member in field when of Array of custom type, even if it is Codable, Hashable, Equitable, RawRepresentable. whole array becomes Bytes in index
- SwiftData cannot query by array member `Can't have a non-relationship collection element in a subquerySUBQUERY`
- SwiftData cannot query dynamic properties (e.g. using Bidning and other dynamic data would not work)

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

- SwiftData crashes on decoding Decimal whose parent struct is optional. SwiftData cannot decode back to optional parents data that is not-optional (Decimal) inside a struct. `Could not cast value of type 'Swift.Optional<Any>' to '__C.NSDecimal'`
- SwiftData does not recognize `enum CodingKeys` in Codable. (defining explicity encode and decode methods does not help)
- SwiftData can try to handle nested structs. Nested data will be serialized automatically. However, serialization ignores CodingKeys, custom encode methods, will try to cast types directly and will crash process instead of throwing exception.
