## Custom Errors Primer 

```solidity
 error Panic(uint code);
```

Errors in solidity are encoded the same way as functions are encoded.
### How are function encoded?
Let's take an example of this 

```
function Panic(uint code , string memory error );
```
code = 43
error = "Forty three"
- Function selector -` bytes4(keccak256(bytes("Panic(uint,string memory)")))`

- uint is static type 
- string is dynamic type 

Memory layout 

At `0x00`, the first parameter , i.e 43 is placed(0x2b in hex) . 
At `0x20`, the pointer to the location of the string is placed(in our case : 0x40)
At `0x40`, the no. of elements in the string `Forty three` is 11(0xb)
At `0x60`, the string encoded in UTF-8 right padded
```
0x00 : 0x000000000000000000000000000000000000000000000000000000000000002b
0x20 : 0x0000000000000000000000000000000000000000000000000000000000000040
0x40 : 0x000000000000000000000000000000000000000000000000000000000000000b
0x60 : 0x466f727479207468726565000000000000000000000000000000000000000000
```
Final result is :
Note(`0xc0589139` is the function signature computed using above formula)
```
0xc0589139000000000000000000000000000000000000000000000000000000000000002b0000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000b466f727479207468726565000000000000000000000000000000000000000000
```

## Similar to this errors are also encoded in the same way . 

Here' s a full POC on how seaport is returning the errors 

```
function testFailReturnPanicError() public pure {
        // Error Panic(uint256 code)
        assembly{
        mstore(0,0x77d5a156)
        mstore(0x20,43)
        revert(0x1c, 0x24)
        }
    }

```

![testPOCPanicError](https://user-images.githubusercontent.com/91280922/218412590-d95f8842-0f7f-4ae0-b274-a23116e01de2.png)

