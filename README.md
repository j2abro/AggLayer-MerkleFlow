# MerkleFlow
Merkle tree flow through bridge deposit

![Local Image](https://github.com/j2abro/MerkleFlow/raw/main/assets/MerkleFlow.svg "Merke Tree Flow")

# Stepping Through a Bridge Deposit Transaction

## <img src="./assets/icon1.png" align="top" width="34" height="34"> Step 1: Initiate TX

User/dapp submits a deposit transaction on Layer 1 Ethereum (L1) via a zkEVM Layer 2 (L2) [RPC endpoint](https://zkevm-rpc.com/). Here is an [example transaction on Etherscan](https://etherscan.io/tx/0xf790f5a6ae551dc8e5b04d92941ae79025ba9d485fc1fb7fe3c00b9393332da8).

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

## <img src="./assets/icon2.png" align="top" width="35" height="35"> Step 2: Lock Tokens
For tokens native to the L1, the tokens bridged across to the L2 will be locked on the L1 bridge contract ([PolygonZkEVMBridgeV2.sol](https://github.com/0xPolygonHermez/zkevm-contracts/blob/4912f4b673015209b3dbe1dd0702a9ffec5c9261/contracts/v2/PolygonZkEVMBridgeV2.sol#L204)). See Etherscan for [locked bridge token holdings](https://etherscan.io/tokenholdings?a=0x2a3DD3EB832aF982ec71669E178424b10Dca2EDe).

## <img src="./assets/icon3.png" align="top" width="35" height="35"> Step 3: Send MER to Global Exit Root Manager Contract

The L1 bridge contract will then update the it's Mainnet Exit Root (MER) which tracks all bridging to any connected L2s. This is the equivilent to the Local Exit Root (LER) merkle tree root maintained by each of the L2s.  The MER is then sent to the global exit root manager contract via [`_updateGlobalExitRoot()`](https://github.com/0xPolygonHermez/zkevm-contracts/blob/4912f4b673015209b3dbe1dd0702a9ffec5c9261/contracts/v2/PolygonZkEVMBridgeV2.sol#L893).


```solidity
    function _updateGlobalExitRoot() internal {
        lastUpdatedDepositCount = uint32(depositCount);
        globalExitRootManager.updateExitRoot(getRoot());
    }
```

## <img src="./assets/icon4.png" align="top" width="35" height="35"> Step 4: Update Global Exit Root

When the MER is updated in the Global Exit Root Manager Contract [()` PolygonZkEVMGlobalExitRootV2.sol`)](https://etherscan.io/address/0x580bda1e7A0CFAe92Fa7F6c20A3794F169CE3CFb) as a results of the bridge contract `bridgeAsset()` function the Global Exit Root (GER) will be calcualted (a hash of the RER and MER) and and append it to the L1 Info tree (a sparse merkle tree).

## <img src="./assets/icon5.png" align="top" width="35" height="35"> Step 5: ddddd
## <img src="./assets/icon6.png" align="top" width="35" height="35"> Step 6: ddddd
## <img src="./assets/icon7.png" align="top" width="35" height="35"> Step 7: ddddd







