# Conduit 

`
Conduit allow registered callers to transfer ERC20/721/1155 tokens on the owner's behalf.
`

![Conduit](https://user-images.githubusercontent.com/91280922/215829233-80a7986b-5701-4e48-b014-8d90dba0132c.svg)

## Functions

### onlyOpenChannel()

```solidity
 /**
     * @notice Ensure that the caller is currently registered as an open channel
     *         on the conduit.
     */
    modifier onlyOpenChannel() {
        // Utilize assembly to access channel storage mapping directly.
        assembly {
            // Write the caller to scratch space.
            mstore(ChannelKey_channel_ptr, caller())

            // Write the storage slot for _channels to scratch space.
            mstore(ChannelKey_slot_ptr, _channels.slot)

            // Derive the position in storage of _channels[msg.sender]
            // and check if the stored value is zero.
            if iszero(
                sload(keccak256(ChannelKey_channel_ptr, ChannelKey_length))
            ) {
                // The caller is not an open channel; revert with
                // ChannelClosed(caller). First, set error signature in memory.
                mstore(ChannelClosed_error_ptr, ChannelClosed_error_signature)

                // Next, set the caller as the argument.
                mstore(ChannelClosed_channel_ptr, caller())

                // Finally, revert, returning full custom error with argument.
                revert(ChannelClosed_error_ptr, ChannelClosed_error_length)
            }
        }

        // Continue with function execution.
        _;
    }
```

Suppose caller is `Bob` . 
To ascertain whether Bob is allowed to call ,we need to check if `_channels[msg.sender]` == `Bob` .

First step : Store the caller in memory at an appropriate offset. 

```
            mstore(ChannelKey_channel_ptr, caller())
```
Here , we are storing the 20 bytes address at offset `ChannelKey_channel_ptr`(0x00Defined in ConduitConstants). 

Say address(Bob) = 0x1b37B1EC6B7faaCbB9AddCCA4043824F36Fb88D8

- Memory Layout :
```
0x00 0x0000000000000000000000001b37b1ec6b7faacbb9addcca4043824f36fb88d8
```
To access the ``_channels[msg.sender]` slot key, the formula is 
`keccak256(account,_channels.slot)`

** Note :=
- _channels.slot will return the general slot number, not the value.

Hence we store the slot number of _channels in the next memory layout.

```solidity
mstore(ChannelKey_slot_ptr, _channels.slot)
```
ChannelKey_slot_ptr = 0x20(as defined by the contract)
_channels.slot = 0 (As it is the first word of the storage layout , Tested via `forge inspect <path> storage` command)

- Memory Layout :-

```
0x00 0x0000000000000000000000001b37b1ec6b7faacbb9addcca4043824f36fb88d8
0x20 0x0000000000000000000000000000000000000000000000000000000000000000
```
Now using these 2 values , we calculate the slot of `_channels[Bob]` 

```solidity
sload(keccak256(ChannelKey_channel_ptr, ChannelKey_length))
```

We use the keccak256 function for this . 
The first input is the offset of the data .
Second input is the no of bytes of the data to capture .
We know that ,
ChannelKey_channel_ptr= 0x00
ChannelKey_length = 0x40

All this was done to get this `keccak256(account,_channels.slot)`
This opcode will return Hash at the top of the stack .Now we use `SLOAD` to get the value using the key a.k.a hash

If `Bob` is an open channel (=1 ), we continue the function execution . But if Bob is not an open channel(=0) , i.e we receive false :-

We throw a custom error message `ChannelClosed(caller)`

For more details on how to throw a custom error message , [here](./Errors.md)





