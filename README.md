# EntropyAccessControl

Access control with EntropyOracle, FHE.allow and FHE.allowTransient

## 🚀 Standard workflow
- Install (first run): `npm install --legacy-peer-deps`
- Compile: `npx hardhat compile`
- Test (local FHE + local oracle/chaos engine auto-deployed): `npx hardhat test`
- Deploy (frontend Deploy button): constructor arg is fixed to EntropyOracle `0x75b923d7940E1BD6689EbFdbBDCD74C1f6695361`
- Verify: `npx hardhat verify --network sepolia <contractAddress> 0x75b923d7940E1BD6689EbFdbBDCD74C1f6695361`

## 📋 Overview

This example demonstrates **access-control** concepts in FHEVM with **EntropyOracle integration**:
- Integrating with EntropyOracle
- Using entropy to enhance access control patterns
- FHE.allow: Grant specific user decryption rights
- FHE.allowTransient: Grant temporary access for a single operation
- Entropy-based access control

## 💡 Key Concepts

### EntropyOracle Integration
The contract uses EntropyOracle to get encrypted randomness for enhanced access control:
```solidity
IEntropyOracle entropyOracle;
uint256 requestId = entropyOracle.requestEntropy{value: fee}(tag);
euint64 entropy = entropyOracle.getEncryptedEntropy(requestId);
```

### Entropy-Enhanced Access Control
Instead of simple access control, permissions can be enhanced with entropy:
- Request entropy from EntropyOracle
- Mix encrypted value with entropy
- Grant user access to entropy-enhanced value
- Result: Entropy-enhanced access control

### Basic Access Control
Standard access control patterns are still available:
- `allowUser()` - Grant permanent decryption rights
- `performTransientOperation()` - Grant temporary access

### FHE.allow vs FHE.allowTransient
- `FHE.allow()`: Permanent access (until revoked)
- `FHE.allowTransient()`: Temporary access (only for one operation)

## 🚀 Quick Start

### Prerequisites

- Node.js 18+
- Hardhat
- Sepolia Testnet (for FHEVM)
- **Deployed EntropyOracle contract** (required for entropy-based access control)

### Installation

```bash
npm install
```

### Compile

```bash
npm run compile
```

### Test

```bash
npm test
```

**Note**: Full entropy tests require a deployed EntropyOracle contract. Set `ENTROPY_ORACLE_ADDRESS` environment variable.

## 📖 Usage Example

### Basic Usage (Without Entropy)

```typescript
// Deploy contract
const contract = await EntropyAccessControlFactory.deploy(oracleAddress);

// Initialize
const input = hre.fhevm.createEncryptedInput(contractAddress, userAddress);
input.add64(42);
const encryptedInput = await input.encrypt();

await contract.initialize(encryptedInput.handles[0], encryptedInput.inputProof);

// Allow user
await contract.allowUser(userAddress);

// Transient operation
await contract.performTransientOperation(userAddress);
```

### Entropy-Enhanced Usage

```typescript
// Request entropy
const tag = ethers.id("access-control-1");
const fee = await contract.entropyOracle.getFee();
const requestId = await contract.requestEntropy(tag, { value: fee });

// Wait for entropy to be ready
await waitForEntropy(requestId);

// Allow user with entropy
await contract.allowUserWithEntropy(userAddress, requestId);
```

## 🔗 Related Examples

- [EntropyUserDecryption](../user-decryption-userdecryptsingle/) - Entropy-based user decryption
- [EntropyPublicDecryption](../public-decryption-publicdecryptsingle/) - Entropy-based public decryption
- [Category: access-control](../)

## 📝 License

BSD-3-Clause-Clear
