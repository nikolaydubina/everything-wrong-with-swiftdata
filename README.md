# Everything Wrong with SwiftData

_nuances, missing features, strange API, things to watchout, and wishlist_

- SwiftData cannot query from inside ObservableObject, only from View query is possible (might be possible if pass modelcontext though, but not sure if notifications will work)
- SwiftData ModelContext is not Sendable, which makes it impossible to make async repositories (e.g. actors) that use SwiftData
- SwiftData cannot query dynamic properties (e.g. using Bidning and other dynamic data would not work) (TODO: verify this)
- SwiftData does NOT automatically remove stale models inserted into modelContext, manual deletion is necessary

## DataStore

- SwiftData DataStore fetch method is sync, which renders it impossible to make HTTP calls
- SwiftData DataStore fetch can detect which type (Model) is being requested to fetch, what is actually fetched is dynamic Swift language predicate that can be encoded to string
