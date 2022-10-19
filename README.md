# EVM puzzles

A collection of EVM puzzles. Each puzzle consists on sending a successful transaction to a contract. The bytecode of the contract is provided, and you need to fill the transaction data that won't revert the execution.

## How to play

Clone this repository and install its dependencies (`npm install` or `yarn`). Then run:

```
npx hardhat play
```

And the game will start.

In some puzzles you only need to provide the value that will be sent to the contract, in others the calldata, and in others both values.

You can use [`evm.codes`](https://www.evm.codes/)'s reference and playground to work through this.

# Notes

## Puzzle 1

`CALLVALUE` : Value in the call

`JUMP` : takes **1 input** as **counter** and jump to that **Program Counter**

**answer** : `8`

## Puzzle 2

`CODESIZE` :
Get size of code running in current environment (Number of instructions) -> this code has **10 instructions** thus `10` which is `0xa` in decimal.

`SUB` : takes **2 inputs** from the top of the stack, and pushes the result of subtraction into the stack.

We should jump to 6 where `JUMPDEST` is, so `CODESIZE` - `CALLVALUE` should be `6`.

(Note that `CALLVALUE` is pushed **first** into the **stack** so we subtract `CALLVALUE` from `CODESIZE`, not vice versa)

**answer** : `4`

## Puzzle 3

`CALLDATASIZE` : Returns the size of calldata in **bytes**

We need to jump to 4, thus need any arbitrary 4 bytes hex input

**answer**: `0x01010101`

## Puzzle 4

We need a value of `CALLVALUE` such that `CODESIZE` ^ `CALLVALUE` equals to `0A`.

Reverse operation of ^ (xor) is still ^.

So `CALLVALUE` equals to `CODESIZE` ^ `0x0A`.

The number of instructions here is `12`, which is `0x0C` in hex.

`0x0C` ^ `0x0A` is `0x06`.

**answer** : `6`

## Puzzle 5

`CALLVALUE ^ 2 ` should be equal to `0100` which is `256` in decimal.

**answer** : 16

## Puzzle 6

`CALLDATALOAD` takes an input for **index** and returns `data[i]` which should be `0x0A` here.
So calldata should be `0x000000000000000000000000000000000000000000000000000000000000000A`

**answer** : `0x000000000000000000000000000000000000000000000000000000000000000A`

## Puzzle 7

Instructions `00` to `F0` basically puts the `calldata` value into **memory** and create/deploy a new contract with value stored in memory as a bytecode.

And instructions from `0B` to `0E` checks if the deployed contract's size is `1 byte`.

`CREATE` opcode takes **3 inputs** - `value`, `offset`, `size`.
So the bytecode in area [`offset`, `offset` + `size`) of memory (`calldata` in this case) will be used for `CREATE` operation.

This code should consist of **2 parts**

1. **creation code**
2. **runtime code**

The bytecode that **creation code** returns gets be deployed as a contract.

So in our case, we should write a **creation code** that returns bytecode that has a length of `1 byte`.

**answer** : `0x600060005360016000F3`

```
PUSH1 0x00 // value to be stored (STOP)
PUSH1 0x00 // offset to store
MSTORE
PUSH1 0x01 // size of code to be returned
PUSH1 0x00 // offset of memory
RETURN
```

## Puzzle 8

From instruction `00` to `F0`, the code deploys a contract with **calldata** as a bytecode like in **Puzzle 7**.

And at instructions `0B` ~ `13`, it's executing `CALL` operation which returns `0` if it reverts, and `1` if it succeeded.

We should make the `CALL` revert to pass this.

**answer** : `0x60FD60005360016000F3`

```
PUSH1 0xFD // value to be stored (REVERT)
PUSH1 0x00 // offset to store
MSTORE
PUSH1 0x01 // size of code to be returned
PUSH1 0x00 // offset of memory
RETURN
```

We just bring the code from **Puzzle 7** and make the **runtime code** to **revert** under any circumstances.

## PUzzle 9

From instruction `00` ~ `06`, it checks if the size of `calldata` is bigger than **3 bytes**.

> **condition 1** : calldata size > 3 bytes

And from instruction `0A` to `0F`, the code checks if `CALLVALUE` \* `CALLDATASIZE` equals ot `0x08`.

> **condition 2** : calldata size \* callvalue = 8

**answer** : `0x01010101` (any arbitrary **4 bytes**), `2 wei`

(Or this can by `0x0101010101010101` (any arbitrary **8 bytes**), `1 wei`)

## Puzzle 10

Instruction `00` ~ `06` : checks if `CODESIZE` (`1b`, `27` in dec) is greater than `CALLVALUE`.

> **condition 1** : `CALLVALUE` < `27 wei`

Instruction `09` ~ `0F` : checks if `CALLDATASIZE` is multiple of 3.

> **condition 2** : `CALLDATASIZE` = `3` \* `n`

Instruction `10` ~ `14` : checks if `CALLVALUE + 0x0A` equals to `0x19`

> **condition 3** : `CALLVALUE` = `0x19 - 0x0A` -> `0x0f` (`15` in dec)

**answer** : `0x010101` (any number that is multiple of `3`), `15 wei`
