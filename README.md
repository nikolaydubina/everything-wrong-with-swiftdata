# Everything Wrong with SwiftData

_nuances, missing features, strange API, things to watchout, and wishlist_

## Development & Production

- Development SwiftData may not use iCloud. to test iCloud at least TestFlight is required
- Development SwiftData builds are not equivalent to Production builds. Build that works in Development can crash on start with obscure stack trace in Production (e.g. Testflight).

## Migrations

- any bump in versioned schema (major, minor, patch) requires migration script
- on schema updates, SwiftData crashes if schema is not compatible (automatically upgradable)
- chain of migration scripts (classes) is necessary
- schema that did not have migration at first, then started to use migraitno scripts may have problems. decision about migration scripts is required from the start

## ModelContext

- SwiftData ModelContext is not Sendable, which makes it impossible to make async repositories (e.g. actors) that use SwiftData
- SwiftData does not automatically remove stale models inserted into ModelContext, manual deletion is necessary


Schema passed in `ModelConfiguration` inside `ModelContainer` will override Schema specified in `ModelContainer`. For example, non-versioned schema in `ModelConfiguration` will override verisioned schema in `ModelContainer` making schema non-versioned.

```swift
ModelContainer(
    for: Schema(versionedSchema: AppSchemaV1.self),
    configurations: [
        ModelConfiguration(
            "CloudData",
            // schema becomes non-versioned here!
            schema: Schema([
                UserSettingsModelV1.self,
            ]),
            isStoredInMemoryOnly: false,
            allowsSave: true,
            cloudKitDatabase: .private(Constant.cloudKitContainerIdentifier),
        ),
    ]
)
```

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

>[!CAUTION]
> iCloud can crash when query non-root fields (e.g. `.rawValue`)

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

- iCloud only supports lightweight migrations (which is not advertised in the guidelines from the beggining but discovered throught errors and crashes)
- iCloud has no static checks. crashes happen at runtime
- iCloud has limited Development environment checks. crashes happen in Production that did not happen in Development
- iCloud crashes when accessing non native Swift types (TODO: re-confirm this, this is very likely. past observation mixed with crahes due to nested field access)
- iCloud crashes when accessing nested fields in predicates
- iCloud schemas in Production cannot be deleted or changed. schemas are permanent and immutable.
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
