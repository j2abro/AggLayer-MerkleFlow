# MerkleFlow
Merkle tree flow through bridge deposit

![Local Image](https://github.com/j2abro/MerkleFlow/raw/main/assets/MerkleFlow.svg "Merke Tree Flow")

# Stepping Through a Bridge Deposit Transaction

## Step 1: Initiate TX**


## <img src="./assets/icon1.png" alt="Custom Icon111" style="width:40px; height:40px; vertical-align:middle;"> Initiate TX
---
## <span style="display: flex; align-items: center;">
  <img src="./assets/icon1.png" alt="Custom Icon222" style="width:40px; height:40px; margin-right: 10px;"> Initiate TX
</span>
---
<span style="display: flex; align-items: center;">
  <img src="./assets/icon1.png" alt="Custom Icon333" style="width:40px; height:40px; margin-right: 10px;"> Initiate TX
</span>
---

```solidity
function bridgeAsset(
        uint32 destinationNetwork,
        address destinationAddress,
        uint256 amount,
        address token,
        bool forceUpdateGlobalExitRoot,
        bytes calldata permitData
    )
```