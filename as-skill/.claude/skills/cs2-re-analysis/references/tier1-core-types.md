# Tier1 Core Type Index

Use this reference when a CS2 / Source 2 binary shows an unfamiliar container, allocator, string, handle, buffer, refcounted object, delegate, or KeyValues-like object.

This is an index, not proof.
Confirm every candidate against the active backend evidence and the user's `hl2sdk-cs2/public/tier1/<header>` file before applying a type or rename.

## Contents

- Workflow
- Linear Storage
- Associative Storage
- Lists, Handles, And References
- Strings, Symbols, And Tokens
- Buffers And Serialization
- Lifetime, Allocation, And Callbacks
- Console And Commands

## Workflow

1. Match the binary shape first: field layout, accessors, element stride, allocation behavior, bounds checks, handles, virtual calls, and xrefs.
2. Use this index to pick likely SDK headers and names.
3. Read the matching SDK header from the user's `hl2sdk-cs2` path.
4. Apply the SDK type only when the layout and use sites match.
5. If the family matches but template parameters or element type are unclear, comment as `<SDK type>-like` and record the uncertainty.

## Linear Storage

- `utlvector.h`: `CUtlVector`, `CUtlBlockVector`, `CUtlVectorFixed`, `CUtlVectorFixedGrowable`, `CUtlVectorConservative`, `CUtlVectorRawAllocator`, `CUtlVectorUltraConservative`, `CUtlStringList`.
- `utlleanvector.h`: `CUtlLeanVector`, `CUtlLeanVectorFixedGrowable`.
- `utlmemory.h`: `CUtlMemory`, `CUtlMemoryFixedGrowable`, `CUtlMemoryFixed`, `CUtlMemoryConservative`, `CUtlMemoryAligned`, `CUtlMemory_RawAllocator`.
- `utlblockmemory.h`: `CUtlBlockMemory`.
- `utlfixedmemory.h`: `CUtlFixedMemory`.
- `utlqueue.h`: `CUtlQueue`, `CUtlQueueFixed`.
- `utlstack.h`: `CUtlStack`.
- `utlpriorityqueue.h`: `CUtlPriorityQueue`.
- `utlsoacontainer.h`: SOA-style container layouts.

Recognition clues:

- `CUtlVector` commonly has memory/allocation state plus a count.
- `CUtlLeanVectorBase` has `m_nCount`, `m_nAllocated`, and `m_pElements`.
- Do not call a compact `{count, data}`-looking layout custom until checking for a hidden allocation/index field nearby.
- Fixed/growable variants may store inline elements before falling back to heap memory.

## Associative Storage

- `utlrbtree.h`: `CUtlRBTree`, `CUtlFixedRBTree`, `CUtlBlockRBTree`, `UtlRBTreeNode_t`, `UtlRBTreeLinks_t`.
- `utlmap.h`: `CUtlMap`, `CUtlOrderedMap`.
- `utldict.h`: `CUtlDict`.
- `utlhash.h`: `CUtlHash`, `CUtlHashFast`, `CUtlHashFixed`, `CUtlScalarHash`.
- `utlhashdict.h`: `CUtlHashDict`.
- `utlhashtable.h`: `CUtlHashtable`, `CUtlStableHashtable`.
- `utltshash.h`: `CUtlTSHash`.
- `utlbidirectionalset.h`: bidirectional set maps.

Recognition clues:

- Tree containers usually expose node links, parent/left/right indexes, color/state bits, and a root index.
- Hash containers usually expose buckets, handles, keys or hashes, and collision chains.
- Map/dict wrappers often hide an underlying tree or hash container.

## Lists, Handles, And References

- `utllinkedlist.h`: `CUtlLinkedList`, `CUtlFixedLinkedList`, `CUtlBlockLinkedList`, `CUtlPtrLinkedList`.
- `utlintrusivelist.h`: `CUtlIntrusiveList`, intrusive double lists, tail-pointer variants.
- `utlhandletable.h`: `CUtlHandleTable`, `CUtlHandle`.
- `utlobjectreference.h`: `CUtlReference`, `CUtlReferenceList`, `CUtlReferenceVector`.

Recognition clues:

- Intrusive lists store links inside the element.
- Handle tables usually combine index/serial validation with table lookup.
- Object references often pair intrusive list membership with pointer lifetime management.

## Strings, Symbols, And Tokens

- `utlstring.h`: `CUtlString`, `CUtlConstString`, `CUtlConstWideString`.
- `bufferstring.h`: `CBufferString`, `CBufferStringN`, `CBufferStringGrowable`.
- `utlstringtoken.h`: `CUtlStringToken`.
- `utlsymbol.h`: `CUtlSymbol`, `CUtlSymbolTable`, `CUtlSymbolTableMT`, `CUtlFilenameSymbolTable`.
- `utlsymbollarge.h`: `CUtlSymbolLarge`, `CUtlSymbolTableLarge`, case-insensitive and MT variants.
- `UtlStringMap.h`: `CUtlStringMap`.

Recognition clues:

- String wrappers may collapse to pointer-sized storage in optimized decompiler output.
- Token/symbol types often compare or hash integers while resolving names through a table.
- Do not rename a string-like field from use of `char *` alone; require ownership or accessor evidence.

## Buffers And Serialization

- `utlbuffer.h`: `CUtlBuffer`, `CUtlInplaceBuffer`, `CUtlCharConversion`.
- `utlbinaryblock.h`: `CUtlBinaryBlock`.
- `bitbuf.h`: `bf_read`, `bf_write`, fixed/static write buffers.
- `KeyValues.h`: `KeyValues`, `KeyValues::AutoDelete`, unpack helpers.
- `keyvalues3.h`: `KeyValues3`, `CKeyValues3Cluster`.

Recognition clues:

- Buffer types usually track base pointer, put/get cursors, flags, and allocation size.
- Bit buffers use bit-level cursor math and bounds checks.
- KeyValues objects often combine type tags, names, value unions, and child/peer traversal.

## Lifetime, Allocation, And Callbacks

- `refcount.h`: `CRefPtr`, `CRefMT`, `CRefST`, `CRefCountService*`, `CRefCounted*`.
- `smartptr.h`: refcount accessors.
- `mempool.h`: `CUtlMemoryPoolBase`, `CUtlMemoryPool`, `CUtlMemoryPoolMT`.
- `memstack.h`: `CUtlMemoryStack`.
- `memblockallocator.h`: `CUtlMemoryBlockAllocator`.
- `rawallocator.h`: raw allocator wrappers.
- `utldelegate.h` and `utldelegateimpl.h`: `CUtlDelegate`, closure/delegate storage.
- `functors.h`: functor callbacks, member-function proxies, late-bound pointers.

Recognition clues:

- Refcounted objects show AddRef/Release-style increments, decrements, and destruction on zero.
- Pools and stacks often use block headers, free lists, bump pointers, or fixed-size chunks.
- Delegates/functors often store object pointer plus function pointer or closure data.

## Console And Commands

- `convar.h`: `ConVarRef`, `ConVarRefAbstract`, `CConVar`, `ConCommand`, `ConCommandRef`, creation records, command callbacks, typed value traits.
- `CommandBuffer.h`: `CCommandBuffer`.

Recognition clues:

- ConVar/ConCommand registration usually has static creation records, name/help strings, flags, callbacks, and typed value storage.
- Prefer the dedicated ConVar pattern in `SKILL.md` when matching cvar registration globals.
