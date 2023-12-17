### TITLE (Root cause + Impact)

### [s-#] Variables stored in storage on-chain are visible to anyone, no matter the solidity visibility keyword meaning the password is not actually a private password.
or
### Storing the password on-chain makes it visible to anyone and no longer private.

**Description:** All data stored on-chain is visible to anyone and can be read directly from the blockchain. The `PaswordStore::s_password` variable is intended to be a private variable and only accessed through the `Password::getPassword` function, which is intended to be only called by the owner of the contract.

we show one such method of reading any data off chain below.

**Impact:** Anyone can read the private password, severly breaking the functionality of the proocol.

**Proof of Concept:** (Proof of Code)

The below test case shows how anyone can read the password directly from the blockchain.

1. create a locally running chain
   ```bash
   make anvil
   ```

2. Deploy the contract to the chain
   ```
   make deploy
   ```

3. Run the storage tool
   We use `1` beacuse that's the storage slot of `s_password` in the contract

   ```
   cast storage <address here> 1 --rpc-url http://127.0.0.1:8545
   ```

   You'll get something like this 

   `0x6d7950617373776f726400000000000000000000000000000000000000000014`

   You can then parse that hex to a string with:

   ```
   cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
   ```

   and get an output of:

   ```
   myPassword
   ```

**Recommended Mitigation:** Due to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain and then store the encrypted password on-chain. This would require the user to remember an additional off-chain password for decryption. However, you'd also likely want to remove the view functions, as you wouldn't want the user to accidentally send a transaction with the password that decrypts your password.