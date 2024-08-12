<details>
### [H-1] On-Chain Visibility of "Private" Variables: Exposing Sensitive Data and Compromising Protocol Security

## Likelihood & Impact:

-Impact: HIGH
-Likelihood: HIGH
-Severity: HIGH

Description:
All data stored on-chain is accessible to anyone, regardless of the Solidity visibility keyword used. In this case, the `PasswordStore::s_password` variable is intended to be private and only accessed through the `PasswordStore::getPassword` function, which is designed to be callable only by the contract owner. However, because blockchain data is publicly accessible, the password is not truly private.

Proof of Concept:
The following steps demonstrate how anyone can retrieve the password directly from the blockchain:

Set up a Local Blockchain:

```
  bash
```

make anvil

Deploy the Contract:

```
bash
```

make deploy

Read the Stored Password:
Use the storage tool to fetch the password:

```
bash
```

cast storage <Contract_Address> 1 --rpc-url http://127.0.0.1:8545
The output will resemble:

0x6d7950617373776f726400000000000000000000000000000000000000000014
Convert the Hexadecimal Value to a String:

```
bash
```

cast --parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
This returns:
mypassword

Recommended Mitigation:
To address this vulnerability, the contract's architecture should be reconsidered. One possible solution is to encrypt the password off-chain before storing it on-chain. This approach would require users to maintain an additional off-chain password for decryption. Additionally, it would be wise to remove the view function to prevent users from inadvertently exposing the password when making a transaction.

</details>

<details>
### [H-2] Lack of Access Control on setPassword Function (Missing Ownership Check + Unauthorized Access)

## Likelihood & Impact:

-Impact: CRITICAL
-Likelihood: HIGH
-Severity: CRITICAL

Description:
The `setPassword` function in the `PasswordStore` contract lacks proper access control. Currently, the function is external, which means anyone can call this function and set the password to a new value without any restrictions.

Impact:
This issue allows any external party to change the stored password without permission, which could lead to unauthorized access or loss of integrity of the stored data. The lack of access control undermines the contract's security by exposing critical functionality to unauthorized users.

Proof of Concept:

```Javascript
// Current vulnerable function in the contract:
function setPassword(string memory newPassword) external {
    s_password = newPassword;
    emit SetN``etPassword();
}

// Any external address can call this function:
address attacker = 0xAttackerAddress;
attacker.call(abi.encodeWithSignature("setPassword(string)", "newMaliciousPassword"));
```

Recommended Mitigation:
Implement an access control mechanism by adding a modifier to check if msg.sender is the owner of the contract before allowing the password to be set. This ensures that only the owner can modify the password.

```Javascript
// Access control modifier to restrict function to owner only
modifier onlyOwner() {
    if (msg.sender != s_owner) {
        revert PasswordStore__NotOwner();
    }
    _;
}

// Updated setPassword function with access control
function setPassword(string memory newPassword) external onlyOwner {
s_password = newPassword;
emit SetNetPassword();
}

```

</details>

<details>
### [L-1] Event Naming Convention (Inconsistent Naming + Lack of Descriptiveness)

## Likelihood & Impact:

-Impact: LOW
-Likelihood: MEDIUM
-Severity: LOW

Description:
The event `SetNetPassword` in the `PasswordStore` contract is not descriptive and does not follow Solidity's common naming conventions. This can make it harder for developers to understand the purpose of the event at a glance.

Impact:
Using non-descriptive event names can lead to confusion and reduce code readability. It can also make it difficult for external tools and developers to accurately interpret the event's purpose.

Proof of Concept:

```javascript
// Current event declaration:
event SetNetPassword();
```

Recommended Mitigation:
Rename the event to something more descriptive, such as PasswordUpdated or PasswordChanged, to better reflect the action being recorded.

```javascript
// Improved event declaration:
event PasswordUpdated();
```

</details>

<details>
### [ i -1] Error Naming Convention (Unnecessary Complexity + Increased Gas Costs)

## Likelihood & Impact:

-Impact: LOW
-Likelihood: HIGH
-Severity: LOW

Description:
The error `PasswordStore__NotOwner` in the `PasswordStore` contract includes double underscores, which are unnecessary and could lead to slightly increased gas costs due to the longer identifier.

Impact:
Longer error names consume more gas and make the code less readable. Simplifying the error name can save gas and improve code clarity.

Proof of Concept:

```javascript
// Current error declaration:
error PasswordStore__NotOwner();
```

Recommended Mitigation:
Simplify the error name to something like NotOwner or Unauthorized to reduce gas costs and improve readability.

```javascript
// Improved error declaration:
error NotOwner();
```

</details>

<details>
### [M-1] Compiler Version (Locked Version + Potential Compatibility Issues)

## Likelihood & Impact:

-Impact: MEDIUM
-Likelihood: MEDIUM
-Severity: MEDIUM

Description:
The contract uses pragma solidity 0.8.18; which locks the contract to a specific compiler version. This approach could lead to issues if bug fixes or optimizations are introduced in later versions of the compiler.

Impact:
Locking the compiler version can prevent the contract from benefiting from future compiler improvements and may also introduce compatibility issues if other contracts or libraries use different versions.

Proof of Concept:

```javascript
// Current pragma directive:
pragma solidity 0.8.18;
```

Recommended Mitigation:
Consider using a version range like ^0.8.18 to allow for patch-level updates while maintaining compatibility.

```javascript
// Improved pragma directive:
pragma solidity ^0.8.18;
```

</details>
