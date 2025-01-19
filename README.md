# Polygon AggLayer State Sync
## Exploring Sparse Merkle Trees and State Synchronization in Polygon's AggLayer

The Polygon AggLayer cross-chain interopability protocol for token bridging (and message passing) relies on the Unified Bridge (formerly LxLy Bridge) to track deposits and withdrawals across connected L2s and Ethereum. Tracking is implemented with a set of Sparse Merkle Trees (SMTs) that represent the state of each connected chain throughout the connected chains. I described the hierarchical structure of these SMTs in a previous post [*Visualizing Polygon AggLayer Data Structures*](https://medium.com/@j2abro/visualizing-polygon-agglayer-data-structures-9d55c060c9b6). This post describes how this state is transmitted across chains by following the flow of the SMTs from end-to-end in a bridge transaction.

This state synchronization process is a fundamental component of the AggLayer which uses a zero-knowledge (ZK) proof, called the Pessimistic Proof, to provide cryptographic verification of the cross-chain transactions. These SMTs are a a core input to the Pessimistic Proof.

![Local Image](https://github.com/j2abro/MerkleFlow/raw/main/assets/MerkleFlow.svg "Merke Tree Flow")

## Bridge Transaction Example
The following description maps to each step of a bridge deposit shown in the diagram above.

## <img src="./assets/icon1.png" align="top" width="34" height="34"> User Initiates Transaction on L1

User/dapp submits a deposit transaction on Layer 1 Ethereum. Here is an [example transaction on Etherscan](https://etherscan.io/tx/0xf790f5a6ae551dc8e5b04d92941ae79025ba9d485fc1fb7fe3c00b9393332da8).

This will call the `bridgeAsset()` function on [PolygonZkEVMBridgeV2.sol](https://github.com/0xPolygonHermez/zkevm-contracts/blob/4912f4b673015209b3dbe1dd0702a9ffec5c9261/contracts/v2/PolygonZkEVMBridgeV2.sol#L204):

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

Setting [`forceUpdateGlobalExitRoot`](https://github.com/0xPolygonHermez/zkevm-contracts/blob/main/contracts/v2/PolygonZkEVMBridgeV2.sol#L312) to `true` will force the state, Global Exit Root (**GER**) to sync across all networks.

## <img src="./assets/icon2.png" align="top" width="35" height="35"> Lock Tokens
For tokens native to the L1, the tokens bridged across to the L2 will be locked on the L1 bridge contract ([PolygonZkEVMBridgeV2.sol](https://github.com/0xPolygonHermez/zkevm-contracts/blob/4912f4b673015209b3dbe1dd0702a9ffec5c9261/contracts/v2/PolygonZkEVMBridgeV2.sol#L204)). The total aggregate tokens locked on the contract can be [viewed on Etherscan](https://etherscan.io/tokenholdings?a=0x2a3DD3EB832aF982ec71669E178424b10Dca2EDe).

## <img src="./assets/icon3.png" align="top" width="35" height="35"> Send MER to Global Exit Root Manager Contract

The L1 bridge contract will then update the Mainnet Exit Root (**MER**) which tracks all bridging to any connected L2s. This is the equivalent to the Local Exit Root (**LER**) merkle tree root maintained by each of the L2s.  The **MER** is then sent to the global exit root manager contract via [`_updateGlobalExitRoot()`](https://github.com/0xPolygonHermez/zkevm-contracts/blob/4912f4b673015209b3dbe1dd0702a9ffec5c9261/contracts/v2/PolygonZkEVMBridgeV2.sol#L893).


```solidity
    function _updateGlobalExitRoot() internal {
        lastUpdatedDepositCount = uint32(depositCount);
        globalExitRootManager.updateExitRoot(getRoot());
    }
```

## <img src="./assets/icon4.png" align="top" width="35" height="35"> Update Global Exit Root

When the **MER** is updated in the Global Exit Root Manager Contract [(`PolygonZkEVMGlobalExitRootV2.sol`)](https://github.com/0xPolygonHermez/zkevm-contracts/blob/main/contracts/v2/PolygonZkEVMGlobalExitRootV2.sol) as a results of the bridge contract `bridgeAsset()` function the Global Exit Root (**GER**) will be calcualted (a hash of the **RER** and **MER**) and and append it to the L1 Info Tree (a sparse merkle tree).

## <img src="./assets/icon5.png" align="top" width="35" height="35"> L2 Sequencer Fetches GER
The L2 sequencer fetches the L1 GER from the Global Exit Root Manager contract to update the L2 Global Exit Root Manager contract [(`PolygonZkEVMGlobalExitRootL2.sol`)](https://github.com/0xPolygonHermez/zkevm-contracts/blob/main/contracts/PolygonZkEVMGlobalExitRootL2.sol)

## <img src="./assets/icon6.png" align="top" width="35" height="35"> User Claims Tokens on L2
User/dapp submits a claim transaction [PolygonZkEVMBridgeV2.sol](https://github.com/0xPolygonHermez/zkevm-contracts/blob/4912f4b673015209b3dbe1dd0702a9ffec5c9261/contracts/v2/PolygonZkEVMBridgeV2.sol#L446) on Layer 2 (L2) via a zkEVM L2 [RPC endpoint](https://zkevm-rpc.com/).
Here is [the matching claim() transaction on the zkEVM block explorer](https://zkevm.polygonscan.com/tx/0x8519b653373f0aec697d0a428e83d7c243c4f4bae2582e49fb0dfb9338270d0b).


```solidity
function claimAsset(
        bytes32[_DEPOSIT_CONTRACT_TREE_DEPTH] calldata smtProofLocalExitRoot,
        bytes32[_DEPOSIT_CONTRACT_TREE_DEPTH] calldata smtProofRollupExitRoot,
        uint256 globalIndex,
        bytes32 mainnetExitRoot,
        bytes32 rollupExitRoot,
        uint32 originNetwork,
        address originTokenAddress,
        uint32 destinationNetwork,
        address destinationAddress,
        uint256 amount,
        bytes calldata metadata
    )
```

The function takes the MER (`bytes32 mainnetExitRoot`), representative of all the mainneet exits, and the RER (`bytes32 rollupExitRoot`) which encompases all the aggregate exits root hashes of the connected L2 rollups.

Then the `bytes32[_DEPOSIT_CONTRACT_TREE_DEPTH] calldata smtProofLocalExitRoot` is the merkle proof (the array of hashes that are required to prove the validity of the transaction) of the exit tree. This is the tree for which the transaction exited, and noting that this same contract is used on both the L1 and L2, it references the exit tree of the originating network; for this case that is the exit tree of the L1 as this bridge transaction originated on the L1. This verifies that the transaction exit for this claim did indeed originate on the network in question.

the `bytes32[_DEPOSIT_CONTRACT_TREE_DEPTH] calldata smtProofRollupExitRoot` proof ensures that this transaction is also represented on the broader Rollup Exit Tree. 

## <img src="./assets/icon7.png" align="top" width="35" height="35"> Verify Proof and Transfer

The claim asset function then verifies the transaction with the `_verifyLeaf` function in [PolygonZkEVMBridgeV2.sol](https://github.com/0xPolygonHermez/zkevm-contracts/blob/4912f4b673015209b3dbe1dd0702a9ffec5c9261/contracts/v2/PolygonZkEVMBridgeV2.sol#L754), which requires the parameters from above to verify the proofs.

```solidity
function _verifyLeaf(
        bytes32[_DEPOSIT_CONTRACT_TREE_DEPTH] calldata smtProofLocalExitRoot,
        bytes32[_DEPOSIT_CONTRACT_TREE_DEPTH] calldata smtProofRollupExitRoot,
        uint256 globalIndex,
        bytes32 mainnetExitRoot,
        bytes32 rollupExitRoot,
        bytes32 leafValue
    )
```

If the verification succeeds the bridged tokens from the transfer will be credited to the user's account and the transaction is completed. If the verification fails the transaction is reverted.




