# Types

The `TFHE` library provides encrypted integer types and a type system that is checked both at compile time and at run time.

Encrypted integers behave as much as possible as Solidity's integer types. Currently, however, behaviour such as "revert on overflow" is not supported as this would leak some information about the encrypted value. Therefore, arithmetic on `e(u)int` types is [unchecked](https://docs.soliditylang.org/en/latest/control-structures.html#checked-or-unchecked-arithmetic), i.e. there is wrap-around on overflow.

Encrypted integers with overflow checking are coming soon to the `TFHE` library. They will allow reversal in case of an overflow,
but will leak some information about the operands.

In terms of implementation in the `fhEVM`, encrypted integers take the form of FHE ciphertexts.
The `TFHE` library abstracts away that and, instead, exposes ciphertext handles to smart contract developers.
The `e(u)int` types are **wrappers** over these handles.

The following encrypted data types are defined:

| type      | supported       |
| --------- | --------------- |
| `ebool`   | yes (1)         |
| `euint8`  | yes             |
| `euint16` | yes             |
| `euint32` | yes             |
| `euint64` | no, coming soon |
| `eint8`   | no, coming soon |
| `eint16`  | no, coming soon |
| `eint32`  | no, coming soon |
| `eint64`  | no, coming soon |

Higher-precision integers are supported in the `TFHE-rs` library and can be added as needed to `fhEVM`.

> **_NOTE 1:_** The `ebool` type is currently implemented as an `euint8`. A more optimized native boolean type will replace `euint8`.

## Verification

When users send serialized ciphertexts as `bytes` to the blockchain, they first need to be converted to the respective encrypted integer type. Conversion verifies if the ciphertext is well-formed and includes proof verification. These steps prevent usage of arbitrary inputs.
For example, following functions are provided for `ebool`, `euint8`, `euint16` and `euint32`:

- `TFHE.asEbool(bytes ciphertext)` verifies the provided ciphertext and returns an `ebool`
- `TFHE.asEuint8(bytes ciphertext)` verifies the provided ciphertext and returns an `euint8`
- `TFHE.asEuint16(bytes ciphertext)` verifies the provided ciphertext and returns an `euint16`
- `TFHE.asEuint32(bytes ciphertext)` verifies the provided ciphertext and returns an `euint32`
- ... more functions for the respective encrypted integer types

### Example

```solidity
function mint(bytes calldata encryptedAmount) public onlyContractOwner {
  euint32 amount = TFHE.asEuint32(encryptedAmount);
  balances[contractOwner] = balances[contractOwner] + amount;
  totalSupply = totalSupply + amount;
}
```
