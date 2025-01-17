# Decrypt and reencrypt

## Decrypt

We allow explicit decryption requests for any encrypted type. The values are decrypted with the network private key (the threshold decryption protocol is in the works).

### Example

```solidity
function getTotalSupply() public view returns (uint32) {
  return TFHE.decrypt(totalSupply);
}

function revertIfConditionIsFalse(ebool condition) public {
  bool plaintextCondition = TFHE.decrypt(condition);
  require(plaintextCondition, "Condition was not met");
  // ... continue execution if `condition` is true
}
```

## Reencrypt

The reencrypt functions takes as inputs a ciphertext and a public encryption key (namely, a [NaCl box](https://nacl.cr.yp.to/index.html)).

During reencryption, the ciphertext is decrypted using the network private key (the threshold decryption protocol is in the works).
Then, the decrypted result is encrypted under the user-provided public encryption key.
The result of this encryption is sent back to the caller as `bytes memory`.

It is also possible to provide a default value to the `reencrypt` function.
In this case, if the provided ciphertext is not initialized (i.e., if the ciphertext handle is `0`), the function will return an encryption of the provided default value.

### Example

```solidity
function balanceOf(bytes32 publicKey) public view returns (bytes memory) {
  return TFHE.reencrypt(balances[msg.sender], publicKey, 0);
}
```

> **_NOTE:_** If one of the following operations is called with an uninitialized ciphertext handle as an operand, this handle will be made to point to a trivial encryption of `0` before the operation is executed.

### Handle private reencryption

In the example above (`balanceOf`), this view function need to validate the user to prevent anyone to reencrypt any user's balance. To prevent this, the user provides a signature of the given public key. The best way to do it is to use [EIP-712 standard](https://eips.ethereum.org/EIPS/eip-712). Since this is something very useful, fhEVM library provide an abstract to use in your contract:

```solidity
import "fhevm/abstracts/EIP712WithModifier.sol";

contract EncryptedERC20 is EIP712WithModifier {
  ...
  constructor() EIP712WithModifier("Authorization token", "1") {
    contractOwner = msg.sender;
  }
  ...
}
```

When a contract uses `EIP712WithModifier` abstract, a modifier is available to check user signature.

```solidity
function balanceOf(
  bytes32 publicKey,
  bytes calldata signature
) public view onlySignedPublicKey(publicKey, signature) returns (bytes memory) {
  return TFHE.reencrypt(balances[msg.sender], publicKey, 0);
}
```

This signature can be generated on client side using [fhevmjs library](../client/reencryption.md).
