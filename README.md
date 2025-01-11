# MerkleFlow
Merkle tree flow through bridge deposit

![Local Image](https://github.com/j2abro/MerkleFlow/raw/main/assets/MerkleFlow.svg "Merke Tree Flow")

# Stepping Through a Bridge Deposit Transaction

## Step 1: Initiate TX**

 <img src="icon1.png" alt="Custom Icon" style="width:20px; height:20px; vertical-align:middle;"> Initiate TX

and

## <img src="icon1.png" alt="Custom Icon" style="width:20px; height:20px; vertical-align:middle;"> Initiate TX
again

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