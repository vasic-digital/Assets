# Architecture -- Assets

## Purpose

Universal lazy asset loading framework for Go. Provides strategy-based resolution, filesystem storage, event-driven lifecycle notifications, and embedded default placeholders. Used by catalog-api for resolving and serving media cover art, thumbnails, and other assets on demand.

## Structure

```
pkg/
  asset/       Core types: Asset, ID, Type, Status
  store/       Storage interface + FileStore (filesystem) + MemoryStore (tests)
  resolver/    Strategy pattern: Resolver interface, ChainResolver, HTTPResolver, LocalFileResolver
  event/       EventBus interface + InMemoryBus for asset lifecycle notifications
  defaults/    Provider interface + EmbeddedProvider with //go:embed placeholder images
  manager/     Orchestrator: resolve + store + events + worker pool + functional options
```

## Key Components

- **`store.Store`** -- Get/Put/Delete/Exists for asset content bytes
- **`resolver.Resolver`** -- Name/CanResolve/Resolve/Priority for strategy-based resolution
- **`event.EventBus`** -- Publish/Subscribe for asset lifecycle events
- **`defaults.Provider`** -- GetDefault/Register for fallback content when assets are not yet resolved
- **`manager.Manager`** -- Orchestrator composing store, resolver chain, event bus, defaults, and a worker pool

## Data Flow

```
Request(asset) -> Manager -> ChainResolver (tries resolvers by priority)
                                |
                          HTTPResolver / LocalFileResolver
                                |
                          Store.Put(resolved content)
                                |
                          EventBus.Publish(AssetResolved)

Get(asset) -> Manager -> Store.Get()
                           |
                    (not found) -> defaults.Provider.GetDefault()
```

## Dependencies

- `github.com/google/uuid` -- UUID generation for asset IDs
- `github.com/stretchr/testify` -- Test assertions

## Testing Strategy

Table-driven tests with `testify`. MemoryStore used in tests instead of FileStore. Tests cover resolver chain priority ordering, event publication, default fallback behavior, and concurrent worker pool operations.
