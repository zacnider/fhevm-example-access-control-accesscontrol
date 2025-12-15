# EntropyAccessControl

Access control with EntropyOracle, FHE.allow and FHE.allowTransient

## 🚀 Quick Start

1. **Clone this repository:**
   ```bash
   git clone https://github.com/zacnider/fhevm-example-access-control-accesscontrol.git
   cd fhevm-example-access-control-accesscontrol
   ```

2. **Install dependencies:**
   ```bash
   npm install --legacy-peer-deps
   ```

3. **Setup environment:**
   ```bash
   npm run setup
   ```
   Then edit `.env` file with your credentials:
   - `SEPOLIA_RPC_URL` - Your Sepolia RPC endpoint
   - `PRIVATE_KEY` - Your wallet private key (for deployment)
   - `ETHERSCAN_API_KEY` - Your Etherscan API key (for verification)

4. **Compile contracts:**
   ```bash
   npm run compile
   ```

5. **Run tests:**
   ```bash
   npm test
   ```

6. **Deploy to Sepolia:**
   ```bash
   npm run deploy:sepolia
   ```

7. **Verify contract (after deployment):**
   ```bash
   npm run verify <CONTRACT_ADDRESS>
   ```

**Alternative:** Use the [Examples page](https://entrofhe.vercel.app/examples) for browser-based deployment and verification.

---

## 📋 Overview

@title EntropyAccessControl
@notice Access control with EntropyOracle, FHE.allow and FHE.allowTransient
@dev Example demonstrating EntropyOracle integration: using entropy for access control patterns
This example shows:
- How to integrate with EntropyOracle
- Using entropy to enhance access control patterns
- FHE.allow: Grant specific user decryption rights
- FHE.allowTransient: Grant temporary access for a single operation
- Entropy-based access control

@notice Constructor - sets EntropyOracle address
@param _entropyOracle Address of EntropyOracle contract

@notice Initialize with encrypted value
@param encryptedInput Encrypted value
@param inputProof Input proof

@notice Request entropy for access control operations
@param tag Unique tag for this request
@return requestId Request ID from EntropyOracle
@dev Requires 0.00001 ETH fee

@notice Allow user to decrypt value
@param user Address of user to allow
@dev Uses FHE.allow to grant permanent decryption rights

@notice Perform operation with transient access
@param user Address to grant transient access
@dev Uses FHE.allowTransient for temporary access

@notice Get encrypted value
@return Encrypted value

@notice Allow user with entropy enhancement
@param user Address of user to allow
@param requestId Request ID from requestEntropy()

@notice Check if user is allowed
@param user Address to check
@return True if user is allowed

@notice Get EntropyOracle address



## 🔐 Zama FHEVM Usage

This example demonstrates the following **Zama FHEVM** features:

### Zama FHEVM Features Used

- **ZamaEthereumConfig**: Inherits from Zama's network configuration
  ```solidity
  contract MyContract is ZamaEthereumConfig {
      // Inherits network-specific FHEVM configuration
  }
  ```

- **FHE Operations**: Uses Zama's FHE library for encrypted operations
  - `FHE.add()` - Zama FHEVM operation
  - `FHE.sub()` - Zama FHEVM operation
  - `FHE.mul()` - Zama FHEVM operation
  - `FHE.eq()` - Zama FHEVM operation
  - `FHE.xor()` - Zama FHEVM operation

- **Encrypted Types**: Uses Zama's encrypted integer types
  - `euint64` - 64-bit encrypted unsigned integer
  - `externalEuint64` - External encrypted value from user

- **Access Control**: Uses Zama's permission system
  - `FHE.allowThis()` - Allow contract to use encrypted values
  - `FHE.allow()` - Allow specific user to decrypt
  - `FHE.allowTransient()` - Temporary permission for single operation
  - `FHE.fromExternal()` - Convert external encrypted values to internal

### Zama FHEVM Imports

```solidity
// Zama FHEVM Core Library - FHE operations and encrypted types
import {FHE, euint64, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";

// Zama Network Configuration - Provides network-specific settings
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";
```

### Zama FHEVM Code Example

```solidity
// Using Zama FHEVM's encrypted integer type
euint64 private encryptedValue;

// Converting external encrypted value to internal (Zama FHEVM)
euint64 internalValue = FHE.fromExternal(encryptedValue, inputProof);
FHE.allowThis(internalValue); // Zama FHEVM permission system

// Performing encrypted operations using Zama FHEVM
euint64 result = FHE.add(encryptedValue, FHE.asEuint64(1));
FHE.allowThis(result);
```

### Zama FHEVM Concepts Demonstrated

1. **Encrypted Arithmetic**: Using Zama FHEVM to encrypted arithmetic
2. **Encrypted Comparison**: Using Zama FHEVM to encrypted comparison
3. **External Encryption**: Using Zama FHEVM to external encryption
4. **Permission Management**: Using Zama FHEVM to permission management
5. **Entropy Integration**: Using Zama FHEVM to entropy integration

### Learn More About Zama FHEVM

- 📚 [Zama FHEVM Documentation](https://docs.zama.org/protocol)
- 🎓 [Zama Developer Hub](https://www.zama.org/developer-hub)
- 💻 [Zama FHEVM GitHub](https://github.com/zama-ai/fhevm)


## 🔍 Contract Code

```solidity
// SPDX-License-Identifier: BSD-3-Clause-Clear
pragma solidity ^0.8.27;

import {FHE, euint64, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";
import "./IEntropyOracle.sol";

/**
 * @title EntropyAccessControl
 * @notice Access control with EntropyOracle, FHE.allow and FHE.allowTransient
 * @dev Example demonstrating EntropyOracle integration: using entropy for access control patterns
 * 
 * This example shows:
 * - How to integrate with EntropyOracle
 * - Using entropy to enhance access control patterns
 * - FHE.allow: Grant specific user decryption rights
 * - FHE.allowTransient: Grant temporary access for a single operation
 * - Entropy-based access control
 */
contract EntropyAccessControl is ZamaEthereumConfig {
    // Entropy Oracle interface
    IEntropyOracle public entropyOracle;
    // Encrypted value
    euint64 private encryptedValue;
    
    // Mapping of allowed users
    mapping(address => bool) public allowedUsers;
    
    bool private initialized;
    
    // Track entropy requests
    mapping(uint256 => bool) public entropyRequests;
    
    event ValueStored(address indexed user);
    event UserAllowed(address indexed user);
    event UserRevoked(address indexed user);
    event TransientOperation(address indexed user);
    event EntropyRequested(uint256 indexed requestId, address indexed caller);
    event EntropyAccessGranted(uint256 indexed requestId, address indexed user);
    
    /**
     * @notice Constructor - sets EntropyOracle address
     * @param _entropyOracle Address of EntropyOracle contract
     */
    constructor(address _entropyOracle) {
        require(_entropyOracle != address(0), "Invalid oracle address");
        entropyOracle = IEntropyOracle(_entropyOracle);
    }
    
    /**
     * @notice Initialize with encrypted value
     * @param encryptedInput Encrypted value
     * @param inputProof Input proof
     */
    function initialize(
        externalEuint64 encryptedInput,
        bytes calldata inputProof
    ) external {
        require(!initialized, "Already initialized");
        
        euint64 internalValue = FHE.fromExternal(encryptedInput, inputProof);
        FHE.allowThis(internalValue);
        
        encryptedValue = internalValue;
        initialized = true;
        
        emit ValueStored(msg.sender);
    }
    
    /**
     * @notice Request entropy for access control operations
     * @param tag Unique tag for this request
     * @return requestId Request ID from EntropyOracle
     * @dev Requires 0.00001 ETH fee
     */
    function requestEntropy(bytes32 tag) external payable returns (uint256 requestId) {
        require(initialized, "Not initialized");
        require(msg.value >= entropyOracle.getFee(), "Insufficient fee");
        
        requestId = entropyOracle.requestEntropy{value: msg.value}(tag);
        entropyRequests[requestId] = true;
        
        emit EntropyRequested(requestId, msg.sender);
        return requestId;
    }
    
    /**
     * @notice Allow user to decrypt value
     * @param user Address of user to allow
     * @dev Uses FHE.allow to grant permanent decryption rights
     */
    function allowUser(address user) external {
        require(initialized, "Not initialized");
        require(user != address(0), "Invalid user");
        require(!allowedUsers[user], "User already allowed");
        
        // Allow user to decrypt
        FHE.allow(encryptedValue, user);
        allowedUsers[user] = true;
        
        emit UserAllowed(user);
    }
    
    /**
     * @notice Perform operation with transient access
     * @param user Address to grant transient access
     * @dev Uses FHE.allowTransient for temporary access
     */
    function performTransientOperation(address user) external returns (euint64) {
        require(initialized, "Not initialized");
        require(user != address(0), "Invalid user");
        
        // Grant transient access (only for this operation)
        FHE.allowTransient(encryptedValue, user);
        
        // Perform operation (e.g., add 1)
        euint64 one = FHE.asEuint64(1);
        euint64 result = FHE.add(encryptedValue, one);
        
        emit TransientOperation(user);
        
        return result;
    }
    
    /**
     * @notice Get encrypted value
     * @return Encrypted value
     */
    function getEncryptedValue() external view returns (euint64) {
        require(initialized, "Not initialized");
        return encryptedValue;
    }
    
    /**
     * @notice Allow user with entropy enhancement
     * @param user Address of user to allow
     * @param requestId Request ID from requestEntropy()
     */
    function allowUserWithEntropy(address user, uint256 requestId) external {
        require(initialized, "Not initialized");
        require(user != address(0), "Invalid user");
        require(!allowedUsers[user], "User already allowed");
        require(entropyRequests[requestId], "Invalid request ID");
        require(entropyOracle.isRequestFulfilled(requestId), "Entropy not ready");
        
        // Get entropy
        euint64 entropy = entropyOracle.getEncryptedEntropy(requestId);
        FHE.allowThis(entropy);
        
        // Mix value with entropy
        euint64 enhancedValue = FHE.xor(encryptedValue, entropy);
        FHE.allowThis(enhancedValue);
        
        // Allow user to decrypt enhanced value
        FHE.allow(enhancedValue, user);
        allowedUsers[user] = true;
        
        entropyRequests[requestId] = false;
        emit EntropyAccessGranted(requestId, user);
        emit UserAllowed(user);
    }
    
    /**
     * @notice Check if user is allowed
     * @param user Address to check
     * @return True if user is allowed
     */
    function isUserAllowed(address user) external view returns (bool) {
        return allowedUsers[user];
    }
    
    /**
     * @notice Get EntropyOracle address
     */
    function getEntropyOracle() external view returns (address) {
        return address(entropyOracle);
    }
}

```

## 🧪 Tests

See [test file](./test/EntropyAccessControl.test.ts) for comprehensive test coverage.

```bash
npm test
```


## 📚 Category

**access**



## 🔗 Related Examples

- [All access examples](https://github.com/zacnider/entrofhe/tree/main/examples)

## 📝 License

BSD-3-Clause-Clear
