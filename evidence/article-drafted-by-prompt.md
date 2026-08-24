# "Permanent" Doesn't Mean Forever: Five Things Developers Get Wrong About Walrus

A team moves their document store to Walrus. The documents matter, so they store them as permanent blobs — the flag says permanent, the data is important, the decision takes about four seconds. They buy 30 epochs, which at 14 days an epoch is a little over a year. Fourteen months later, reads start returning nothing. No key was lost. No one ran a delete. The storage simply reached its end epoch and the nodes discarded the slivers, and because an expired blob cannot be extended, the only remedy left was re-uploading data that Walrus had been holding as the canonical copy. The word "permanent" had told them who could delete the blob. It had said nothing about how long it would survive.

## 1. "Permanent" means undeletable, not eternal

This is the most expensive misreading in the system, and it is worth killing before anything else.

Walrus offers two dispositions for a stored blob. **Deletable** is the default: the owner of the Sui object can remove the blob before its storage expires. **Permanent** is the opt-in: nobody can remove it early, including the uploader who paid for it.

That is the entire contrast. It is a statement about authority — who holds the power to end this blob's life ahead of schedule — and not a statement about duration.

Both kinds expire. A permanent blob reaches the end of its paid epochs and becomes unavailable exactly like a deletable one. Choosing permanent buys you protection against your own delete call and against a compromised key holding the object. It does not buy you a single extra day of storage.

If you came from object storage, the intuition to discard is that durability and retention are the same property. On Walrus they are separate, and only one of them is bought with the permanent flag.

## 2. Storage is a lease, not a purchase

Here is the mental model that replaces it. You are not buying space. You are renting it, in fixed units, with a hard ceiling.

Storage is metered in epochs. A mainnet epoch is 14 days. A single purchase runs to a maximum of 53 epochs — roughly two years — and that ceiling applies to what you can hold at once, not to the blob's total possible lifetime.

To outlive the ceiling, you extend. Extension is a real operation, not a grace mechanism, and it comes with three edges that decide whether your renewal path works:

- **You must extend before expiry.** An expired blob cannot be extended at all. Once it lapses, re-uploading the bytes is the only route back.
- **There is no grace period.** A blob expires at the *beginning* of its end epoch, so a blob with end epoch 314 is already gone when 314 starts, not when it finishes. Off-by-one here costs you the data.
- **Extension requires the Sui object ID**, not the blob ID. See section 4 — this is where the two identifiers stop being trivia.

Frame this as renewable rather than temporary, because the framing determines what you build. "Temporary" produces a system that quietly loses data. "Renewable" produces a system with a monitored expiry and a scheduled extension job. The second one is not much more work, and it is the whole difference.

## 3. Decentralised is not the same as private

This is the section where a subtle error becomes a disclosure, so it is worth stating flatly.

**Walrus does not encrypt your data.** All blobs on Walrus are public by default. Anyone holding the blob ID can read the bytes back through any aggregator. There is no access control at the Walrus layer — none to configure, none to forget to configure.

What Walrus provides is availability and integrity: your data is there, and it is the data you stored. Confidentiality is not on that list, and no amount of decentralisation supplies it. Distribution across independent nodes is a property about *who can withhold* your data, not about *who can read* it.

If content needs to be private, you encrypt it client-side, before upload. **Seal** is the standard route — threshold encryption with on-chain access control — and the important structural point is that Seal sits *on top of* Walrus rather than inside it. Walrus Memory works exactly this way: memory content is encrypted with Seal before it ever reaches Walrus storage. The privacy belongs to the layer above; Walrus stores ciphertext and is none the wiser.

The rule that follows is short. Never put anything on Walrus you would not publish, unless it was encrypted before it left the client.

One consequence worth checking against your own threat model: deleting a deletable blob reclaims your storage but is a weak privacy tool, since identical content uploaded by someone else persists independently and cached copies are unaffected. [CHECK — supported by the managing-blobs documentation but not yet in project canon.]

## 4. Two identifiers, two jobs

Walrus hands you two IDs, and they are not interchangeable.

The **blob ID** is a URL-safe base64 string derived from the content itself. It is content-addressed and it is what you read with.

The **Sui object ID** is a `0x` hex string identifying the on-chain `Blob` object. It is unique per upload and it is what lifecycle operations act on — extend, delete, attributes.

Here is the consequence that makes it click: **upload the same file twice and you get one blob ID but two object IDs.** Same bytes, same content address, two separate on-chain objects with their own owners and their own expiry clocks.

Which means a system that stores only blob IDs can read its data and cannot renew it. That is section 2's failure arriving through section 4's door — the renewal path you build needs the object ID persisted alongside the content reference, and it is much cheaper to notice that now than at epoch 52.

## 5. You need two tokens

Storing on Walrus requires WAL and SUI, and having one is not enough.

WAL buys the storage resource — the right to occupy space across nodes for a number of epochs. SUI pays gas for the Sui transactions the flow depends on, since registration, certification, extension and deletion are all on-chain operations.

A wallet with plenty of WAL and no SUI fails at the first transaction, before any storage is bought — and it usually fails with a gas-coin error that never mentions WAL at all. Fund both, and expect the error message to point at the wrong one.

## Why the model looks like this

All five of these follow from one design decision.

Walrus does not copy your blob to every node. It erasure-codes it: the blob is encoded into slivers distributed across storage nodes, and it reconstructs from a subset of them even when some are faulty or offline. Redundancy without replication.

That is why the pricing works the way it does. For large blobs, storage overhead is a small multiple of the content size rather than n-fold — you are not paying for fifty copies because there aren't fifty copies. And because storage is an ongoing service that nodes perform rather than a write that completes, the meter runs in epochs. [CHECK — the erasure-coding-to-pricing link is canon; attributing the *epoch* model to the same cause is my inference, not established.]

The catch that matters for what you build: every blob also carries a fixed metadata overhead of roughly **64 MB regardless of its size**. Below about 10 MB, that fixed cost dominates the content cost entirely. A 4 KB JSON document and a 4 MB image cost approximately the same to store, which is why quilts exist — batching many small blobs amortises the per-blob overhead across all of them.

**On Monday:** persist the object ID next to every blob ID you store, or you have no renewal path. Put expiry monitoring in before you put data in, and remember the deadline is the *start* of the end epoch. Encrypt before upload if the content is not publishable. Fund WAL and SUI together. And if you are writing many small records, batch them — otherwise you are paying 64 MB of overhead to store a paragraph.

---

**~1,290 words.** Two `[CHECK]` markers where I went past canon rather than asserting: the delete-doesn't-guarantee-removal point in §3, and the erasure-coding→epoch-metering causal link in the close. Both are resolvable — the first is in the managing-blobs docs and could be filed as its own rule; the second I'd rather leave unclaimed than hand-wave, since "why epochs" is a design-rationale question canon doesn't answer.

Nothing written to memory this turn. Per the extract rule I record after a passage is final, not on first draft — say the word once you've read it and I'll file what it settles, plus the piece's title and thesis as a `[canon:decision]` and a chronicle entry for the drafting session.
