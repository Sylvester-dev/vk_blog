---
title: Storage VS Memory VS Calldata
tags: [blockchain]
date: 2022-03-13T05:25:44.226Z
path: blog/about-variables
cover: ./preview.png
excerpt: Every function call uses gas in EVM; every bit of savings helps when summing over the potential lifetime of total function invocations.

---

### Why we need to worry about using it?

Whenever you are writing smart contracts in Solidity, you must be cognizant of how your variables and data are handled by the EVM. The choices you make will influence, among other things, gas costs — to call your functions or deploy your contract — as well as storage layout.

Given that every bit of block space in Ethereum is highly valued (hence the cost of Eth and Gas), this heavily influences the efficiency of your code and resultant contract, which hopefully will be invoked frequently. Every function call uses gas; every bit of savings helps when summing over the potential lifetime of total function invocations.

### Storage

``Storage`` is the easiest to grasp — it is where all state variables are stored. Because state can be altered in a contract (for example, within a function), storage variables must be mutable. However, their location is persistent, and they are stored on the blockchain.

State variables in storage are arranged in a compact way — if possible, multiple values will occupy the same storage slot. Besides the special cases of dynamically sized arrays and structs, other variables are packed together in blocks of 32 bytes.

If these variables are less than 32 bytes, they will be combined to occupy the same slot. Otherwise, they will be pushed onto the next storage slot. The data is stored contiguously (ie, one after the other), starting from the 0 slot (to slots 1, 2, 3, etc), in order of their declaration in the contract.

Dynamic arrays and structs always occupy a new slot, and any variables following them will also be initialized to start a new storage slot.

Because the size of both dynamic arrays and structs are unknown a priori (ie until you assign them later in your contract) they cannot be stored with their data in between other state variables. Instead, they are assumed to take up 32 bytes, and the elements within them are stored starting at a separate storage slot that is computed using a Keccak-256 hash.

However, constant state variables are not saved into a storage slot. Rather, they are injected directly into the contract bytecode — whenever those variables are read, the contract automatically switches them out for their assigned constant value. 

### Memory

``Memory`` is reserved for variables that are defined within the scope of a function. They only persist while a function is called, and thus are temporary variables that cannot be accessed outside this scope (ie anywhere else in your contract besides within that function). However, they are mutable within that function.

Solidity reserves four 32-byte slots for memory, with specific byte ranges, consisting of: 1) 64-byte scratch space for hashing methods; 2) 32 bytes for currently allocated memory size, which is the free memory pointer where Solidity always places new objects; and 3) a 32-byte zero slot — which is used as the initial value for dynamic memory arrays and should never be written to.

Because of these layout differences, there are situations for arrays and structs where they will occupy different amounts of space depending on being either in storage or memory.

Example

```

uint8[4] arr;
struct Str {
    uint v1;
    uint v2;
    uint8 v3;
    uint8 v4;
}

```
In both cases, the array arr and the struct Str occupy 128 bytes in memory (ie 4 items, 32 bytes each). However, as storage, arr only occupies 32 bytes (1 slot) while Str occupies 96 bytes (3 slots, 32 bytes each).

### Calldata

``Calldata`` is an immutable, temporary location where function arguments are stored, and behaves mostly like memory.

It is recommended to try to use calldata because it avoids unnecessary copies and ensures that the data is unaltered. Arrays and structs with calldata data location can also be returned from functions.

This type of data is assumed to be in a format defined by the ABI specification, ie padded to multiples of 32 bytes (which differs from internal function calls). Arguments for constructors are slightly different, as they are directly appended to the end of the contract’s code (also in ABI encoding).

### Comparision

Whenever you define a reference type variable (array or struct) you will also need to define its data location — unless it’s a state variable, in which case it is automatically interpreted as storage. Since Solidity v0.6.9, memory and calldata are allowed in all functions regardless of their visibility type (ie external, public, etc).

Assignments will either result in copies being created, or mere references to the same piece of data — similar to objects or arrays in Javascript:

- Assignments between storage and memory (or from calldata) always create a separate copy.
- Assignments from memory to memory only create references. Therefore changing one memory variable alters all other memory variables that refer to the same data.
- Assignments from storage to a local storage variable also only assign a reference.
- All other assignments to storage always copy.

For array parameters in functions, it is recommended to use calldata over memory , as this provides significant gas savings. For example, a summing function that loops over an input array can save roughly 1829 gas (~3.5%) by using calldata.

```

// Gas used: 50992
function func1 (uint[] memory nums) external {
  for (uint i = 0; i < nums.length; ++i) {
     ...
  }
}
// Gas used: 49163
function func2 (uint[] calldata nums) external {
  for (uint i = 0; i < nums.length; ++i) {
     ...
  }
}

```

### References

- [Docs](https://docs.soliditylang.org/en/v0.8.13/types.html#data-location-and-assignment-behaviour)
- [Docs](https://docs.soliditylang.org/en/v0.8.13/types.html#reference-types)
- [Docs](https://docs.soliditylang.org/en/v0.8.13/internals/layout_in_storage.html)
- [Twitter](https://twitter.com/PatrickAlphaC/status/1514257121302429696)


<img src="./end.png" alt="The end" style="display:block;margin:auto">
