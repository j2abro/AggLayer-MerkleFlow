# MerkleFlow
Merkle tree flow through bridge deposit

![Local Image](https://github.com/j2abro/MerkleFlow/raw/main/assets/MerkleFlow.svg "Merke Tree Flow")

# Stepping Through a Bridge Deposit Transaction

## <p><img src="./assets/icon1.png" align="top" width="34" height="34"> Step 1: Initiate TX</p>

User/dapp submits a deposit transaction on Layer 1 Ethereum (L1) via a zkEVM Layer 2 (L2) [(RPC endpoint)](https://zkevm-rpc.com/). Here is an [example transaction on Etherscan](https://etherscan.io/tx/0xf790f5a6ae551dc8e5b04d92941ae79025ba9d485fc1fb7fe3c00b9393332da8).

This will call the `bridgeAsset()` function on [PolygonZkEVMBridgeV2.sol](https://github.com/0xPolygonHermez/zkevm-contracts/blob/4912f4b673015209b3dbe1dd0702a9ffec5c9261/contracts/v2/PolygonZkEVMBridgeV2.sol#L204).


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

Setting [`forceUpdateGlobalExitRoot`](https://github.com/0xPolygonHermez/zkevm-contracts/blob/main/contracts/v2/PolygonZkEVMBridgeV2.sol#L312) to `true' will force the state (global exit root) to sync across all networks.

## <p><img src="./assets/icon2.png" align="top" width="35" height="35"> Step 2: Lock Tokens</p>
For tokens native to the L1, the tokens bridged across to the L

## <p><img src="./assets/icon3.png" align="top" width="35" height="35"> Step 3: ddddd</p>
## <p><img src="./assets/icon4.png" align="top" width="35" height="35"> Step 4: ddddd</p>
## <p><img src="./assets/icon5.png" align="top" width="35" height="35"> Step 5: ddddd</p>
## <p><img src="./assets/icon6.png" align="top" width="35" height="35"> Step 6: ddddd</p>
## <p><img src="./assets/icon7.png" align="top" width="35" height="35"> Step 7: ddddd</p>







