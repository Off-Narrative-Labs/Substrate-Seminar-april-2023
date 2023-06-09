# Tuxedo

Write UTXO-based Substrate Runtimes

---v

## Substrate Seminar

* Started 2019, zoom call, hosted by Joshy
* Migrated to Crowdcast
* Handed off to Dan Forbes
* Handed off to Sacha Lansky
* Migrated to Twitch
* Handed off to Lauren and Aaron

Notes:

Spend ~5 min talking about seminar itself and this new era that Lauren and Aaron are ushering in

Joshy tell story of very first seminar and what a shit show it was and how much seminar has grown and evolved.

---v

## Off-Narrative Labs

<div style="float: left; width: 33%">

Andrew
![](./coax1d.png)

[Github](https://github.com/coax1d/)

</div>
<div style="float: left; width: 33%">

Off Narrative Labs
![](./Off-Narrative-Labs.png)

[Github](https://github.com/Off-Narrative-Labs/)

</div>
<div style="float: left; width: 33%">

Joshy
![](./JoshOrndorff.jpg)

[Github](https://github.com/JoshOrndorff/)

</div>

Notes:

Andrew and Joshy each self-intro

For sure talk about meeting at the first academy, doing the UTXO final project, teaching together at the second academy where we met Lauren and Aaron.

---v

## About Tuxedo

Web 3 Grant: https://github.com/w3f/Grants-Program/blob/master/applications/tuxedo.md

Milestones:
- ✅ Core Tuxedo Functionality
- ✅ Template wallet
- 🏗️ Full Documentation and Tutorial

Notes:
This seminar is an opportunity for us to try out or beta test our plan for the docs and tutorial that we will deliver for The third milestone which we plan to submit in about two weeks.

---

# Substrate

![](./Substrate.svg)

Notes:

The client is a normal program that runs natively on a computer. It handles general purpose blockchain administration logic.

* Transaction Pool
* P2P Networking
* Fork Awareness and Management
* RPC Requests

The Runtime is a WebAssembly program that gets executed by the Client.

The client does not have any idea what the runtime does, concretely.
The runtime just implements some API that the client knows how to call.

---v

## Runtime APIs - Core

* [Rust Docs](https://paritytech.github.io/substrate/master/sp_api/trait.Core.html)
* For executing arbitrary blocks
```rust
pub trait Core<Block: BlockT> {
    fn execute_block(
        &self,
        block: Block
    ) -> Result<(), ApiError>;
}
```

---v
## Runtime APIs - BlockBuilder

* [Rust Docs](https://paritytech.github.io/substrate/master/sp_block_builder/trait.BlockBuilder.html)
* For authoring your own blocks

```rust
pub trait BlockBuilder<Block: BlockT> {
  fn initialize_block(
      &self,
      header: &Block::Header
  ) -> Result<(), ApiError>;
  fn apply_extrinsic(
      &self,
      extrinsic: Block::Extrinsic
  ) -> Result<ApplyExtrinsicResult, ApiError>;
  fn finalize_block(
      &self,
  ) -> Result<Block::Header, ApiError>;
}
```

Notes:
This is a slightly idealized version of the way the apis are actually implemented. I've omitted the versions of these functions. Learn more at https://paritytech.github.io/substrate/master/sp_api/trait.RuntimeApiInfo.html. I've also shown `initialize_block` in block builder instead of core. Learn more at https://substrate.stackexchange.com/questions/3128

There are also several other runtime APIs for things like off-chain workers and the transaction pool. These are interesting, but not that important and we're not going to dive into them right now. We've shown the main ones.

There are also things called host functions. These allow the runtime to call back into the client to get help with specific subtasks. These aren't really relevant at the level we're discussing here.

---v

## The Runtime

<img src="./nintendo-game-polkadot.png" style="height: 300px;" />
<img src="./nintendo-console.png" style="height: 300px;" />

<img src="./nintendo-game-moonbeam.png" class="fragment" style="height: 300px; position: absolute; left: 100px; top: 150px;" />
<img src="./nintendo-game-scratch.png" class="fragment" style="height: 300px; position: absolute; left: 150px; top: 200px;" />
<img src="./nintendo-game-dex.png" class="fragment" style="height: 300px; position: absolute; left: 200px; top: 250px;" />

Notes:

The Runtime represents the application that a particular blockchain runs.

Typically when you dive in to the runtime. This is where people start talking about FRAME and pallets.
But FRAME is just one of many ways to write a runtime. In fact, you can write a runtime entirely from scratch.
All you have to do is implement those runtime apis we saw. We do this at the academy.

* Write runtime from scratch
* Write runtime using FRAME
* Write runtime using Tuxedo
* Hopefully more frameworks to come

Blog analogy:

You can write your blog in plain html if you want to.
You can also use gatsby.
Or Hugo, or Jekyll, or drupal, or whatever.
There will be more blog frameworks in the future that aren't even existing yet.

---

# Accounts vs UTXOs

* Accounts - Great for reducing State bloat, not so great for privacy. Great for general computation, Not so great for parallelization 
* UTXOS - Great for privacy, Not so great for State bloat, Not so great in terms of general computation(for now!), great for parallelization  

---v

## Cryptocurrency Example

<div style="display: flex;" >
  <div style="flex: 45%; margin: 30px;">

  Accounts:

  | Address | Balance  |
  | ------- | -------- |
  | 0xA11ce | 50 coins |
  | 0xB0bbb | 10 coins |

  </div>
  <div style="flex: 55%; margin: 30px;">

  UTXOs:

  | Serial | Address | Amount   |
  | ------ | ------- | -------- |
  | 0x1111 | 0xA11ce | 50 coins |
  | 0x2222 | 0xB0bbb | 2 coins  |
  | 0x3333 | 0xB0bbb | 4 coins  |
  | 0x4444 | 0xB0bbb | 4 coins  |

  </div>
</div>

Notes:

First let's consider the example of cryptocurrency.

In the accounts model, each address is mapped to a balance. There is a special storage key for each address that holds the accounts balance. The balance is a single value that is the sum of all the accounts previous dealings.

In the UTXO model, the storage addresses are the coin's serial numbers (which are calculated based on the transaction that includes them). There is no single or global place to check an address's balance. You have to know about all the UTXOs and sum them up.

Having considered this motivating example, lets generalize the big picture differences between the models

---v

## Accounts Model

* Global storage items at known addresses
* On-chain logic must be deterministic
* Transactions execute on latest state
  * And they may put constraints on the state
* On-chain code calculates the final state

---v

## UTXO Model

* All state is local to individual outputs, addresses are based on transaction hash.
* Transactions are fully deterministic
* On-chain code may be non-deterministic
* Transaction specifies exact input state
* Transaction specifies exact final state
* On-chain code just checks that constraints are satisfied

---v

## UTXO for offchain worker/sidechains
* Since in the UTXO model you must in a any given input set only spend a single UTXO we can utilize this for performance.

---v

## UTXO sidechain

<img style="width: 600px" src="utxo_sidechains.svg" />

---v

## Accounts for offchain worker/sidechains

* In comparison it can be quite complicated for the accounts model in this regard.

---v
## Accounts sidechain

<img style="width: 600px" src="accounts_sidechains.svg" />

---v

## Zero-knowledge with UTXOS

* We have some amount of UTXOS (For someone) Utxo1(pubkey, val=1) Utxo2(pubkey, val=2) .. etc
* We want to prove that we can indeed spend 4 without revealing which utxos will be spent or our pubkey.
* So what we want to prove is:
    1. I control a particular key which can spend some number of utxos which can sum to 4
    2. The UTXO/s are infact valid and unspent
    3. The UTXO/s are atleast a value of 4

---

# <small>Order Book DEX Example</small>

* A Decentralized Exchange
* NOT an Automated Market Maker
* Users trade directly with each other

<div style="display: flex;" >
  <div style="flex: 45%;">

```json
{
  "side": "buy",
  "token_a": 150,
  "token_b": 200, 
}
```

  </div>
  <div style="flex: 45%;">

```json
{
  "side": "sell",
  "token_a": 160,
  "token_b": 190, 
}
```

  </div>
</div>

[More Fundamentals](https://blog.atani.com/dex-orderbook-vs-liquidity-pool/)

Notes:
Now let's consider how a more complex application might look in both frameworks.

We will live code on this project together later in the seminar. For now we'll do some design.

Centralized exchanges like Kraken or Poloniex have existed since Mt Gox. They typically operate as an order book. Customers place Orders which are agreements to make a trade of some token for some other token at a given price. The Exchange matches customer orders, and usually gets a profit on the difference, plus sometimes an explicit trading fee.

Decentralized exchanges operate on-chain and eliminate the need to trust the centralized exchange operator. They exist on many chains like Ethereum. On Ethereum they are mostly based on automated market makers. This design is well suited for an accounts model. When two users interact with the DEX around the same time, they don't know which one will be applied first, and therefore, don't know which one will get the better price.

This is not well suited for UTXOs. When sending UTXO transactions, you specify the exact pre-state that you are going to consume. Therefore, you know the exact price you will get and the exact outcome of the transaction. However, it is very inconvenient for users because they are constantly racing to consume the state of the AMM.

In a UTXO setting, it makes more sense to use the order book model that centralized exchanges previously took.

---v

## Sorting the Order Book

![](./depth-chart.png)

Notes:

When you run the order book, it is useful to keep the orders sorted so that you can see when the buys and sells overlap and matches can be made.

If you want to do your match-making on-chain you will need to store your orders this way. This may be expensive. OTOH, you can move the computation of making a match offchain and allow anyone who happens to spot a match, submit a transaction and get a reward. We will take this second approach.

It is common, in the UTXO model, that you can move computation offchain and only require verification on-chain in a way that feels natural.

---v
## Order Book Actions

<div style="display: flex;" >
  <div style="flex: 33%;">

Core

* Place Order
* Make Match

</div>
<div style="flex: 33%;">

Enhancement

* Cancel Order
* Take Order

</div>
<div style="flex: 33%;">

Development

* Mint Token A
* Mint Token B

</div>
</div>

Notes:

The primary actions in the dex are placing orders, and settling orders by matching them with compatible counterparts. A few nice-to-haves that make it more user-friendly would include
* The ability to cancel an order that you are no longer interested in. As a workaround, you could make an exact opposite order and match them (if you have enough collateral)
* The ability to take the exact opposite position as an existing order. This is basically an atomic action for making the opposite order and matching them together.
* The ability to partially fill orders. This is essential for a real-life dex, but for learning, we can skip it.

For the sake of testing the chain out, we need an easy way to get our hands on some of each of the tokens that we are trading. For that, we will have a way to mint each of them.

---

# Tuxedo

A **framework** for writing **Substrate Runtimes** that are based on the **UTXO model**.

* Github: https://github.com/Off-Narrative-Labs/Tuxedo/
* Rustdocs: https://off-narrative-labs.github.io/Tuxedo/tuxedo_core/index.html

---v

## The Transaction Type

```rust
pub struct Transaction<V, C> {
    pub inputs: Vec<Input>,
    pub outputs: Vec<Output<V>>,
    pub checker: C,
}
```
[Rustdocs](https://off-narrative-labs.github.io/Tuxedo/tuxedo_core/types/struct.Transaction.html)

Notes:

Lists all the exact state to be consumed. Lists all the exact state to be produced. constraint checker is kind of like FRAME's `Call` enum.

When we check a transaction:
* No duplicate inputs
* All inputs exist
* All inputs verifiers are properly redeemed
* No conflicting outputs exist
* The constraint checker is satisfied

---v

## The Verifier Trait

```rust
pub trait Verifier: Debug + Encode + Decode + Clone {
    fn verify(&self, simplified_tx: &[u8], redeemer: &[u8]) -> bool;
}
```

[Rustdocs](https://off-narrative-labs.github.io/Tuxedo/tuxedo_core/verifier/trait.Verifier.html)

Notes:

The verifier operates on the level of an individual input. Each input's verifier is checked independently. The verifier typically represents a cryptographic signature, but there is room for creativity here.

---v

## The ConstraintChecker Trait

```rust
pub trait ConstraintChecker: Debug + Encode + Decode + Clone {
    /// The error type that this constraint checker may return
    type Error: Debug;

    /// The actual check validation logic
    fn check<V: Verifier>(
        &self,
        inputs: &[Output<V>],
        outputs: &[Output<V>],
    ) -> Result<TransactionPriority, Self::Error>;
}
```

[Rustdocs](https://off-narrative-labs.github.io/Tuxedo/tuxedo_core/constraint_checker/trait.ConstraintChecker.html)

---v

## Simple Constraint Checker

```rust
pub trait SimpleConstraintChecker: Debug + Encode + Decode + Clone {
    /// The error type that this constraint checker may return
    type Error: Debug;

    /// The actual check validation logic
    fn check(
        &self,
        input_data: &[DynamicallyTypedData],
        output_data: &[DynamicallyTypedData],
    ) -> Result<TransactionPriority, Self::Error>;
}
```

[Rustdocs](https://off-narrative-labs.github.io/Tuxedo/tuxedo_core/constraint_checker/trait.SimpleConstraintChecker.html)

Notes:

This trait is very similar to constraint checker, but provides less contextual information to the implementor.
This trait only provides the dynamically typed data enclosed in each output.
It does not contain the verifiers.
In many cases this information is sufficient and the simpler trait is preferable.
In other cases you may want to enforce constraints on the relationship between verifiers.

---

## Tuxedo Pieces

Typically a collection a types and constraint checkers. Sometimes some verifiers.

Notes:

Analogous to FRAME pallets.

---v

## Aggregation

```rust
#[tuxedo_verifier]
pub enum OuterVerifier {
    SigCheck(SigCheck),
    UpForGrabs(UpForGrabs),
    ThresholdMultiSignature(ThresholdMultiSignature),
}
```

[`tuxedo_verifier` rustdocs](https://off-narrative-labs.github.io/Tuxedo/tuxedo_core/attr.tuxedo_verifier.html)

[`tuxedo-constraint-checker` rustdocs](https://off-narrative-labs.github.io/Tuxedo/tuxedo_core/attr.tuxedo_constraint_checker.html)

Notes:
This code snippet is from the tuxedo template runtime.
It shows how we aggregate out verifiers into an outer verifier.

This is similar to how FRAME works in many ways, but also different.

There are many outer types. We make the developer explicitly write each outer enum. This is a little more code
but a lot more transparency. In Tuxedo, the aggregation can happen recursively. This allows for packaging collections
of related pieces and configurations together.

---v

## The Executive Type

The `Executive` type is provided by Tuxedo core and implements all the Runtime APIs

[Rustdocs](https://off-narrative-labs.github.io/Tuxedo/tuxedo_core/struct.Executive.html)

Notes:

---

# Code Walkthrough

Start at the root. Skim the readme. Point out the four main directories.
* Tuxedo-core
* Tuxedo-template-runtime
* node
* wallet

Show the node VERY BRIEFLY only to make the point that it is the typical Substrate node template.
Show the core. Show where it implements the APIs. Talk about the dynamic typing system.
Show the Verifier and ConstraintChecker definitions.
Mention what the wallet is, but don't show the code unless we need to fill time.
Show the runtime. Show that there are many pieces.
Look at the runtime aggregation.


Notes:
At this point we will look away from the slides and shift to the Tuxedo repo. Nonetheless,
I'll sketch the tour route here.

---

# Demo

Run a local node, use the wallet to send some coins, etc TODO better script. Maybe just follow the script in the
wallet readme.

Notes:
This wallet is intended to be extensible so that any tuxedo-based application can implement some hooks and the wallet can track the corresponding state.
We aren't quite there yet. Right now it works well with the cryptocurrency, but some of the logic is still intermixed in the core, and the APIS need better definitions.

---

# Live Code

Let's try to implement the ORder Book DEX we discussed.

This will written out as a full tutorial to complete our third milestone.

Notes:

Would be cool to do a runtime upgrade, but we have never tried that. I'm worried it won't work because we haven't set up opaque extrinsics properly yet. But if we don't change any types. I guess it should work. Let's see if we have time to try it out or not.

Worst case, we start a new `--dev` chain with the new runtime installed and all is well.


---
# Calls to Action

* Try Tuxedo
* Fork / Clone / Star our repo
* Open issues and PRs

Let us know what you want from Tuxedo

Notes:

We would be happy to help you write your runtime until you get the hang of it.
