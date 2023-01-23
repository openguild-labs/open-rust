---
title: Runtime Migrations Try Runtime
description: A design system to use within slides of a presentation
instructors: ["Kian Paimani"]
teaching-assistants: ["Sacha Lansky"]
---

# Runtime Migrations and Try Runtime

---

## Runtime upgrades...

### _and how to survive them_

---

### _At the end of this lecture, you will be able to:_

- Justify when runtime migrations are needed.
- Write a the full a runtime upgrade that includes migrations, end-to-end.
- Test runtime upgrades before executing on a network using `try-runtime` and `remote-externalities`.

Notes:

Reference material:
https://docs.google.com/presentation/d/1hr3fiqOI0JlXw0ISs8uV9BXiDQ5mGOQLc3b_yWK6cxU/edit#slide=id.g43d9ae013f_0_82
https://www.crowdcast.io/e/substrate-seminar/41

---

## When is a Migration Required?

---v

### When is a Migration Required?

- In a typical runtime upgrade, you typically only replace `:code:`. This is _**Runtime Upgrade**_.
- If you change the _storage layout_, then this is also a _**Runtime Migration**_.

> Anything that changes **encoding** is a migration!

---v

### When is a Migration Required?

```rust
#[pallet::storage]
pub type FooValue = StorageValue<_, Foo>;
```

```rust
// old
pub struct Foo(u32)
// new
pub struct Foo(u64)
```

A clear migration.

<!-- .element: class="fragment" -->

---v

### When is a Migration Required?

```rust
#[pallet::storage]
pub type FooValue = StorageValue<_, Foo>;
```

```rust
// old
pub struct Foo(u32)
// new
pub struct Foo(i32)
// or
pub struct Foo(u16, u16)
```

The data still _fits_, but the _interpretations_ is almost certainly different!

<!-- .element: class="fragment" -->

---v

### When is a Migration Required?

```rust
#[pallet::storage]
pub type FooValue = StorageValue<_, Foo>;
```

```rust
// old
pub struct Foo { a: u32, b: u32 }
// new
pub struct Foo { a: u32, b: u32, c: u32 }
```

This is still a migration, because `Foo`'s decoding changed.

<!-- .element: class="fragment" -->

---v

### When is a Migration Required?

```rust
#[pallet::storage]
pub type FooValue = StorageValue<_, Foo>;
```

```rust
// old
pub struct Foo { a: u32, b: u32 }
// new
pub struct Foo { a: u32, b: u32, c: PhantomData<_> }
```

If for whatever reason `c` has a type that its encoding is like `()`, then this would work.

<!-- .element: class="fragment" -->

---v

### When is a Migration Required?

```rust
#[pallet::storage]
pub type FooValue = StorageValue<_, Foo>;
```

```rust
  // old
  pub enum Foo { A(u32), B(u32) }
  // new
  pub enum Foo { A(u32), B(u32), C(u128) }
```

Extending an enum is even more interesting, because if you add the variant to the end, no migration is needed.

<!-- .element: class="fragment" -->

Assuming that no value is initialized with `C`, this is _not_ a migration.

<!-- .element: class="fragment" -->

---v

### When is a Migration Required?

```rust
#[pallet::storage]
pub type FooValue = StorageValue<_, Foo>;
```

```rust
// old
pub enum Foo { A(u32), B(u32) }
// new
pub enum Foo { A(u32), C(u128), B(u32) }
```

You probably _never_ want to do this, but it is a migration.

<!-- .element: class="fragment" -->

---v

### 🦀 Rust Recall 🦀

Enums are encoded as the variant enum, followed by the inner data:

- The order matters!
- Enums that implement `Encode` cannot have more than 255 variants.

---v

### When is a Migration Required?

```rust
#[pallet::storage]
pub type FooValue = StorageValue<_, u32>;
```

```rust
// new
#[pallet::storage]
pub type BarValue = StorageValue<_, u32>;
```

- So far everything is changing the _value_ format.<br>

<div>

- The _key_ changing is also a migration!

</div>

<!-- .element: class="fragment" -->

---v

### When is a Migration Required?

```rust
#[pallet::storage]
pub type FooValue = StorageValue<_, u32>;
```

```rust
// new
#[pallet::storage_prefix = "FooValue"]
#[pallet::storage]
pub type I_can_NOW_BE_renamEd_hahAA = StorageValue<_, u32>;
```

Handy macro if you must rename a storage type.<br>
This does _not_ require a migration.

<!-- .element: class="fragment" -->

---

## Writing Runtime Migrations

- Now that we know how to detect if a storage change is a **migration**, let's see how we write one.

---v

### Writing Runtime Migrations

- Once you upgrade a runtime, the code is expecting the data to be in a new format.
- Any `on_initialize` or transaction might fail decoding data, and potentially `panic!`

---v

### Writing Runtime Migrations

- We need a **_hook_** that is executed **ONCE** as a part of the new runtime...
- But before **ANY** other code (on_initialize, any transaction) with the new runtime is migrated.

> This is `OnRuntimeUpgrade`.

<!-- .element: class="fragment" -->

---v

### Writing Runtime Migrations

- Optional activity: Go into `executive` and `system`, and find out how `OnRuntimeUpgrade` is called
  only when the code changes!

---

## Pallet Internal Migrations

---v

### Pallet Internal Migrations

One way to write a migration is to write it inside the pallet.

```rust
#[pallet::hooks]
impl<T: Config> Hooks<T::BlockNumber> for Pallet<T> {
  fn on_runtime_upgrade() -> Weight {
    migrate_stuff_and_things_here_and_there<T>();
  }
}
```

> This approach is likely to be deprecated and is no longer practiced within Parity either.

---v

### Pallet Internal Migrations

```rust
#[pallet::hooks]
impl<T: Config> Hooks<T::BlockNumber> for Pallet<T> {
  fn on_runtime_upgrade() -> Weight {
    if guard_that_stuff_has_not_been_migrated() {
      migrate_stuff_and_things_here_and_there<T>();
    } else {
      // nada
    }
  }
}
```

If you execute `migrate_stuff_and_things_here_and_there` twice as well, then you are doomed 😫.

---v

### Pallet Internal Migrations

**Historically**, something like this was used:

```rust
#[derive(Encode, Decode, ...)]
enum StorageVersion {
  V1, V2, V3, // add a new variant with each version
}

#[pallet::storage]
pub type Version = StorageValue<_, StorageVersion>;

#[pallet::hooks]
impl<T: Config> Hooks<T::BlockNumber> for Pallet<T> {
  fn on_runtime_upgrade() -> Weight {
    if let StorageVersion::V2 = Version::<T>::get() {
      // do migration
      Version::<T>::put(StorageVersion::V3);
    } else {
      // nada
    }
  }
}
```

---v

### Pallet Internal Migrations

FRAME introduced macros to manage migrations: `#[pallet::storage_version]`.

```rust
// your current storage version.
const STORAGE_VERSION: StorageVersion = StorageVersion::new(2);

#[pallet::pallet]
#[pallet::storage_version(STORAGE_VERSION)]
pub struct Pallet<T>(_);
```

This adds two function to the `Pallet<_>` struct:

```rust
Pallet::<T>::current_storage_version()
Pallet::<T>::on_chain_storage_version()
```

---v

### Pallet Internal Migrations

```rust
#[pallet::hooks]
impl<T: Config> Hooks<T::BlockNumber> for Pallet<T> {
  fn on_runtime_upgrade() -> Weight {
    let current = Pallet::<T>::current_storage_version();
  	let onchain = Pallet::<T>::on_chain_storage_version();

		if current == 1 && onchain == 0 {
      // do stuff
    } else {
      current.put::<Pallet<T>>();
    }
  }
}

```

Stores the version as u16 in `twox(pallet_name) ++ twox(:__STORAGE_VERSION__:)`.

Go checkout the code!

<!-- TODO add link to code -->

---

## External Migrations

---v

### External Migrations

- Managing migrations within a pallet could be hard.
- The `#[pallet::hooks]` does nothing more than implementing `OnRuntimeUpgrade` for `Pallet<_>`.
- Every runtime can explicitly pass anything that implements `OnRuntimeUpgrade` to `Executive`.
- End of the day, Executive does:
  - `<(COnRuntimeUpgrade, AllPalletsWithSystem) as OnRuntimeUpgrade>::on_runtime_upgrade()`.

---v

### External Migrations

- The main point of external migrations is making it more clear:
- "_What migrations did exactly execute on upgrade to spec_version xxx_"

---

## Migration Best Practices

---v

### Migration Best Practices

- Expose your migration as a standalone function or struct implementing `OnRuntimeUpgrade` inside a `pub mod v<version_number>`.

- Guard the code of the migration with `pallet::storage_version`

  - Don't forget to write the new version!

---v

### Migration Best Practices

- Pass it to the runtime per-release.

- Bonus: Do your best to make sure your migrations are 100% independent of the state/config of the pallet.

  - Main trick is to not depend on `<T: Config>`
    - Is it clear why?
    - Only relevant if others use your pallet as well.

---v

### Migration Best Practices

- Full example: TODO: elections-phragmen

---

### Utilities in `frame-support`.

- `translate` methods:
  - For `StorageValue`, `StorageMap`, etc.
- https://paritytech.github.io/substrate/master/frame_support/storage/migration/index.html
- But generally don't restrict yourself. There are lots of ways to write migration code.

<!-- TODO expand section? -->

---

## Case Studies

1. The day we destroyed all balances in Polkadot
2. First ever migration (`pallet-elections-phragmen`)
3. Fairly independent migrations in `elections-phragmen`;

<!-- TODO expand section? -->

---

## Testing Upgrades

---v

### Testing Upgrades

`try-runtime` + `RemoteExternalities` allow you to examine and test a runtime in detail with a high degree of control over the environment.

It is meant to try things out, and inspired by traits like `TryFrom`, the name `TryRuntime` was chosen.

---v

### Testing Upgrades

Recall: The runtime communicates with the client via host.
An environment that provides these host functions is called `Externalities`.

One example of which is `TestExternalities`, which you have already seen.
Moreover, the client communicates with the runtime via runtime APIs.

---v

### Testing Upgrades: `remote-externalities`

`remote-externalities` ia a builder pattern that loads the state of a live chain inside `TestExternalities`.

```rust
let mut ext = Builder::<Block>::new()
  .mode(Mode::Online(OnlineConfig {
  	transport: "wss://rpc.polkadot.io",
  	pallets: vec!["PalletA", "PalletB", "PalletC", "RandomPrefix"],
  	..Default::default()
  }))
  .build()
  .await
  .unwrap();
```

Reading all this data over RPC can be slow!

---v

### Testing Upgrades: `remote-externalities`

`remote-externalities` supports:

- Custom prefixes -> Read a specific pallet
- Injecting custom keys -> Read `:code:` as well.
- Injecting custom key-values -> Overwrite `:code:` with `0x00`!
- Reading child-tree data -> Relevant for crowdloan pallet etc.
- Caching everything in disk for repeated use.

---v

### Testing Upgrades: `remote-externalities`

`remote-externalities` is in itself a very useful tool to:

- Go back in time and re-running some code.
- Writing `remote-tests`, like `pallet-bags-list`.

<!-- TODO more on bags list? -->

---v

### Testing Upgrades: `try-runtime`

`try-runtime` allows execution `OnRuntimeUpgrade` code of a new runtime, on top of a real chain's data.

It's lightweight, and independent of the substrate client.

---v

### Testing Upgrades: `try-runtime`

`try-runtime` is all inspired by the benchmarking pipeline, and it they both work very similar:

- `StateMachine` is capable of calling into any runtime api, given that it has some `impl Externalities`.
- `remote-externalities` will be the `Externalities` implementation.
- Custom feature gated runtime apis that execute runtime migrations, and more.

<!-- TODO: figure for this, runtime, ext, state machine. -->

---

## `try-runtime` exercise: Testing a Migration

---v

### `try-runtime` exercise: Testing a Migration

<!-- TODO -->

---

## `try-runtime` exercise: Additional Hooks

---v

### `try-runtime` exercise: Additional Hooks

- `pre_upgrade` and `post_upgrade`.

<!-- TODO -->

---

<!-- TODO Kian - below this line is unfinished -->

### Be Aware: Runtime Against Which You Run

- The only detail of try-runtime that is worth elaborating further is understanding **exactly which
  runtime you are using**.

- You always run try-runtime linked to a runtime, i.e. your local code-base. In Substrate, this is
  `node-runtime`, in Polkadot it is `polkadot-runtime`, `kusama-runtime` etc. This is (analogous to)
  your **native runtime**.

- You could also be fetching some code from `:code:` from the remote node to which you are
  connecting. This is analogous to the **wasm runtime**. If you don't fetch this, the one in the
  genesis state of your local chain will be used e.g. `polkadot-dev`.

- A try-runtime execution, similar to the main client has a `execution` flag:
  - `wasm`: always use **wasm runtime**.
  - `native`: uses native runtime IFF spec versions match, else wasm.

> Q: You want to re-execute an old block, but you also want to add some logs to the code, How would
> you do this both for wasm and native execution?

> A: TODO

### `try-runtime`: What Else?

- `try-runtime` can do many more things.
- Subject to change, and well documented, so no reason to go over now.

- But just so you know:
  - Execute old blocks.
  - Execute `OffchainWorker` of old blocks.
  - **Follow the chain**
    - Not really used yet, but can be a very powerful tool
    - "runtime burn-in".
  - Soon: `SanityChecks`.

---

## Exercises

- Find the storage version of nomination pools pallet in Kusama.
- Give them a poorly written migration code, and try and fix it. Things they need to fix:
  - The migration depends on `<T: Config>`
  - Does not manage version properly
  - is hardcoded in the pallet.
- Re-execute the block at which the runtime went OOM in May 25th 2021 Polkadot.