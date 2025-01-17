# Condition

The result of [comparison operations](functions.md#comparison-operation-eq-ne-ge-gt-le-lt) is of type `ebool`. Typical boolean operations are not currently supported for this type, because it is encrypted.

That said, there are possibilities to condition on `ebool` with or without information leakage.

## Decryption and condition

The first way to have a condition based on an `ebool` is to [decrypt](decrypt.md) it and then either use an if statement or pass it to a `require`. This solution is the simplest way to do a condition but it will leak information. To illustrate this, let's take an example where a user bids for an item in a blind auction.

```solidity
function bid(bytes calldata encryptedBid) internal {
  euint32 bid = TFHE.asEuint32(encryptedBid);
  ebool isAbove = TFHE.le(bid, highestBid);

  // Be sure that the bid is above current highestBid
  require(TFHE.decrypt(isAbove));

  // Replace highest bid
  highestBid = bid;
}
```

In this code, we first evaluate a homomorphic comparison checking that the user has bid more than the highest bid. This homomorphic comparison will return an encryption of 0 if false, or an encryption of 1 if true. Since we are decrypting this value with `TFHE.decrypt`, we are leaking information: if the user didn't bid enough tokens, the transaction is reverted.
For example, a user can know the value of the highest bid by trying every possible values and finally bid just one token above.

## Homomorphic condition

To avoid information leakage, fhEVM provides a method which acts as a ternary operator on encrypted integers. This method is called [cmux](functions.md#multiplexer-operator-cmux).

```solidity
function bid(bytes calldata encryptedBid) internal {
  euint32 bid = TFHE.asEuint32(encryptedBid);
  ebool isAbove = TFHE.le(bid, highestBid);

  // Replace highest bid
  highestBid = TFHE.cmux(isAbove, bid, highestBid);
}
```

It is important to keep in mind that each time we assign a value using `TFHE.cmux`, the value changes, even if the plaintext value remains the same.

## Optimistic encrypted require statements

The decryption statements described above may lead to important delays during the transaction execution as several of them may need to be processed in a single transaction.
Given that those decryptions might be used for control flow by using the Solidity `require` function, we introduce optimistic require statements (`optReq`).
These require statements take as input a value to type `ebool` and are accumulated throughout the execution of the transaction.
The accumulated boolean value is decrypted via the threshold decryption protocol either when an explicit decryption is executed, or at the very end of a transaction execution.
If the decryption returns `false`, the transaction is reverted. Otherwise, state changes are persisted as usual.
Optimistic requires may be more efficient, but this efficiency comes at the price of paying the full transaction gas cost if one of the boolean predicates is false.

```solidity
function transfer(address to, bytes calldata encryptedAmount) internal {
  euint32 amount = TFHE.asEuint32(encryptedAmount);

  ebool hasEnoughTokens = TFHE.le(amount, balances[msg.sender]);

  TFHE.optReq(hasEnoughTokens);

  balances[to] = balances[to] + amount;
  balances[msg.sender] = balances[msg.sender] - amount;
}
```
