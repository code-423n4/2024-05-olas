# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings) | 42 |
| [GAS-2](#GAS-2) | Using bools for storage incurs overhead | 7 |
| [GAS-3](#GAS-3) | Cache array length outside of loop | 22 |
| [GAS-4](#GAS-4) | For Operations that will not overflow, you could use unchecked | 294 |
| [GAS-5](#GAS-5) | Avoid contract existence checks by using low level calls | 9 |
| [GAS-6](#GAS-6) | `++i` costs less gas compared to `i++` or `i += 1` (same for `--i` vs `i--` or `i -= 1`) | 7 |
| [GAS-7](#GAS-7) | Using `private` rather than `public` for constants, saves gas | 31 |
| [GAS-8](#GAS-8) | Use shift right/left instead of division/multiplication if possible | 4 |
| [GAS-9](#GAS-9) | `uint256` to `bool` `mapping`: Utilizing Bitmaps to dramatically save on Gas | 1 |
| [GAS-10](#GAS-10) | Increments/decrements can be unchecked in for-loops | 44 |
| [GAS-11](#GAS-11) | Use != 0 instead of > 0 for unsigned integer comparison | 58 |
| [GAS-12](#GAS-12) | `internal` functions not called by the contract should be removed | 2 |
### <a name="GAS-1"></a>[GAS-1] `a = a + b` is more gas effective than `a += b` for state variables (excluding arrays and mappings)
This saves **16 gas per instance.**

*Instances (42)*:
```solidity
File: governance/contracts/VoteWeighting.sol

231:             t += WEEK;

271:             t += WEEK;

539:             pointsWeight[nomineeHash][nextTime].slope += newSlope.slope;

540:             pointsSum[nextTime].slope += newSlope.slope;

548:         changesWeight[nomineeHash][newSlope.end] += newSlope.slope;

549:         changesSum[newSlope.end] += newSlope.slope;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

575:                     totalRewards += eligibleServiceRewards[numServices];

624:                     updatedTotalRewards += updatedReward;

629:                     mapServiceInfo[curServiceId].reward += updatedReward;

634:                 updatedTotalRewards += updatedReward;

639:                     updatedReward += lastAvailableRewards - updatedTotalRewards;

643:                 mapServiceInfo[curServiceId].reward += updatedReward;

653:                     mapServiceInfo[curServiceId].reward += eligibleServiceRewards[i];

922:         reward += calculateStakingLastReward(serviceId);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

540:             totalValueAmount += valueAmounts[i];

609:                 transferAmounts[i] += stakingIncentive;

610:                 totalAmounts[0] += stakingIncentive;

611:                 totalAmounts[2] += returnAmount;

630:             totalAmounts[1] += transferAmounts[i];

924:                     returnAmount += stakingIncentive - availableStakingAmount;

936:                     returnAmount += stakingIncentive - normalizedStakingAmount;

941:                 totalStakingIncentive += stakingIncentive;

944:             totalReturnAmount += returnAmount;

1157:             totalReturnAmount += stakingPoint.stakingIncentive * stakingWeight;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

947:                             mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

953:                             mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;

954:                             mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;

1019:         donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;

1138:             inflationPerEpoch += (block.timestamp - yearEndTime) * curInflationPerSecond;

1254:             inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;

1270:         curMaxBond += effectiveBond;

1370:             reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1373:             topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

1440:                     reward += totalIncentives / 100;

1451:                     topUp += totalIncentives / sumUnitIncentives;

1456:             reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1458:             topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

59:                 supplyCap += (supplyCap * maxMintCapFraction) / 100;

96:                 supplyCap += (supplyCap * maxMintCapFraction) / 100;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/TokenomicsConstants.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

172:                 localWithheldAmount += amount;

182:                 localWithheldAmount += targetWithheldAmount;

209:             withheldAmount += localWithheldAmount;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

### <a name="GAS-2"></a>[GAS-2] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (7)*:
```solidity
File: registries/contracts/staking/StakingVerifier.sol

64:     bool public implementationsCheck;

67:     mapping(address => bool) public mapImplementations;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

367:     mapping(uint256 => mapping(uint256 => bool)) public mapNewUnits;

369:     mapping(address => bool) public mapNewOwners;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

93:     mapping(bytes32 => bool) public stakingQueueingNonces;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/WormholeDepositProcessorL1.sol

18:     mapping(bytes32 => bool) public mapDeliveryHashes;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/WormholeTargetDispenserL2.sol

54:     mapping(bytes32 => bool) public mapDeliveryHashes;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeTargetDispenserL2.sol)

### <a name="GAS-3"></a>[GAS-3] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (22)*:
```solidity
File: governance/contracts/VoteWeighting.sol

573:         for (uint256 i = 0; i < accounts.length; ++i) {

793:         for (uint256 i = 0; i < accounts.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

332:         for (uint256 i = 0; i < _stakingParams.agentIds.length; ++i) {

669:             for (uint256 i = 0; i < serviceIds.length; ++i) {

835:         for (; idx < serviceIds.length; ++idx) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingToken.sol

92:         for (uint256 i = 0; i < serviceAgentIds.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingToken.sol)

```solidity
File: registries/contracts/staking/StakingVerifier.sol

150:         for (uint256 i = 0; i < implementations.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

450:         for (uint256 i = 0; i < chainIds.length; ++i) {

462:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

473:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

485:                 for (uint256 j = 0; j < updatedStakingTargets.length; ++j) {

526:         for (uint256 i = 0; i < chainIds.length; ++i) {

550:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

593:         for (uint256 i = 0; i < chainIds.length; ++i) {

600:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

717:         for (uint256 i = 0; i < chainIds.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

1335:         for (uint256 i = 0; i < unitIds.length; ++i) {

1357:         for (uint256 i = 0; i < unitIds.length; ++i) {

1407:         for (uint256 i = 0; i < unitIds.length; ++i) {

1429:         for (uint256 i = 0; i < unitIds.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

154:         for (uint256 i = 0; i < targets.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

94:         for (uint256 i = 0; i < targets.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)

### <a name="GAS-4"></a>[GAS-4] For Operations that will not overflow, you could use unchecked

*Instances (294)*:
```solidity
File: governance/contracts/VoteWeighting.sol

159:     uint256 public constant MAX_EVM_CHAIN_ID = type(uint64).max / 2 - 36;

214:         timeSum = block.timestamp / WEEK * WEEK;

227:         for (uint256 i = 0; i < MAX_NUM_WEEKS; i++) {

231:             t += WEEK;

232:             uint256 dBias = pt.slope * WEEK;

234:                 pt.bias -= dBias;

236:                 pt.slope -= dSlope;

267:         for (uint256 i = 0; i < MAX_NUM_WEEKS; i++) {

271:             t += WEEK;

272:             uint256 dBias = pt.slope * WEEK;

274:                 pt.bias -= dBias;

276:                 pt.slope -= dSlope;

309:         uint256 nextTime = (block.timestamp + WEEK) / WEEK * WEEK;

357:             revert Underflow(chainId, MAX_EVM_CHAIN_ID + 1);

424:         uint256 t = time / WEEK * WEEK;

432:             weight = 1e18 * nomineeWeight / totalSum;

489:         uint256 nextTime = (block.timestamp + WEEK) / WEEK * WEEK;

502:         uint256 nextAllowedVotingTime = lastUserVote[msg.sender][nomineeHash] + WEIGHT_VOTE_DELAY;

511:             oldBias = oldSlope.slope * (oldSlope.end - nextTime);

515:             slope: uint256(uint128(userSlope)) * weight / MAX_WEIGHT,

520:         uint256 newBias = newSlope.slope * (lockEnd - nextTime);

523:         powerUsed = powerUsed + newSlope.power - oldSlope.power;

532:         pointsWeight[nomineeHash][nextTime].bias = _maxAndSub(_getWeight(account, chainId) + newBias, oldBias);

533:         pointsSum[nextTime].bias = _maxAndSub(_getSum() + newBias, oldBias);

536:                 _maxAndSub(pointsWeight[nomineeHash][nextTime].slope + newSlope.slope, oldSlope.slope);

537:             pointsSum[nextTime].slope = _maxAndSub(pointsSum[nextTime].slope + newSlope.slope, oldSlope.slope);

539:             pointsWeight[nomineeHash][nextTime].slope += newSlope.slope;

540:             pointsSum[nextTime].slope += newSlope.slope;

544:             changesWeight[nomineeHash][oldSlope.end] -= oldSlope.slope;

545:             changesSum[oldSlope.end] -= oldSlope.slope;

548:         changesWeight[nomineeHash][newSlope.end] += newSlope.slope;

549:         changesSum[newSlope.end] += newSlope.slope;

573:         for (uint256 i = 0; i < accounts.length; ++i) {

579:         return a > b ? a - b : 0;

605:         uint256 nextTime = (block.timestamp + WEEK) / WEEK * WEEK;

610:         uint256 newSum = oldSum - oldWeight;

622:         nominee = setNominees[setNominees.length - 1];

659:             changesWeight[nomineeHash][oldSlope.end] -= oldSlope.slope;

660:             changesSum[oldSlope.end] -= oldSlope.slope;

665:         powerUsed = powerUsed - oldSlope.power;

692:         return setNominees.length - 1;

699:         return setRemovedNominees.length - 1;

746:         uint256 totalNumNominees = setNominees.length - 1;

763:         uint256 totalNumRemovedNominees = setRemovedNominees.length - 1;

793:         for (uint256 i = 0; i < accounts.length; ++i) {

804:             nextAllowedVotingTimes[i] = lastUserVote[voters[i]][nomineeHash] + WEIGHT_VOTE_DELAY;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingActivityChecker.sol

58:             uint256 ratio = ((curNonces[0] - lastNonces[0]) * 1e18) / ts;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingActivityChecker.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

4: import {ERC721TokenReceiver} from "../../lib/solmate/src/tokens/ERC721.sol";

332:         for (uint256 i = 0; i < _stakingParams.agentIds.length; ++i) {

350:         minStakingDuration = _stakingParams.minNumStakingPeriods * livenessPeriod;

353:         maxInactivityDuration = _stakingParams.maxNumInactivityPeriods * livenessPeriod;

356:         emissionsAmount = _stakingParams.rewardsPerSecond * _stakingParams.maxNumServices *

380:         for (uint256 i = 0; i < numAgentIds; ++i) {

449:         for (uint256 i = 0; i < totalNumServices; ++i) {

459:                 sCounter++;

464:         for (uint256 i = numEvictServices; i > 0; --i) {

466:             totalNumServices--;

468:             uint256 idx = serviceIndexes[i - 1];

536:         if (block.timestamp - tsCheckpointLast >= livenessPeriod && lastAvailableRewards > 0) {

548:             for (uint256 i = 0; i < size; ++i) {

566:                 ts = block.timestamp - serviceCheckpoint;

574:                     eligibleServiceRewards[numServices] = rewardsPerSecond * ts;

575:                     totalRewards += eligibleServiceRewards[numServices];

577:                     ++numServices;

620:                 for (uint256 i = 1; i < numServices; ++i) {

622:                     updatedReward = (eligibleServiceRewards[i] * lastAvailableRewards) / totalRewards;

624:                     updatedTotalRewards += updatedReward;

629:                     mapServiceInfo[curServiceId].reward += updatedReward;

633:                 updatedReward = (eligibleServiceRewards[0] * lastAvailableRewards) / totalRewards;

634:                 updatedTotalRewards += updatedReward;

639:                     updatedReward += lastAvailableRewards - updatedTotalRewards;

643:                 mapServiceInfo[curServiceId].reward += updatedReward;

648:                 for (uint256 i = 0; i < numServices; ++i) {

653:                     mapServiceInfo[curServiceId].reward += eligibleServiceRewards[i];

657:                 lastAvailableRewards -= totalRewards;

669:             for (uint256 i = 0; i < serviceIds.length; ++i) {

678:                     serviceInactivity[i] = mapServiceInfo[curServiceId].inactivity + serviceInactivity[i];

685:                         numServices++;

701:             uint256 epochLength = block.timestamp - tsCheckpoint;

706:             epochCounter = eCounter + 1;

773:             for (uint256 i = 0; i < numAgents; ++i) {

817:         uint256 ts = block.timestamp - tsStart;

835:         for (; idx < serviceIds.length; ++idx) {

858:             setServiceIds[idx] = setServiceIds[setServiceIds.length - 1];

899:         for (uint256 i = 0; i < numServices; ++i) {

904:                     reward = (eligibleServiceRewards[i] * lastAvailableRewards) / totalRewards;

922:         reward += calculateStakingLastReward(serviceId);

941:         tsNext = tsCheckpoint + livenessPeriod;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingFactory.sol

4: import {StakingProxy} from "./StakingProxy.sol";

239:         nonce = localNonce + 1;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: registries/contracts/staking/StakingNativeToken.sol

4: import {StakingBase} from "./StakingBase.sol";

30:         balance -= amount;

41:         uint256 newBalance = balance + msg.value;

42:         uint256 newAvailableRewards = availableRewards + msg.value;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingNativeToken.sol)

```solidity
File: registries/contracts/staking/StakingToken.sol

4: import {StakingBase} from "./StakingBase.sol";

5: import {SafeTransferLib} from "../utils/SafeTransferLib.sol";

92:         for (uint256 i = 0; i < serviceAgentIds.length; ++i) {

106:         balance -= amount;

117:         uint256 newBalance = balance + amount;

118:         uint256 newAvailableRewards = availableRewards + amount;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingToken.sol)

```solidity
File: registries/contracts/staking/StakingVerifier.sol

91:         emissionsLimit = _rewardsPerSecondLimit * _timeForEmissionsLimit * _numServicesLimit;

150:         for (uint256 i = 0; i < implementations.length; ++i) {

247:         emissionsLimit = _rewardsPerSecondLimit * _timeForEmissionsLimit * _numServicesLimit;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: registries/contracts/utils/SafeTransferLib.sol

32:             mstore(4, from) // Append the "from" argument.

33:             mstore(36, to) // Append the "to" argument.

34:             mstore(68, amount) // Append the "amount" argument.

46:             mstore(0x60, 0) // Restore the zero slot to zero.

47:             mstore(0x40, memPointer) // Restore the memPointer.

74:             mstore(4, to) // Append the "to" argument.

75:             mstore(36, amount) // Append the "amount" argument.

87:             mstore(0x60, 0) // Restore the zero slot to zero.

88:             mstore(0x40, memPointer) // Restore the memPointer.

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/utils/SafeTransferLib.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

275:     uint256 public constant MAX_EVM_CHAIN_ID = type(uint64).max / 2 - 36;

376:             revert Overflow(firstClaimedEpoch, eCounter - 1);

380:         uint256 epochAfterRemoved = mapRemovedNomineeEpochs[nomineeHash] + 1;

384:             revert Overflow(firstClaimedEpoch, epochAfterRemoved - 1);

388:         lastClaimedEpoch = firstClaimedEpoch + numClaimedEpochs;

450:         for (uint256 i = 0; i < chainIds.length; ++i) {

462:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

465:                     ++numActualTargets;

473:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

477:                     ++numPos;

485:                 for (uint256 j = 0; j < updatedStakingTargets.length; ++j) {

526:         for (uint256 i = 0; i < chainIds.length; ++i) {

540:             totalValueAmount += valueAmounts[i];

550:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

593:         for (uint256 i = 0; i < chainIds.length; ++i) {

600:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

609:                 transferAmounts[i] += stakingIncentive;

610:                 totalAmounts[0] += stakingIncentive;

611:                 totalAmounts[2] += returnAmount;

619:                         withheldAmount -= transferAmounts[i];

622:                         transferAmounts[i] -= withheldAmount;

630:             totalAmounts[1] += transferAmounts[i];

717:         for (uint256 i = 0; i < chainIds.length; ++i) {

766:         uint256 endTime = ITokenomics(tokenomics).getEpochEndTime(eCounter - 1);

773:         uint256 maxAllowedTime = endTime + epochLen - 1 weeks;

811:         if ((reward + topUp) > 0) {

822:                 olasBalance = IToken(olas).balanceOf(msg.sender) - olasBalance;

881:         for (uint256 j = firstClaimedEpoch; j < lastClaimedEpoch; ++j) {

905:                 stakingDiff = availableStakingAmount - totalWeightSum;

911:             if (stakingWeight < uint256(stakingPoint.minStakingWeight) * 1e14) {

913:                 returnAmount = ((stakingDiff + availableStakingAmount) * stakingWeight) / 1e18;

916:                 stakingIncentive = (availableStakingAmount * stakingWeight) / 1e18;

918:                 returnAmount = (stakingDiff * stakingWeight) / 1e18;

924:                     returnAmount += stakingIncentive - availableStakingAmount;

932:                     uint256 normalizedStakingAmount = stakingIncentive / (10 ** (18 - bridgingDecimals));

933:                     normalizedStakingAmount *= 10 ** (18 - bridgingDecimals);

936:                     returnAmount += stakingIncentive - normalizedStakingAmount;

941:                 totalStakingIncentive += stakingIncentive;

944:             totalReturnAmount += returnAmount;

1016:                     withheldAmount -= transferAmount;

1020:                     transferAmount -= withheldAmount;

1034:                 olasBalance = IToken(olas).balanceOf(address(this)) - olasBalance;

1112:                 olasBalance = IToken(olas).balanceOf(address(this)) - olasBalance;

1146:         for (uint256 j = firstClaimedEpoch; j < lastClaimedEpoch; ++j) {

1157:             totalReturnAmount += stakingPoint.stakingIncentive * stakingWeight;

1159:         totalReturnAmount /= 1e18;

1183:         uint256 withheldAmount = mapChainIdWithheldAmounts[chainId] + amount;

1221:             uint256 normalizedAmount = amount / (10 ** (18 - bridgingDecimals));

1222:             normalizedAmount *= 10 ** (18 - bridgingDecimals);

1228:         uint256 withheldAmount = mapChainIdWithheldAmounts[chainId] + amount;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

4: import {convert, UD60x18} from "@prb/math/src/UD60x18.sol";

5: import {TokenomicsConstants} from "./TokenomicsConstants.sol";

6: import {IDonatorBlacklist} from "./interfaces/IDonatorBlacklist.sol";

7: import {IErrorsTokenomics} from "./interfaces/IErrorsTokenomics.sol";

464:         if (block.timestamp >= (_timeLaunch + ONE_YEAR)) {

465:             revert Overflow(_timeLaunch + ONE_YEAR, block.timestamp);

469:         uint256 zeroYearSecondsLeft = uint32(_timeLaunch + ONE_YEAR - block.timestamp);

473:         uint256 _inflationPerSecond = getInflationForYear(0) / zeroYearSecondsLeft;

509:         uint256 _maxBond = (_inflationPerSecond * _epochLen * _maxBondFraction) / 100;

692:         emit TokenomicsParametersUpdateRequested(epochCounter + 1, _devsPerCapital, _codePerDev, _epsilonRate, _epochLen,

717:         if (_rewardComponentFraction + _rewardAgentFraction > 100) {

718:             revert WrongAmount(_rewardComponentFraction + _rewardAgentFraction, 100);

722:         uint256 sumTopUpFractions = _maxBondFraction + _topUpComponentFraction + _topUpAgentFraction +

729:         uint256 eCounter = epochCounter + 1;

735:         tp.epochPoint.rewardTreasuryFraction = uint8(100 - _rewardComponentFraction - _rewardAgentFraction);

772:         uint256 eCounter = epochCounter + 1;

798:             eBond -= amount;

814:         uint256 eBond = effectiveBond + amount;

833:         uint256 stakingIncentive = mapEpochStakingPoints[eCounter].stakingIncentive + amount;

854:             totalIncentives *= mapEpochTokenomics[epochNum].unitPoints[unitType].rewardUnitFraction;

856:             totalIncentives = mapUnitIncentives[unitType][unitId].reward + totalIncentives / 100;

869:             totalIncentives *= mapEpochTokenomics[epochNum].epochPoint.totalTopUpsOLAS;

870:             totalIncentives *= mapEpochTokenomics[epochNum].unitPoints[unitType].topUpUnitFraction;

871:             uint256 sumUnitIncentives = uint256(mapEpochTokenomics[epochNum].unitPoints[unitType].sumUnitTopUpsOLAS) * 100;

872:             totalIncentives = mapUnitIncentives[unitType][unitId].topUp + totalIncentives / sumUnitIncentives;

904:         for (uint256 i = 0; i < numServices; ++i) {

916:             for (uint256 unitType = 0; unitType < 2; ++unitType) {

926:                 if (incentiveFlags[unitType] || incentiveFlags[unitType + 2]) {

928:                     uint96 amount = uint96(amounts[i] / numServiceUnits);

930:                     for (uint256 j = 0; j < numServiceUnits; ++j) {

947:                             mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeReward += amount;

952:                         if (topUpEligible && incentiveFlags[unitType + 2]) {

953:                             mapUnitIncentives[unitType][serviceUnitIds[j]].pendingRelativeTopUp += amount;

954:                             mapEpochTokenomics[curEpoch].unitPoints[unitType].sumUnitTopUpsOLAS += amount;

960:                 for (uint256 j = 0; j < numServiceUnits; ++j) {

964:                         mapEpochTokenomics[curEpoch].unitPoints[unitType].numNewUnits++;

970:                             mapEpochTokenomics[curEpoch].epochPoint.numNewOwners++;

1010:         for (uint256 i = 0; i < numServices; ++i) {

1019:         donationETH += mapEpochTokenomics[curEpoch].epochPoint.totalDonationsETH;

1061:         idf = 1e18 + fKD;

1097:         uint256 prevEpochTime = mapEpochTokenomics[epochCounter - 1].epochPoint.endTime;

1098:         uint256 diffNumSeconds = block.timestamp - prevEpochTime;

1116:         incentives[1] = (incentives[0] * tp.epochPoint.rewardTreasuryFraction) / 100;

1118:         incentives[2] = (incentives[0] * tp.unitPoints[0].rewardUnitFraction) / 100;

1119:         incentives[3] = (incentives[0] * tp.unitPoints[1].rewardUnitFraction) / 100;

1126:         uint256 numYears = (block.timestamp - timeLaunch) / ONE_YEAR;

1132:             uint256 yearEndTime = timeLaunch + numYears * ONE_YEAR;

1134:             inflationPerEpoch = (yearEndTime - prevEpochTime) * curInflationPerSecond;

1136:             curInflationPerSecond = getInflationForYear(numYears) / ONE_YEAR;

1138:             inflationPerEpoch += (block.timestamp - yearEndTime) * curInflationPerSecond;

1146:             inflationPerEpoch = curInflationPerSecond * diffNumSeconds;

1152:         incentives[4] = (inflationPerEpoch * tp.epochPoint.maxBondFraction) / 100;

1166:             incentives[4] = effectiveBond + incentives[4] - curMaxBond;

1171:         TokenomicsPoint storage nextEpochPoint = mapEpochTokenomics[eCounter + 1];

1189:             emit TokenomicsParametersUpdated(eCounter + 1);

1196:             emit IncentiveFractionsUpdated(eCounter + 1);

1199:             for (uint256 i = 0; i < 2; ++i) {

1206:             mapEpochStakingPoints[eCounter + 1].stakingFraction = mapEpochStakingPoints[eCounter].stakingFraction;

1213:             emit StakingParamsUpdated(eCounter + 1);

1216:             mapEpochStakingPoints[eCounter + 1].maxStakingIncentive = mapEpochStakingPoints[eCounter].maxStakingIncentive;

1217:             mapEpochStakingPoints[eCounter + 1].minStakingWeight = mapEpochStakingPoints[eCounter].minStakingWeight;

1223:         uint256 accountRewards = incentives[2] + incentives[3];

1225:         incentives[5] = (inflationPerEpoch * tp.unitPoints[0].topUpUnitFraction) / 100;

1227:         incentives[6] = (inflationPerEpoch * tp.unitPoints[1].topUpUnitFraction) / 100;

1231:         uint256 accountTopUps = incentives[5] + incentives[6];

1237:         incentives[8] = incentives[7] + (inflationPerEpoch * mapEpochStakingPoints[eCounter].stakingFraction) / 100;

1243:         numYears = (block.timestamp + curEpochLen - timeLaunch) / ONE_YEAR;

1248:             uint256 yearEndTime = timeLaunch + numYears * ONE_YEAR;

1250:             inflationPerEpoch = (yearEndTime - block.timestamp) * curInflationPerSecond;

1252:             curInflationPerSecond = getInflationForYear(numYears) / ONE_YEAR;

1254:             inflationPerEpoch += (block.timestamp + curEpochLen - yearEndTime) * curInflationPerSecond;

1256:             curMaxBond = (inflationPerEpoch * nextEpochPoint.epochPoint.maxBondFraction) / 100;

1263:             curMaxBond = (curEpochLen * curInflationPerSecond * nextEpochPoint.epochPoint.maxBondFraction) / 100;

1270:         curMaxBond += effectiveBond;

1290:             epochCounter = uint32(eCounter + 1);

1329:         for (uint256 i = 0; i < 2; ++i) {

1335:         for (uint256 i = 0; i < unitIds.length; ++i) {

1357:         for (uint256 i = 0; i < unitIds.length; ++i) {

1370:             reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1373:             topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

1401:         for (uint256 i = 0; i < 2; ++i) {

1407:         for (uint256 i = 0; i < unitIds.length; ++i) {

1429:         for (uint256 i = 0; i < unitIds.length; ++i) {

1438:                     totalIncentives *= mapEpochTokenomics[lastEpoch].unitPoints[unitTypes[i]].rewardUnitFraction;

1440:                     reward += totalIncentives / 100;

1447:                     totalIncentives *= mapEpochTokenomics[lastEpoch].epochPoint.totalTopUpsOLAS;

1448:                     totalIncentives *= mapEpochTokenomics[lastEpoch].unitPoints[unitTypes[i]].topUpUnitFraction;

1449:                     uint256 sumUnitIncentives = uint256(mapEpochTokenomics[lastEpoch].unitPoints[unitTypes[i]].sumUnitTopUpsOLAS) * 100;

1451:                     topUp += totalIncentives / sumUnitIncentives;

1456:             reward += mapUnitIncentives[unitTypes[i]][unitIds[i]].reward;

1458:             topUp += mapUnitIncentives[unitTypes[i]][unitIds[i]].topUp;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

15:     uint256 public constant ONE_YEAR = 1 days * 365;

19:     uint256 public constant MAX_EPOCH_LENGTH = ONE_YEAR - 1 days;

51:             numYears -= 9;

58:             for (uint256 i = 0; i < numYears; ++i) {

59:                 supplyCap += (supplyCap * maxMintCapFraction) / 100;

88:             numYears -= 9;

95:             for (uint256 i = 1; i < numYears; ++i) {

96:                 supplyCap += (supplyCap * maxMintCapFraction) / 100;

100:             inflationAmount = (supplyCap * maxMintCapFraction) / 100;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/TokenomicsConstants.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol

4: import {DefaultDepositProcessorL1, IToken} from "./DefaultDepositProcessorL1.sol";

159:             cost[0] = maxSubmissionCostToken + TOKEN_GAS_LIMIT * gasPriceBid;

163:         cost[1] = maxSubmissionCostMessage + gasLimitMessage * gasPriceBid;

165:         uint256 totalCost = cost[0] + cost[1];

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol

4: import {DefaultTargetDispenserL2} from "./DefaultTargetDispenserL2.sol";

48:             l1AliasedDepositProcessor = address(uint160(_l1DepositProcessor) + offset);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/DefaultDepositProcessorL1.sol

4: import {IBridgeErrors} from "../interfaces/IBridgeErrors.sol";

30:     uint256 public constant MAX_CHAIN_ID = type(uint64).max / 2 - 36;

153:         stakingBatchNonce++;

179:         stakingBatchNonce++;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

4: import {IBridgeErrors} from "../interfaces/IBridgeErrors.sol";

64:     uint256 public constant MAX_CHAIN_ID = type(uint64).max / 2 - 36;

154:         for (uint256 i = 0; i < targets.length; ++i) {

172:                 localWithheldAmount += amount;

181:                 uint256 targetWithheldAmount = amount - limitAmount;

182:                 localWithheldAmount += targetWithheldAmount;

205:         stakingBatchNonce = batchNonce + 1;

209:             withheldAmount += localWithheldAmount;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

94:         for (uint256 i = 0; i < targets.length; ++i) {

108:                 uint256 refundAmount = amount - limitAmount;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)

```solidity
File: tokenomics/contracts/staking/GnosisDepositProcessorL1.sol

4: import {DefaultDepositProcessorL1, IToken} from "./DefaultDepositProcessorL1.sol";

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/GnosisTargetDispenserL2.sol

4: import "./DefaultTargetDispenserL2.sol";

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/OptimismDepositProcessorL1.sol

4: import {DefaultDepositProcessorL1, IToken} from "./DefaultDepositProcessorL1.sol";

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/OptimismTargetDispenserL2.sol

4: import {DefaultTargetDispenserL2} from "./DefaultTargetDispenserL2.sol";

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/PolygonDepositProcessorL1.sol

4: import {DefaultDepositProcessorL1, IToken} from "./DefaultDepositProcessorL1.sol";

5: import {FxBaseRootTunnel} from "../../lib/fx-portal/contracts/tunnel/FxBaseRootTunnel.sol";

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/PolygonTargetDispenserL2.sol

4: import {DefaultTargetDispenserL2} from "./DefaultTargetDispenserL2.sol";

5: import {FxBaseChildTunnel} from "../../lib/fx-portal/contracts/tunnel/FxBaseChildTunnel.sol";

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/WormholeDepositProcessorL1.sol

4: import {DefaultDepositProcessorL1} from "./DefaultDepositProcessorL1.sol";

5: import {TokenBase, TokenSender} from "wormhole-solidity-sdk/TokenBase.sol";

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/WormholeTargetDispenserL2.sol

4: import {DefaultTargetDispenserL2} from "./DefaultTargetDispenserL2.sol";

5: import {TokenBase, TokenReceiver} from "wormhole-solidity-sdk/TokenBase.sol";

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeTargetDispenserL2.sol)

### <a name="GAS-5"></a>[GAS-5] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including `EXTCODESIZE` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

*Instances (9)*:
```solidity
File: tokenomics/contracts/Dispenser.sol

815:                 olasBalance = IToken(olas).balanceOf(msg.sender);

822:                 olasBalance = IToken(olas).balanceOf(msg.sender) - olasBalance;

1028:                 uint256 olasBalance = IToken(olas).balanceOf(address(this));

1034:                 olasBalance = IToken(olas).balanceOf(address(this)) - olasBalance;

1106:                 uint256 olasBalance = IToken(olas).balanceOf(address(this));

1112:                 olasBalance = IToken(olas).balanceOf(address(this)) - olasBalance;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

189:             if (IToken(olas).balanceOf(address(this)) >= amount && localPaused == 1) {

287:         uint256 olasBalance = IToken(olas).balanceOf(address(this));

440:         uint256 amount = IToken(olas).balanceOf(address(this));

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

### <a name="GAS-6"></a>[GAS-6] `++i` costs less gas compared to `i++` or `i += 1` (same for `--i` vs `i--` or `i -= 1`)
Pre-increments and pre-decrements are cheaper.

For a `uint256 i` variable, the following is true with the Optimizer enabled at 10k:

**Increment:**

- `i += 1` is the most expensive form
- `i++` costs 6 gas less than `i += 1`
- `++i` costs 5 gas less than `i++` (11 gas less than `i += 1`)

**Decrement:**

- `i -= 1` is the most expensive form
- `i--` costs 11 gas less than `i -= 1`
- `--i` costs 5 gas less than `i--` (16 gas less than `i -= 1`)

Note that post-increments (or post-decrements) return the old value before incrementing or decrementing, hence the name *post-increment*:

```solidity
uint i = 1;  
uint j = 2;
require(j == i++, "This will be false as i is incremented after the comparison");
```
  
However, pre-increments (or pre-decrements) return the new value:
  
```solidity
uint i = 1;  
uint j = 2;
require(j == ++i, "This will be true as i is incremented before the comparison");
```

In the pre-increment case, the compiler has to create a temporary variable (when used) for returning `1` instead of `2`.

Consider using pre-increments and pre-decrements where they are relevant (meaning: not where post-increments/decrements logic are relevant).

*Saves 5 gas per instance*

*Instances (7)*:
```solidity
File: governance/contracts/VoteWeighting.sol

227:         for (uint256 i = 0; i < MAX_NUM_WEEKS; i++) {

267:         for (uint256 i = 0; i < MAX_NUM_WEEKS; i++) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

459:                 sCounter++;

466:             totalNumServices--;

685:                         numServices++;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: tokenomics/contracts/staking/DefaultDepositProcessorL1.sol

153:         stakingBatchNonce++;

179:         stakingBatchNonce++;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol)

### <a name="GAS-7"></a>[GAS-7] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (31)*:
```solidity
File: governance/contracts/VoteWeighting.sol

145:     uint256 public constant WEEK = 604_800;

148:     uint256 public constant WEIGHT_VOTE_DELAY = 864_000;

155:     uint256 public constant MAX_NUM_WEEKS = 53;

157:     uint256 public constant MAX_WEIGHT = 10_000;

159:     uint256 public constant MAX_EVM_CHAIN_ID = type(uint64).max / 2 - 36;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

224:     string public constant VERSION = "0.2.0";

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingFactory.sol

88:     uint256 public constant SELECTOR_DATA_LENGTH = 4;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: registries/contracts/staking/StakingProxy.sol

23:     bytes32 public constant SERVICE_STAKING_PROXY = 0x9e5e169c1098011e4e5940a3ec1797686b2a8294a9b77a4c676b121bdc0ebb5e;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingProxy.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

275:     uint256 public constant MAX_EVM_CHAIN_ID = type(uint64).max / 2 - 36;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

10:     string public constant VERSION = "1.2.0";

13:     bytes32 public constant PROXY_TOKENOMICS = 0xbd5523e7c3b6a94aa0e3b24d1120addc2f95c7029e097b466b2bedc8d4b4362f;

15:     uint256 public constant ONE_YEAR = 1 days * 365;

17:     uint256 public constant MIN_EPOCH_LENGTH = 10 days;

19:     uint256 public constant MAX_EPOCH_LENGTH = ONE_YEAR - 1 days;

21:     uint256 public constant MIN_PARAM_VALUE = 1e14;

23:     uint256 public constant MAX_STAKING_WEIGHT = 10_000;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/TokenomicsConstants.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol

72:     uint256 public constant BRIDGE_PAYLOAD_LENGTH = 160;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultDepositProcessorL1.sol

28:     bytes4 public constant RECEIVE_MESSAGE = bytes4(keccak256(bytes("receiveMessage(bytes)")));

30:     uint256 public constant MAX_CHAIN_ID = type(uint64).max / 2 - 36;

33:     uint256 public constant TOKEN_GAS_LIMIT = 300_000;

35:     uint256 public constant MESSAGE_GAS_LIMIT = 2_000_000;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

62:     bytes4 public constant RECEIVE_MESSAGE = bytes4(keccak256(bytes("receiveMessage(bytes)")));

64:     uint256 public constant MAX_CHAIN_ID = type(uint64).max / 2 - 36;

67:     uint256 public constant GAS_LIMIT = 300_000;

70:     uint256 public constant MAX_GAS_LIMIT = 2_000_000;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/GnosisDepositProcessorL1.sol

34:     uint256 public constant BRIDGE_PAYLOAD_LENGTH = 32;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/GnosisTargetDispenserL2.sol

28:     uint256 public constant BRIDGE_PAYLOAD_LENGTH = 32;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/OptimismDepositProcessorL1.sol

61:     uint256 public constant BRIDGE_PAYLOAD_LENGTH = 64;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/OptimismTargetDispenserL2.sol

39:     uint256 public constant BRIDGE_PAYLOAD_LENGTH = 64;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/WormholeDepositProcessorL1.sol

13:     uint256 public constant BRIDGE_PAYLOAD_LENGTH = 64;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/WormholeTargetDispenserL2.sol

52:     uint256 public constant BRIDGE_PAYLOAD_LENGTH = 64;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeTargetDispenserL2.sol)

### <a name="GAS-8"></a>[GAS-8] Use shift right/left instead of division/multiplication if possible
While the `DIV` / `MUL` opcode uses 5 gas, the `SHR` / `SHL` opcode only uses 3 gas. Furthermore, beware that Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting. Eventually, overflow checks are never performed for shift operations as they are done for arithmetic operations. Instead, the result is always truncated, so the calculation can be unchecked in Solidity version `0.8+`
- Use `>> 1` instead of `/ 2`
- Use `>> 2` instead of `/ 4`
- Use `<< 3` instead of `* 8`
- ...
- Use `>> 5` instead of `/ 2^5 == / 32`
- Use `<< 6` instead of `* 2^6 == * 64`

TL;DR:
- Shifting left by N is like multiplying by 2^N (Each bits to the left is an increased power of 2)
- Shifting right by N is like dividing by 2^N (Each bits to the right is a decreased power of 2)

*Saves around 2 gas + 20 for unchecked per instance*

*Instances (4)*:
```solidity
File: governance/contracts/VoteWeighting.sol

159:     uint256 public constant MAX_EVM_CHAIN_ID = type(uint64).max / 2 - 36;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

275:     uint256 public constant MAX_EVM_CHAIN_ID = type(uint64).max / 2 - 36;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/staking/DefaultDepositProcessorL1.sol

30:     uint256 public constant MAX_CHAIN_ID = type(uint64).max / 2 - 36;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

64:     uint256 public constant MAX_CHAIN_ID = type(uint64).max / 2 - 36;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

### <a name="GAS-9"></a>[GAS-9] `uint256` to `bool` `mapping`: Utilizing Bitmaps to dramatically save on Gas
https://soliditydeveloper.com/bitmaps

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/structs/BitMaps.sol

- [BitMaps.sol#L5-L16](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/structs/BitMaps.sol#L5-L16):

```solidity
/**
 * @dev Library for managing uint256 to bool mapping in a compact and efficient way, provided the keys are sequential.
 * Largely inspired by Uniswap's https://github.com/Uniswap/merkle-distributor/blob/master/contracts/MerkleDistributor.sol[merkle-distributor].
 *
 * BitMaps pack 256 booleans across each bit of a single 256-bit slot of `uint256` type.
 * Hence booleans corresponding to 256 _sequential_ indices would only consume a single slot,
 * unlike the regular `bool` which would consume an entire slot for a single value.
 *
 * This results in gas savings in two ways:
 *
 * - Setting a zero value to non-zero only once every 256 times
 * - Accessing the same warm slot for every 256 _sequential_ indices
 */
```

*Instances (1)*:
```solidity
File: tokenomics/contracts/Tokenomics.sol

367:     mapping(uint256 => mapping(uint256 => bool)) public mapNewUnits;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

### <a name="GAS-10"></a>[GAS-10] Increments/decrements can be unchecked in for-loops
In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

[ethereum/solidity#10695](https://github.com/ethereum/solidity/issues/10695)

The change would be:

```diff
- for (uint256 i; i < numIterations; i++) {
+ for (uint256 i; i < numIterations;) {
 // ...  
+   unchecked { ++i; }
}  
```

These save around **25 gas saved** per instance.

The same can be applied with decrements (which should use `break` when `i == 0`).

The risk of overflow is non-existent for `uint256`.

*Instances (44)*:
```solidity
File: governance/contracts/VoteWeighting.sol

227:         for (uint256 i = 0; i < MAX_NUM_WEEKS; i++) {

267:         for (uint256 i = 0; i < MAX_NUM_WEEKS; i++) {

573:         for (uint256 i = 0; i < accounts.length; ++i) {

793:         for (uint256 i = 0; i < accounts.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

332:         for (uint256 i = 0; i < _stakingParams.agentIds.length; ++i) {

380:         for (uint256 i = 0; i < numAgentIds; ++i) {

449:         for (uint256 i = 0; i < totalNumServices; ++i) {

464:         for (uint256 i = numEvictServices; i > 0; --i) {

548:             for (uint256 i = 0; i < size; ++i) {

620:                 for (uint256 i = 1; i < numServices; ++i) {

648:                 for (uint256 i = 0; i < numServices; ++i) {

669:             for (uint256 i = 0; i < serviceIds.length; ++i) {

773:             for (uint256 i = 0; i < numAgents; ++i) {

835:         for (; idx < serviceIds.length; ++idx) {

899:         for (uint256 i = 0; i < numServices; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingToken.sol

92:         for (uint256 i = 0; i < serviceAgentIds.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingToken.sol)

```solidity
File: registries/contracts/staking/StakingVerifier.sol

150:         for (uint256 i = 0; i < implementations.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

450:         for (uint256 i = 0; i < chainIds.length; ++i) {

462:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

473:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

485:                 for (uint256 j = 0; j < updatedStakingTargets.length; ++j) {

526:         for (uint256 i = 0; i < chainIds.length; ++i) {

550:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

593:         for (uint256 i = 0; i < chainIds.length; ++i) {

600:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

717:         for (uint256 i = 0; i < chainIds.length; ++i) {

881:         for (uint256 j = firstClaimedEpoch; j < lastClaimedEpoch; ++j) {

1146:         for (uint256 j = firstClaimedEpoch; j < lastClaimedEpoch; ++j) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

904:         for (uint256 i = 0; i < numServices; ++i) {

916:             for (uint256 unitType = 0; unitType < 2; ++unitType) {

930:                     for (uint256 j = 0; j < numServiceUnits; ++j) {

960:                 for (uint256 j = 0; j < numServiceUnits; ++j) {

1010:         for (uint256 i = 0; i < numServices; ++i) {

1199:             for (uint256 i = 0; i < 2; ++i) {

1329:         for (uint256 i = 0; i < 2; ++i) {

1335:         for (uint256 i = 0; i < unitIds.length; ++i) {

1357:         for (uint256 i = 0; i < unitIds.length; ++i) {

1401:         for (uint256 i = 0; i < 2; ++i) {

1407:         for (uint256 i = 0; i < unitIds.length; ++i) {

1429:         for (uint256 i = 0; i < unitIds.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

58:             for (uint256 i = 0; i < numYears; ++i) {

95:             for (uint256 i = 1; i < numYears; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/TokenomicsConstants.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

154:         for (uint256 i = 0; i < targets.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

94:         for (uint256 i = 0; i < targets.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)

### <a name="GAS-11"></a>[GAS-11] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (58)*:
```solidity
File: governance/contracts/VoteWeighting.sol

295:         if (mapNomineeIds[nomineeHash] > 0) {

300:         if (mapRemovedNominees[nomineeHash] > 0) {

430:         if (totalSum > 0) {

478:         if (mapRemovedNominees[nomineeHash] > 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingActivityChecker.sol

57:         if (ts > 0 && curNonces[0] > lastNonces[0]) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingActivityChecker.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

450:             if (evictServiceIds[i] > 0) {

464:         for (uint256 i = numEvictServices; i > 0; --i) {

536:         if (block.timestamp - tsCheckpointLast >= livenessPeriod && lastAvailableRewards > 0) {

612:         if (numServices > 0) {

665:         if (serviceIds.length > 0) {

676:                 if (serviceInactivity[i] > 0) {

696:             if (numServices > 0) {

728:         if (sInfo.tsStart > 0) {

751:         if (threshold > 0 && threshold != service.threshold) {

767:         if (size > 0) {

818:         if (ts <= minStakingDuration && availableRewards > 0) {

867:         if (reward > 0) {

932:         } else if (sInfo.tsStart > 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingFactory.sol

220:             if (returnData.length > 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

419:         if (transferAmount > 0) {

455:             if (transferAmounts[i] > 0) {

463:                 if (stakingIncentives[i][j] > 0) {

615:             if (transferAmounts[i] > 0) {

617:                 if (withheldAmount > 0) {

811:         if ((reward + topUp) > 0) {

814:             if (topUp > 0) {

821:             if (topUp > 0){

1000:         if (returnAmount > 0) {

1007:         if (stakingIncentive > 0) {

1013:             if (withheldAmount > 0) {

1027:             if (transferAmount > 0) {

1098:         if (totalAmounts[2] > 0) {

1103:         if (totalAmounts[0] > 0) {

1105:             if (totalAmounts[1] > 0) {

1161:         if (totalReturnAmount > 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

670:         if (_epsilonRate > 0 && _epsilonRate <= 17e18) {

684:         if (uint96(_veOLASThreshold) > 0) {

853:         if (totalIncentives > 0) {

866:         if (totalIncentives > 0) {

896:         incentiveFlags[0] = (mapEpochTokenomics[curEpoch].unitPoints[0].rewardUnitFraction > 0);

897:         incentiveFlags[1] = (mapEpochTokenomics[curEpoch].unitPoints[1].rewardUnitFraction > 0);

898:         incentiveFlags[2] = (mapEpochTokenomics[curEpoch].unitPoints[0].topUpUnitFraction > 0);

899:         incentiveFlags[3] = (mapEpochTokenomics[curEpoch].unitPoints[1].topUpUnitFraction > 0);

1176:             if (nextEpochLen > 0) {

1183:             if (nextVeOLASThreshold > 0) {

1261:         } else if (tokenomicsParametersUpdated > 0) {

1274:         if (incentives[0] > 0) {

1363:             if (lastEpoch > 0 && lastEpoch < curEpoch) {

1433:             if (lastEpoch > 0 && lastEpoch < curEpoch) {

1437:                 if (totalIncentives > 0) {

1444:                 if (totalIncentives > 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol

153:         if (transferAmount > 0) {

172:         if (transferAmount > 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

208:         if (localWithheldAmount > 0) {

442:         if (amount > 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/GnosisDepositProcessorL1.sol

63:         if (transferAmount > 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/OptimismDepositProcessorL1.sol

107:         if (transferAmount > 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/PolygonDepositProcessorL1.sol

64:         if (transferAmount > 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonDepositProcessorL1.sol)

### <a name="GAS-12"></a>[GAS-12] `internal` functions not called by the contract should be removed
If the functions are required by an interface, the contract should inherit from that interface and use the `override` keyword

*Instances (2)*:
```solidity
File: registries/contracts/utils/SafeTransferLib.sol

22:     function safeTransferFrom(address token, address from, address to, uint256 amount) internal {

64:     function safeTransfer(address token, address to, uint256 amount) internal {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/utils/SafeTransferLib.sol)


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Replace `abi.encodeWithSignature` and `abi.encodeWithSelector` with `abi.encodeCall` which keeps the code typo/type safe | 6 |
| [NC-2](#NC-2) | Use `string.concat()` or `bytes.concat()` instead of `abi.encodePacked` | 5 |
| [NC-3](#NC-3) | `constant`s should be defined rather than using magic numbers | 95 |
| [NC-4](#NC-4) | Control structures do not follow the Solidity Style Guide | 38 |
| [NC-5](#NC-5) | Functions should not be longer than 50 lines | 169 |
| [NC-6](#NC-6) | Change int to int256 | 24 |
| [NC-7](#NC-7) | `mapping` definitions do not follow the Solidity Style Guide | 1 |
| [NC-8](#NC-8) | Use a `modifier` instead of a `require/if` statement for a special `msg.sender` actor | 50 |
| [NC-9](#NC-9) | Consider using named mappings | 28 |
| [NC-10](#NC-10) | `address`s shouldn't be hard-coded | 1 |
| [NC-11](#NC-11) | Take advantage of Custom Error's return value property | 86 |
| [NC-12](#NC-12) | Avoid the use of sensitive terms | 17 |
| [NC-13](#NC-13) | Internal and private variables and functions names should begin with an underscore | 2 |
| [NC-14](#NC-14) | Constants should be defined rather than using magic numbers | 7 |
| [NC-15](#NC-15) | Variables need not be initialized to zero | 39 |
### <a name="NC-1"></a>[NC-1] Replace `abi.encodeWithSignature` and `abi.encodeWithSelector` with `abi.encodeCall` which keeps the code typo/type safe
When using `abi.encodeWithSignature`, it is possible to include a typo for the correct function signature.
When using `abi.encodeWithSignature` or `abi.encodeWithSelector`, it is also possible to provide parameters that are not of the correct type for the function.

To avoid these pitfalls, it would be best to use [`abi.encodeCall`](https://solidity-by-example.org/abi-encode/) instead.

*Instances (6)*:
```solidity
File: tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol

187:         bytes memory data = abi.encodeWithSelector(RECEIVE_MESSAGE, abi.encode(targets, stakingIncentives));

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol

55:         bytes memory data = abi.encodeWithSelector(RECEIVE_MESSAGE, abi.encode(amount));

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/GnosisDepositProcessorL1.sol

73:             bytes memory data = abi.encodeWithSelector(RECEIVE_MESSAGE, abi.encode(targets, stakingIncentives));

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/GnosisTargetDispenserL2.sol

77:         bytes memory data = abi.encodeWithSelector(RECEIVE_MESSAGE, abi.encode(amount));

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/OptimismDepositProcessorL1.sol

136:         bytes memory data = abi.encodeWithSelector(RECEIVE_MESSAGE, abi.encode(targets, stakingIncentives));

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/OptimismTargetDispenserL2.sol

85:         bytes memory data = abi.encodeWithSelector(RECEIVE_MESSAGE, abi.encode(amount));

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismTargetDispenserL2.sol)

### <a name="NC-2"></a>[NC-2] Use `string.concat()` or `bytes.concat()` instead of `abi.encodePacked`
Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

Solidity version 0.8.12 introduces `string.concat()` (vs `abi.encodePacked(<str>,<str>), which catches concatenation errors (in the event of a `bytes` data mixed in the concatenation)`)

*Instances (5)*:
```solidity
File: registries/contracts/staking/StakingFactory.sol

143:         bytes32 salt = keccak256(abi.encodePacked(block.chainid, localNonce));

146:         bytes memory deploymentData = abi.encodePacked(type(StakingProxy).creationCode,

151:             abi.encodePacked(

201:         bytes32 salt = keccak256(abi.encodePacked(block.chainid, localNonce));

203:         bytes memory deploymentData = abi.encodePacked(type(StakingProxy).creationCode,

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

### <a name="NC-3"></a>[NC-3] `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*Instances (95)*:
```solidity
File: registries/contracts/staking/StakingBase.sol

302:         if (_stakingParams.minStakingDeposit < 2) {

303:             revert LowerThan(_stakingParams.minStakingDeposit, 2);

411:         if (success && returnData.length > 63 && (returnData.length % 32 == 0)) {

420:             if (success && returnData.length == 32) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingFactory.sol

176:         _locked = 2;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: registries/contracts/staking/StakingVerifier.sol

212:             if (returnData.length == 32) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: registries/contracts/utils/SafeTransferLib.sol

39:                 or(and(eq(mload(0), 1), gt(returndatasize(), 31)), iszero(returndatasize())),

43:                 call(gas(), token, 0, 0, 100, 0, 32)

80:                 or(and(eq(mload(0), 1), gt(returndatasize(), 31)), iszero(returndatasize())),

84:                 call(gas(), token, 0, 0, 68, 0, 32)

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/utils/SafeTransferLib.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

741:             ITreasury(treasury).paused() == 2) {

797:         _locked = 2;

802:             ITreasury(treasury).paused() == 2) {

931:                 if (bridgingDecimals < 18) {

933:                     normalizedStakingAmount *= 10 ** (18 - bridgingDecimals);

963:         _locked = 2;

984:             ITreasury(treasury).paused() == 2) {

1069:         _locked = 2;

1081:             ITreasury(treasury).paused() == 2) {

1134:         _locked = 2;

1220:         if (bridgingDecimals < 18) {

1222:             normalizedAmount *= 10 ** (18 - bridgingDecimals);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

437:         veOLASThreshold = 10_000e18;

490:         tp.unitPoints[0].rewardUnitFraction = 83;

491:         tp.unitPoints[1].rewardUnitFraction = 17;

502:         uint256 _maxBondFraction = 50;

504:         tp.unitPoints[0].topUpUnitFraction = 41;

505:         tp.unitPoints[1].topUpUnitFraction = 9;

509:         uint256 _maxBond = (_inflationPerSecond * _epochLen * _maxBondFraction) / 100;

670:         if (_epsilonRate > 0 && _epsilonRate <= 17e18) {

717:         if (_rewardComponentFraction + _rewardAgentFraction > 100) {

718:             revert WrongAmount(_rewardComponentFraction + _rewardAgentFraction, 100);

724:         if (sumTopUpFractions > 100) {

725:             revert WrongAmount(sumTopUpFractions, 100);

856:             totalIncentives = mapUnitIncentives[unitType][unitId].reward + totalIncentives / 100;

871:             uint256 sumUnitIncentives = uint256(mapEpochTokenomics[epochNum].unitPoints[unitType].sumUnitTopUpsOLAS) * 100;

916:             for (uint256 unitType = 0; unitType < 2; ++unitType) {

926:                 if (incentiveFlags[unitType] || incentiveFlags[unitType + 2]) {

952:                         if (topUpEligible && incentiveFlags[unitType + 2]) {

1116:         incentives[1] = (incentives[0] * tp.epochPoint.rewardTreasuryFraction) / 100;

1118:         incentives[2] = (incentives[0] * tp.unitPoints[0].rewardUnitFraction) / 100;

1119:         incentives[3] = (incentives[0] * tp.unitPoints[1].rewardUnitFraction) / 100;

1152:         incentives[4] = (inflationPerEpoch * tp.epochPoint.maxBondFraction) / 100;

1199:             for (uint256 i = 0; i < 2; ++i) {

1225:         incentives[5] = (inflationPerEpoch * tp.unitPoints[0].topUpUnitFraction) / 100;

1227:         incentives[6] = (inflationPerEpoch * tp.unitPoints[1].topUpUnitFraction) / 100;

1237:         incentives[8] = incentives[7] + (inflationPerEpoch * mapEpochStakingPoints[eCounter].stakingFraction) / 100;

1256:             curMaxBond = (inflationPerEpoch * nextEpochPoint.epochPoint.maxBondFraction) / 100;

1263:             curMaxBond = (curEpochLen * curInflationPerSecond * nextEpochPoint.epochPoint.maxBondFraction) / 100;

1329:         for (uint256 i = 0; i < 2; ++i) {

1401:         for (uint256 i = 0; i < 2; ++i) {

1440:                     reward += totalIncentives / 100;

1449:                     uint256 sumUnitIncentives = uint256(mapEpochTokenomics[lastEpoch].unitPoints[unitTypes[i]].sumUnitTopUpsOLAS) * 100;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

35:         if (numYears < 10) {

37:                 529_659_000e18,

38:                 569_913_084e18,

39:                 610_313_084e18,

40:                 666_313_084e18,

41:                 746_313_084e18,

42:                 818_313_084e18,

43:                 882_313_084e18,

44:                 930_313_084e18,

45:                 970_313_084e18,

51:             numYears -= 9;

55:             uint256 maxMintCapFraction = 2;

59:                 supplyCap += (supplyCap * maxMintCapFraction) / 100;

71:         if (numYears < 10) {

74:                 3_159_000e18,

75:                 40_254_084e18,

76:                 40_400_000e18,

77:                 56_000_000e18,

78:                 80_000_000e18,

79:                 72_000_000e18,

80:                 64_000_000e18,

81:                 48_000_000e18,

82:                 40_000_000e18,

83:                 29_686_916e18

88:             numYears -= 9;

92:             uint256 maxMintCapFraction = 2;

96:                 supplyCap += (supplyCap * maxMintCapFraction) / 100;

100:             inflationAmount = (supplyCap * maxMintCapFraction) / 100;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/TokenomicsConstants.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol

141:         if (gasPriceBid < 2 || gasLimitMessage < 2 || maxSubmissionCostMessage == 0) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultDepositProcessorL1.sol

213:         return 18;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

144:         _locked = 2;

165:             if (success && returnData.length == 32) {

271:         _locked = 2;

274:         if (paused == 2) {

328:         _locked = 2;

331:         if (paused == 2) {

359:         paused = 2;

382:         _locked = 2;

417:         _locked = 2;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

91:         _locked = 2;

174:         return 18;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)

```solidity
File: tokenomics/contracts/staking/WormholeDepositProcessorL1.sol

139:         return 8;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeDepositProcessorL1.sol)

### <a name="NC-4"></a>[NC-4] Control structures do not follow the Solidity Style Guide
See the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) section of the Solidity Style Guide

*Instances (38)*:
```solidity
File: registries/contracts/staking/StakingBase.sol

287:         if (_stakingParams.metadataHash == 0 || _stakingParams.maxNumServices == 0 ||

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingFactory.sol

18:     function verifyImplementation(address implementation) external view returns (bool);

24:     function verifyInstance(address instance, address implementation) external view returns (bool);

58: error UnverifiedImplementation(address implementation);

62: error UnverifiedProxy(address instance);

83:     event VerifierUpdated(address indexed verifier);

94:     address public verifier;

105:         verifier = _verifier;

133:         verifier = newVerifier;

134:         emit VerifierUpdated(newVerifier);

194:         address localVerifier = verifier;

196:             revert UnverifiedImplementation(implementation);

232:             revert UnverifiedProxy(instance);

283:         address localVerifier = verifier;

285:             return IStakingVerifier(localVerifier).verifyInstance(instance, implementation);

296:         bool success = verifyInstance(instance);

303:             address localVerifier = verifier;

306:                 uint256 maxEmissions = IStakingVerifier(localVerifier).getEmissionsAmountLimit(instance);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: registries/contracts/staking/StakingVerifier.sol

166:     function verifyImplementation(address implementation) external view returns (bool){

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

330:         if (_olas == address(0) || _tokenomics == address(0) || _treasury == address(0) ||

740:         if (currentPause == Pause.StakingIncentivesPaused || currentPause == Pause.AllPaused ||

801:         if (currentPause == Pause.DevIncentivesPaused || currentPause == Pause.AllPaused ||

821:             if (topUp > 0){

903:             uint256 stakingDiff;

905:                 stakingDiff = availableStakingAmount - totalWeightSum;

913:                 returnAmount = ((stakingDiff + availableStakingAmount) * stakingWeight) / 1e18;

918:                 returnAmount = (stakingDiff * stakingWeight) / 1e18;

983:         if (currentPause == Pause.StakingIncentivesPaused || currentPause == Pause.AllPaused ||

1080:         if (currentPause == Pause.StakingIncentivesPaused || currentPause == Pause.AllPaused ||

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

427:         if (_olas == address(0) || _treasury == address(0) || _depository == address(0) || _dispenser == address(0) ||

1098:         uint256 diffNumSeconds = block.timestamp - prevEpochTime;

1146:             inflationPerEpoch = curInflationPerSecond * diffNumSeconds;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

18:     function verifyInstanceAndGetEmissionsAmount(address instance) external view returns (uint256 amount);

109:         if (_olas == address(0) || _stakingFactory == address(0) || _l2MessageRelayer == address(0)

160:             bytes memory verifyData = abi.encodeCall(IStakingFactory.verifyInstanceAndGetEmissionsAmount, target);

161:             (bool success, bytes memory returnData) = stakingFactory.call(verifyData);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

28:     function verifyInstanceAndGetEmissionsAmount(address instance) external view returns (uint256 amount);

99:             uint256 limitAmount = IStakingFactory(stakingFactory).verifyInstanceAndGetEmissionsAmount(target);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)

### <a name="NC-5"></a>[NC-5] Functions should not be longer than 50 lines
Overly complex code can make understanding functionality more difficult, try to further modularize your code to ensure readability 

*Instances (169)*:
```solidity
File: governance/contracts/VoteWeighting.sol

8:     function addNominee(bytes32 nomineeHash) external;

12:     function removeNominee(bytes32 nomineeHash) external;

36:     function lockedEnd(address account) external view returns (uint256 unlockTime);

41:     function getLastUserPoint(address account) external view returns (PointVoting memory pv);

254:     function _getWeight(bytes32 account, uint256 chainId) internal returns (uint256) {

292:     function _addNominee(Nominee memory nominee) internal {

324:     function addNomineeEVM(address account, uint256 chainId) external {

349:     function addNomineeNonEVM(bytes32 account, uint256 chainId) external {

386:     function changeDispenser(address newDispenser) external {

406:     function checkpointNominee(bytes32 account, uint256 chainId) external {

473:     function voteForNomineeWeights(bytes32 account, uint256 chainId, uint256 weight) public {

578:     function _maxAndSub(uint256 a, uint256 b) internal pure returns (uint256) {

586:     function removeNominee(bytes32 account, uint256 chainId) external {

641:     function revokeRemovedNomineeVotingPower(bytes32 account, uint256 chainId) external {

674:     function getNomineeWeight(bytes32 account, uint256 chainId) external view returns (uint256) {

684:     function getWeightsSum() external view returns (uint256) {

691:     function getNumNominees() external view returns (uint256) {

698:     function getNumRemovedNominees() external view returns (uint256) {

705:     function getAllNominees() external view returns (Nominee[] memory) {

712:     function getAllRemovedNominees() external view returns (Nominee[] memory) {

720:     function getNomineeId(bytes32 account, uint256 chainId) external view returns (uint256) {

732:     function getRemovedNomineeId(bytes32 account, uint256 chainId) external view returns (uint256) {

744:     function getNominee(uint256 id) external view returns (Nominee memory) {

761:     function getRemovedNominee(uint256 id) external view returns (Nominee memory) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingActivityChecker.sol

36:     function getMultisigNonces(address multisig) external view virtual returns (uint256[] memory nonces) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingActivityChecker.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

11:     function getMultisigNonces(address multisig) external view returns (uint256[] memory nonces);

72:     function safeTransferFrom(address from, address to, uint256 id) external;

77:     function getService(uint256 serviceId) external view returns (Service memory service);

83:     function getAgentParams(uint256 serviceId) external view

390:     function _withdraw(address to, uint256 amount) internal virtual;

482:     function _claim(uint256 serviceId, bool execCheckPoint) internal returns (uint256 reward) {

522:     function _calculateStakingRewards() internal view returns (

805:     function unstake(uint256 serviceId) external returns (uint256 reward) {

877:     function claim(uint256 serviceId) external returns (uint256) {

884:     function checkpointAndClaim(uint256 serviceId) external returns (uint256) {

893:     function calculateStakingLastReward(uint256 serviceId) public view returns (uint256 reward) {

916:     function calculateStakingReward(uint256 serviceId) external view returns (uint256 reward) {

928:     function getStakingState(uint256 serviceId) external view returns (StakingState stakingState) {

939:     function getNextRewardCheckpointTimestamp() external view returns (uint256 tsNext) {

947:     function getServiceInfo(uint256 serviceId) external view returns (ServiceInfo memory sInfo) {

953:     function getServiceIds() public view returns (uint256[] memory) {

959:     function getAgentIds() external view returns (uint256[] memory) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingFactory.sol

10:     function emissionsAmount() external view returns (uint256);

18:     function verifyImplementation(address implementation) external view returns (bool);

24:     function verifyInstance(address instance, address implementation) external view returns (bool);

28:     function getEmissionsAmountLimit(address) external view returns (uint256);

127:     function changeVerifier(address newVerifier) external {

141:     function getProxyAddressWithNonce(address implementation, uint256 localNonce) public view returns (address) {

161:     function getProxyAddress(address implementation) external view returns (address) {

249:     function setInstanceStatus(address instance, bool isEnabled) external {

267:     function verifyInstance(address instance) public view returns (bool) {

294:     function verifyInstanceAndGetEmissionsAmount(address instance) external view returns (uint256 amount) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: registries/contracts/staking/StakingNativeToken.sol

20:     function initialize(StakingParams memory _stakingParams) external {

28:     function _withdraw(address to, uint256 amount) internal override {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingNativeToken.sol)

```solidity
File: registries/contracts/staking/StakingProxy.sol

54:     function getImplementation() external view returns (address implementation) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingProxy.sol)

```solidity
File: registries/contracts/staking/StakingToken.sol

16:     function mapServiceIdTokenDeposit(uint256 serviceId) external view returns (address, uint96);

22:     function getAgentBond(uint256 serviceId, uint256 agentId) external view returns (uint256 bond);

74:     function _checkTokenStakingDeposit(uint256 serviceId, uint256, uint32[] memory serviceAgentIds) internal view override {

104:     function _withdraw(address to, uint256 amount) internal override {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingToken.sol)

```solidity
File: registries/contracts/staking/StakingVerifier.sol

8:     function rewardsPerSecond() external view returns (uint256);

12:     function maxNumServices() external view returns (uint256);

16:     function stakingToken() external view returns (address);

113:     function setImplementationsCheck(bool setCheck) external {

166:     function verifyImplementation(address implementation) external view returns (bool){

179:     function verifyInstance(address instance, address implementation) external view returns (bool) {

255:     function getEmissionsAmountLimit(address) external view returns (uint256) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: registries/contracts/utils/SafeTransferLib.sol

22:     function safeTransferFrom(address token, address from, address to, uint256 amount) internal {

64:     function safeTransfer(address token, address to, uint256 amount) internal {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/utils/SafeTransferLib.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

11:     function sendMessage(address target, uint256 stakingIncentive, bytes memory bridgePayload,

19:     function sendMessageBatch(address[] memory targets, uint256[] memory stakingIncentives, bytes memory bridgePayload,

27:     function sendMessageNonEVM(bytes32 target, uint256 stakingIncentive, bytes memory bridgePayload,

35:     function sendMessageBatchNonEVM(bytes32[] memory targets, uint256[] memory stakingIncentives,

40:     function getBridgingDecimals() external pure returns (uint256);

48:     function balanceOf(address account) external view returns (uint256);

54:     function transfer(address to, uint256 amount) external returns (bool);

115:     function accountOwnerIncentives(address account, uint256[] memory unitTypes, uint256[] memory unitIds) external

120:     function epochCounter() external view returns (uint32);

124:     function epochLen() external view returns (uint32);

129:     function getEpochEndTime(uint256 epoch) external view returns (uint256 endTime);

134:     function mapEpochStakingPoints(uint256 eCounter) external view returns (StakingPoint memory);

138:     function refundFromStaking(uint256 amount) external;

151:     function withdrawToAccount(address account, uint256 accountRewards, uint256 accountTopUps) external

170:     function mapNomineeIds(bytes32 nomineeHash) external returns (uint256);

175:     function checkpointNominee(bytes32 account, uint256 chainId) external;

185:     function nomineeRelativeWeight(bytes32 account, uint256 chainId, uint256 time) external view returns (uint256, uint256);

356:     function _checkpointNomineeAndGetClaimedEpochCounters(

655:     function changeManagers(address _tokenomics, address _treasury, address _voteWeighting) external {

683:     function changeStakingParams(uint256 _maxNumClaimingEpochs, uint256 _maxNumStakingTargets) external {

705:     function setDepositProcessorChainIds(address[] memory depositProcessors, uint256[] memory chainIds) external {

732:     function addNominee(bytes32 nomineeHash) external {

750:     function removeNominee(bytes32 nomineeHash) external {

1174:     function syncWithheldAmount(uint256 chainId, uint256 amount) external {

1199:     function syncWithheldAmountMaintenance(uint256 chainId, uint256 amount) external {

1241:     function setPauseState(Pause pauseState) external {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

13:     function timeLaunch() external view returns (uint256);

21:     function ownerOf(uint256 tokenId) external view returns (address);

25:     function totalSupply() external view returns (uint256);

33:     function rebalanceTreasury(uint256 treasuryRewards) external returns (bool success);

46:     function exists(uint256 serviceId) external view returns (bool);

53:     function getUnitIdsOfService(UnitType unitType, uint256 serviceId) external view

61:     function getVotes(address account) external view returns (uint256);

516:     function tokenomicsImplementation() external view returns (address implementation) {

526:     function changeTokenomicsImplementation(address implementation) external {

565:     function changeManagers(address _treasury, address _depository, address _dispenser) external {

592:     function changeRegistries(address _componentRegistry, address _agentRegistry, address _serviceRegistry) external {

616:     function changeDonatorBlacklist(address _donatorBlacklist) external {

751:     function changeStakingParams(uint256 _maxStakingIncentive, uint256 _minStakingWeight) external {

787:     function reserveAmountForBondProgram(uint256 amount) external returns (bool success) {

808:     function refundFromBondProgram(uint256 amount) external {

826:     function refundFromStaking(uint256 amount) external {

847:     function _finalizeIncentivesForUnitId(uint256 epochNum, uint256 unitType, uint256 unitId) internal {

1033:     function _calculateIDF(uint256 treasuryRewards, uint256 numNewOwners) internal view returns (uint256 idf) {

1466:     function getUnitPoint(uint256 epoch, uint256 unitType) external view returns (UnitPoint memory) {

1472:     function getLastIDF() external view returns (uint256) {

1479:     function getEpochEndTime(uint256 epoch) external view returns (uint256) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

33:     function getSupplyCapForYear(uint256 numYears) public pure returns (uint256 supplyCap) {

69:     function getInflationForYear(uint256 numYears) public pure returns (uint256 inflationAmount) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/TokenomicsConstants.sol)

```solidity
File: tokenomics/contracts/interfaces/IDonatorBlacklist.sol

9:     function isDonatorBlacklisted(address account) external view returns (bool status);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/interfaces/IDonatorBlacklist.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol

63:     function l2ToL1Sender() external view returns (address);

196:     function receiveMessage(bytes memory data) external {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol

16:     function sendTxToL1(address destination, bytes calldata data) external payable returns (uint256);

53:     function _sendMessage(uint256 amount, bytes memory) internal override {

66:     function receiveMessage(bytes memory data) external payable {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/DefaultDepositProcessorL1.sol

7:     function syncWithheldAmount(uint256 chainId, uint256 amount) external;

15:     function approve(address spender, uint256 amount) external returns (bool);

107:     function _receiveMessage(address l1Relayer, address l2Dispenser, bytes memory data) internal virtual {

186:     function _setL2TargetDispenser(address l2Dispenser) internal {

206:     function setL2TargetDispenser(address l2Dispenser) external virtual {

212:     function getBridgingDecimals() external pure virtual returns (uint256) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

18:     function verifyInstanceAndGetEmissionsAmount(address instance) external view returns (uint256 amount);

26:     function balanceOf(address account) external view returns (uint256);

32:     function approve(address spender, uint256 amount) external returns (bool);

38:     function transfer(address to, uint256 amount) external returns (bool);

139:     function _processData(bytes memory data) internal {

218:     function _sendMessage(uint256 amount, bytes memory bridgePayload) internal virtual;

266:     function redeem(address target, uint256 amount, uint256 batchNonce) external {

311:     function processDataMaintenance(bytes memory data) external {

323:     function syncWithheldTokens(bytes memory bridgePayload) external payable {

377:     function drain() external returns (uint256 amount) {

412:     function migrate(address newL2TargetDispenser) external {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

9:     function approve(address spender, uint256 amount) external returns (bool);

15:     function transfer(address to, uint256 amount) external returns (bool);

28:     function verifyInstanceAndGetEmissionsAmount(address instance) external view returns (uint256 amount);

86:     function _deposit(address[] memory targets, uint256[] memory stakingIncentives) internal {

173:     function getBridgingDecimals() external pure returns (uint256) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)

```solidity
File: tokenomics/contracts/staking/GnosisDepositProcessorL1.sol

15:     function requireToPassMessage(address target, bytes memory data, uint256 maxGasLimit) external returns (bytes32);

21:     function relayTokensAndCall(address token, address receiver, uint256 amount, bytes memory payload) external;

25:     function messageSender() external returns (address);

98:     function receiveMessage(bytes memory data) external {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/GnosisTargetDispenserL2.sol

15:     function requireToPassMessage(address target, bytes memory data, uint256 maxGasLimit) external returns (bytes32);

19:     function messageSender() external returns (address);

58:     function _sendMessage(uint256 amount, bytes memory bridgePayload) internal override {

87:     function receiveMessage(bytes memory data) external {

99:     function onTokenBridged(address, uint256, bytes calldata data) external {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/OptimismDepositProcessorL1.sol

52:     function xDomainMessageSender() external view returns (address);

148:     function receiveMessage(bytes memory data) external payable {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/OptimismTargetDispenserL2.sol

30:     function xDomainMessageSender() external view returns (address);

56:     function _sendMessage(uint256 amount, bytes memory bridgePayload) internal override {

96:     function receiveMessage(bytes memory data) external payable {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/PolygonDepositProcessorL1.sol

15:     function depositFor(address user, address rootToken, bytes calldata depositData) external;

91:     function _processMessageFromChild(bytes memory data) internal override {

98:     function setFxChildTunnel(address l2Dispenser) public override {

117:     function setL2TargetDispenser(address l2Dispenser) external override {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/PolygonTargetDispenserL2.sol

32:     function _sendMessage(uint256 amount, bytes memory) internal override {

52:     function _processMessageFromRoot(uint256, address sender, bytes memory data) internal override {

59:     function setFxRootTunnel(address l1Processor) external override {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/WormholeDepositProcessorL1.sol

132:     function setL2TargetDispenser(address l2Dispenser) external override {

138:     function getBridgingDecimals() external pure override returns (uint256) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/WormholeTargetDispenserL2.sol

89:     function _sendMessage(uint256 amount, bytes memory bridgePayload) internal override {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeTargetDispenserL2.sol)

### <a name="NC-6"></a>[NC-6] Change int to int256
Throughout the code base, some variables are declared as `int`. To favor explicitness, consider changing all instances of `int` to `int256`

*Instances (24)*:
```solidity
File: governance/contracts/VoteWeighting.sol

108: struct Point {

226:         Point memory pt = pointsSum[t];

266:         Point memory pt = pointsWeight[nomineeHash][t];

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

360:         tsCheckpoint = block.timestamp;

557:                 uint256 serviceCheckpoint = tsCheckpointLast;

561:                     serviceCheckpoint = ts;

703:             tsCheckpoint = block.timestamp;

941:         tsNext = tsCheckpoint + livenessPeriod;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

61:     struct EpochPoint {

90:     struct StakingPoint {

134:     function mapEpochStakingPoints(uint256 eCounter) external view returns (StakingPoint memory);

883:             ITokenomics.StakingPoint memory stakingPoint =

1148:             ITokenomics.StakingPoint memory stakingPoint = ITokenomics(tokenomics).mapEpochStakingPoints(j);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

160: struct UnitPoint {

177: struct EpochPoint {

207: struct TokenomicsPoint {

212:     EpochPoint epochPoint;

233: struct StakingPoint {

482:         TokenomicsPoint storage tp = mapEpochTokenomics[1];

730:         TokenomicsPoint storage tp = mapEpochTokenomics[eCounter];

773:         StakingPoint storage stakingPoint = mapEpochStakingPoints[eCounter];

1107:         TokenomicsPoint storage tp = mapEpochTokenomics[eCounter];

1171:         TokenomicsPoint storage nextEpochPoint = mapEpochTokenomics[eCounter + 1];

1466:     function getUnitPoint(uint256 epoch, uint256 unitType) external view returns (UnitPoint memory) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

### <a name="NC-7"></a>[NC-7] `mapping` definitions do not follow the Solidity Style Guide
See the [mappings](https://docs.soliditylang.org/en/latest/style-guide.html#mappings) section of the Solidity Style Guide

*Instances (1)*:
```solidity
File: registries/contracts/staking/StakingBase.sol

272:     mapping (uint256 => ServiceInfo) public mapServiceInfo;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

### <a name="NC-8"></a>[NC-8] Use a `modifier` instead of a `require/if` statement for a special `msg.sender` actor
If a function is supposed to be access-controlled, a `modifier` should be used instead of a `require/if` statement for more readability.

*Instances (50)*:
```solidity
File: governance/contracts/VoteWeighting.sol

370:         if (msg.sender != owner) {

388:         if (msg.sender != owner) {

588:         if (msg.sender != owner) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

485:         if (msg.sender != sInfo.owner) {

808:         if (msg.sender != sInfo.owner) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingFactory.sol

112:         if (msg.sender != owner) {

129:         if (msg.sender != owner) {

255:         if (msg.sender != deployer) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: registries/contracts/staking/StakingVerifier.sol

98:         if (msg.sender != owner) {

115:         if (owner != msg.sender) {

137:         if (owner != msg.sender) {

235:         if (owner != msg.sender) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

638:         if (msg.sender != owner) {

657:         if (msg.sender != owner) {

685:         if (msg.sender != owner) {

707:         if (msg.sender != owner) {

734:         if (msg.sender != voteWeighting) {

752:         if (msg.sender != voteWeighting) {

1178:         if (msg.sender != depositProcessor) {

1201:         if (msg.sender != owner) {

1243:         if (msg.sender != owner) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

528:         if (msg.sender != owner) {

548:         if (msg.sender != owner) {

567:         if (msg.sender != owner) {

594:         if (msg.sender != owner) {

618:         if (msg.sender != owner) {

647:         if (msg.sender != owner) {

712:         if (msg.sender != owner) {

753:         if (msg.sender != owner) {

789:         if (depository != msg.sender) {

810:         if (depository != msg.sender) {

828:         if (dispenser != msg.sender) {

997:         if (treasury != msg.sender) {

1314:         if (dispenser != msg.sender) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol

198:         if (msg.sender != bridge) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol

68:         if (msg.sender != l1AliasedDepositProcessor) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/DefaultDepositProcessorL1.sol

139:         if (msg.sender != l1Dispenser) {

171:         if (msg.sender != l1Dispenser) {

188:         if (msg.sender != owner) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

249:         if (msg.sender != owner) {

313:         if (msg.sender != owner) {

355:         if (msg.sender != owner) {

366:         if (msg.sender != owner) {

385:         if (msg.sender != owner) {

420:         if (msg.sender != owner) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

137:         if (msg.sender != dispenser) {

162:         if (msg.sender != dispenser) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)

```solidity
File: tokenomics/contracts/staking/GnosisTargetDispenserL2.sol

101:         if (msg.sender != l2TokenRelayer) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/PolygonDepositProcessorL1.sol

100:         if (msg.sender != owner) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/PolygonTargetDispenserL2.sol

61:         if (msg.sender != owner) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonTargetDispenserL2.sol)

### <a name="NC-9"></a>[NC-9] Consider using named mappings
Consider moving to solidity version 0.8.18 or later, and using [named mappings](https://ethereum.stackexchange.com/questions/51629/how-to-name-the-arguments-in-mapping/145555#145555) to make it easier to understand the purpose of each mapping

*Instances (28)*:
```solidity
File: governance/contracts/VoteWeighting.sol

172:     mapping(bytes32 => uint256) public mapNomineeIds;

174:     mapping(bytes32 => uint256) public mapRemovedNominees;

177:     mapping(address => mapping(bytes32 => VotedSlope)) public voteUserSlopes;

179:     mapping(address => uint256) public voteUserPower;

181:     mapping(address => mapping(bytes32 => uint256)) public lastUserVote;

190:     mapping(bytes32 => mapping(uint256 => Point)) public pointsWeight;

192:     mapping(bytes32 => mapping(uint256 => uint256)) public changesWeight;

194:     mapping(bytes32 => uint256) public timeWeight;

197:     mapping(uint256 => Point) public pointsSum;

199:     mapping(uint256 => uint256) public changesSum;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingFactory.sol

99:     mapping(address => InstanceParams) public mapInstanceParams;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: registries/contracts/staking/StakingVerifier.sol

67:     mapping(address => bool) public mapImplementations;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

302:     mapping(bytes32 => uint256) public mapLastClaimedStakingEpochs;

304:     mapping(bytes32 => uint256) public mapRemovedNomineeEpochs;

306:     mapping(uint256 => address) public mapChainIdDepositProcessors;

308:     mapping(uint256 => uint256) public mapChainIdWithheldAmounts;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

210:     mapping(uint256 => UnitPoint) unitPoints;

359:     mapping(uint256 => uint256) public mapServiceAmounts;

361:     mapping(address => uint256) public mapOwnerRewards;

363:     mapping(address => uint256) public mapOwnerTopUps;

365:     mapping(uint256 => TokenomicsPoint) public mapEpochTokenomics;

367:     mapping(uint256 => mapping(uint256 => bool)) public mapNewUnits;

369:     mapping(address => bool) public mapNewOwners;

371:     mapping(uint256 => mapping(uint256 => IncentiveBalances)) public mapUnitIncentives;

374:     mapping(uint256 => StakingPoint) public mapEpochStakingPoints;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

93:     mapping(bytes32 => bool) public stakingQueueingNonces;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/WormholeDepositProcessorL1.sol

18:     mapping(bytes32 => bool) public mapDeliveryHashes;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/WormholeTargetDispenserL2.sol

54:     mapping(bytes32 => bool) public mapDeliveryHashes;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeTargetDispenserL2.sol)

### <a name="NC-10"></a>[NC-10] `address`s shouldn't be hard-coded
It is often better to declare `address`es as `immutable`, and assign them via constructor arguments. This allows the code to remain the same across deployments on different networks, and avoids recompilation when addresses need to change.

*Instances (1)*:
```solidity
File: tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol

46:         uint160 offset = uint160(0x1111000000000000000000000000000000001111);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol)

### <a name="NC-11"></a>[NC-11] Take advantage of Custom Error's return value property
An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

*Instances (86)*:
```solidity
File: governance/contracts/VoteWeighting.sol

208:             revert ZeroAddress();

327:             revert ZeroAddress();

332:             revert ZeroValue();

352:             revert ZeroAddress();

376:             revert ZeroAddress();

654:             revert ZeroValue();

749:             revert ZeroValue();

766:             revert ZeroValue();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingActivityChecker.sol

27:             revert ZeroValue();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingActivityChecker.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

283:             revert AlreadyInitialized();

291:             revert ZeroValue();

306:             revert ZeroAddress();

343:             revert ZeroValue();

499:             revert ZeroValue();

722:             revert NoRewardsAvailable();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingFactory.sol

118:             revert ZeroAddress();

174:             revert ReentrancyGuard();

180:             revert ZeroAddress();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: registries/contracts/staking/StakingProxy.sol

30:             revert ZeroImplementationAddress();

47:                 revert(0, returndatasize())

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingProxy.sol)

```solidity
File: registries/contracts/staking/StakingToken.sol

64:             revert ZeroTokenAddress();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingToken.sol)

```solidity
File: registries/contracts/staking/StakingVerifier.sol

78:             revert ZeroAddress();

83:             revert ZeroValue();

104:             revert ZeroAddress();

153:                 revert ZeroAddress();

241:             revert ZeroValue();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

332:             revert ZeroAddress();

337:             revert ZeroValue();

370:             revert ZeroValue();

536:                 revert ZeroValue();

644:             revert ZeroAddress();

691:             revert ZeroValue();

720:                 revert ZeroValue();

742:             revert Paused();

795:             revert ReentrancyGuard();

803:             revert Paused();

862:             revert ZeroValue();

867:             revert ZeroAddress();

961:             revert ReentrancyGuard();

967:             revert ZeroValue();

972:             revert ZeroAddress();

985:             revert Paused();

1067:             revert ReentrancyGuard();

1082:             revert Paused();

1132:             revert ReentrancyGuard();

1207:             revert ZeroValue();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

423:             revert AlreadyInitialized();

430:             revert ZeroAddress();

534:             revert ZeroAddress();

554:             revert ZeroAddress();

759:             revert ZeroValue();

1088:             revert DelegatecallOnly();

1093:             revert SameBlockNumberViolation();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol

103:             revert ZeroAddress();

142:             revert ZeroValue();

155:                 revert ZeroValue();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultDepositProcessorL1.sol

68:             revert ZeroAddress();

73:             revert ZeroValue();

194:             revert ZeroAddress();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

111:             revert ZeroAddress();

116:             revert ZeroValue();

142:             revert ReentrancyGuard();

255:             revert ZeroAddress();

269:             revert ReentrancyGuard();

275:             revert Paused();

326:             revert ReentrancyGuard();

332:             revert Paused();

338:             revert ZeroValue();

380:             revert ReentrancyGuard();

392:             revert ZeroValue();

415:             revert ReentrancyGuard();

426:             revert Unpaused();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

73:             revert ZeroAddress();

89:             revert ReentrancyGuard();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)

```solidity
File: tokenomics/contracts/staking/GnosisDepositProcessorL1.sol

81:                 revert ZeroValue();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/GnosisTargetDispenserL2.sol

51:             revert ZeroAddress();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/OptimismDepositProcessorL1.sol

88:             revert ZeroAddress();

122:             revert ZeroValue();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/OptimismTargetDispenserL2.sol

67:             revert ZeroValue();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/PolygonDepositProcessorL1.sol

50:             revert ZeroAddress();

106:             revert ZeroAddress();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/PolygonTargetDispenserL2.sol

67:             revert ZeroAddress();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/WormholeDepositProcessorL1.sol

42:             revert ZeroAddress();

47:             revert ZeroValue();

75:             revert ZeroValue();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/WormholeTargetDispenserL2.sol

78:             revert ZeroAddress();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeTargetDispenserL2.sol)

### <a name="NC-12"></a>[NC-12] Avoid the use of sensitive terms
Use [alternative variants](https://www.zdnet.com/article/mysql-drops-master-slave-and-blacklist-whitelist-terminology/), e.g. allowlist/denylist instead of whitelist/blacklist

*Instances (17)*:
```solidity
File: registries/contracts/staking/StakingVerifier.sol

46:     event ImplementationsWhitelistUpdated(address[] implementations, bool[] statuses, bool setCheck);

160:         emit ImplementationsWhitelistUpdated(implementations, statuses, setCheck);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

6: import {IDonatorBlacklist} from "./interfaces/IDonatorBlacklist.sol";

110: error DonatorBlacklisted(address account);

276:     event DonatorBlacklistUpdated(address indexed blacklist);

353:     address public donatorBlacklist;

419:         address _donatorBlacklist

459:         donatorBlacklist = _donatorBlacklist;

616:     function changeDonatorBlacklist(address _donatorBlacklist) external {

622:         donatorBlacklist = _donatorBlacklist;

623:         emit DonatorBlacklistUpdated(_donatorBlacklist);

1002:         address bList = donatorBlacklist;

1003:         if (bList != address(0) && IDonatorBlacklist(bList).isDonatorBlacklisted(donator)) {

1004:             revert DonatorBlacklisted(donator);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/interfaces/IDonatorBlacklist.sol

5: interface IDonatorBlacklist {

9:     function isDonatorBlacklisted(address account) external view returns (bool status);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/interfaces/IDonatorBlacklist.sol)

```solidity
File: tokenomics/contracts/interfaces/IErrorsTokenomics.sol

116:     error DonatorBlacklisted(address account);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/interfaces/IErrorsTokenomics.sol)

### <a name="NC-13"></a>[NC-13] Internal and private variables and functions names should begin with an underscore
According to the Solidity Style Guide, Non-`external` variable and function names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)

*Instances (2)*:
```solidity
File: registries/contracts/utils/SafeTransferLib.sol

22:     function safeTransferFrom(address token, address from, address to, uint256 amount) internal {

64:     function safeTransfer(address token, address to, uint256 amount) internal {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/utils/SafeTransferLib.sol)

### <a name="NC-14"></a>[NC-14] Constants should be defined rather than using magic numbers

*Instances (7)*:
```solidity
File: registries/contracts/utils/SafeTransferLib.sol

33:             mstore(36, to) // Append the "to" argument.

34:             mstore(68, amount) // Append the "amount" argument.

75:             mstore(36, amount) // Append the "amount" argument.

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/utils/SafeTransferLib.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

932:                     uint256 normalizedStakingAmount = stakingIncentive / (10 ** (18 - bridgingDecimals));

933:                     normalizedStakingAmount *= 10 ** (18 - bridgingDecimals);

1221:             uint256 normalizedAmount = amount / (10 ** (18 - bridgingDecimals));

1222:             normalizedAmount *= 10 ** (18 - bridgingDecimals);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

### <a name="NC-15"></a>[NC-15] Variables need not be initialized to zero
The default value for variables is zero, so initializing them to zero is superfluous.

*Instances (39)*:
```solidity
File: governance/contracts/VoteWeighting.sol

227:         for (uint256 i = 0; i < MAX_NUM_WEEKS; i++) {

267:         for (uint256 i = 0; i < MAX_NUM_WEEKS; i++) {

573:         for (uint256 i = 0; i < accounts.length; ++i) {

793:         for (uint256 i = 0; i < accounts.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

332:         for (uint256 i = 0; i < _stakingParams.agentIds.length; ++i) {

380:         for (uint256 i = 0; i < numAgentIds; ++i) {

449:         for (uint256 i = 0; i < totalNumServices; ++i) {

548:             for (uint256 i = 0; i < size; ++i) {

648:                 for (uint256 i = 0; i < numServices; ++i) {

669:             for (uint256 i = 0; i < serviceIds.length; ++i) {

773:             for (uint256 i = 0; i < numAgents; ++i) {

899:         for (uint256 i = 0; i < numServices; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingToken.sol

92:         for (uint256 i = 0; i < serviceAgentIds.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingToken.sol)

```solidity
File: registries/contracts/staking/StakingVerifier.sol

150:         for (uint256 i = 0; i < implementations.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

450:         for (uint256 i = 0; i < chainIds.length; ++i) {

462:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

473:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

485:                 for (uint256 j = 0; j < updatedStakingTargets.length; ++j) {

526:         for (uint256 i = 0; i < chainIds.length; ++i) {

550:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

593:         for (uint256 i = 0; i < chainIds.length; ++i) {

600:             for (uint256 j = 0; j < stakingTargets[i].length; ++j) {

717:         for (uint256 i = 0; i < chainIds.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

904:         for (uint256 i = 0; i < numServices; ++i) {

916:             for (uint256 unitType = 0; unitType < 2; ++unitType) {

930:                     for (uint256 j = 0; j < numServiceUnits; ++j) {

960:                 for (uint256 j = 0; j < numServiceUnits; ++j) {

1010:         for (uint256 i = 0; i < numServices; ++i) {

1199:             for (uint256 i = 0; i < 2; ++i) {

1329:         for (uint256 i = 0; i < 2; ++i) {

1335:         for (uint256 i = 0; i < unitIds.length; ++i) {

1357:         for (uint256 i = 0; i < unitIds.length; ++i) {

1401:         for (uint256 i = 0; i < 2; ++i) {

1407:         for (uint256 i = 0; i < unitIds.length; ++i) {

1429:         for (uint256 i = 0; i < unitIds.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

58:             for (uint256 i = 0; i < numYears; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/TokenomicsConstants.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

150:         uint256 localWithheldAmount = 0;

154:         for (uint256 i = 0; i < targets.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

94:         for (uint256 i = 0; i < targets.length; ++i) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | `approve()`/`safeApprove()` may revert if the current approval is not zero | 7 |
| [L-2](#L-2) | `decimals()` should be of type `uint8` | 1 |
| [L-3](#L-3) | Division by zero not prevented | 11 |
| [L-4](#L-4) | External call recipient may consume all transaction gas | 4 |
| [L-5](#L-5) | Initializers could be front-run | 5 |
| [L-6](#L-6) | Signature use at deadlines should be allowed | 7 |
| [L-7](#L-7) | Possible rounding issue | 4 |
| [L-8](#L-8) | Loss of precision | 14 |
| [L-9](#L-9) | Solidity version 0.8.20+ may not work on other chains due to `PUSH0` | 1 |
| [L-10](#L-10) | Unsafe ERC20 operation(s) | 11 |
| [L-11](#L-11) | Unsafe solidity low-level call can cause gas grief attack | 2 |
| [L-12](#L-12) | Upgradeable contract not initialized | 11 |
| [L-13](#L-13) | A year is not always 365 days | 1 |
### <a name="L-1"></a>[L-1] `approve()`/`safeApprove()` may revert if the current approval is not zero
- Some tokens (like the *very popular* USDT) do not work when changing the allowance from an existing non-zero allowance value (it will revert if the current approval is not zero to protect against front-running changes of approvals). These tokens must first be approved for zero and then the actual allowance can be approved.
- Furthermore, OZ's implementation of safeApprove would throw an error if an approve is attempted from a non-zero value (`"SafeERC20: approve from non-zero to non-zero allowance"`)

Set the allowance to zero immediately before each of the existing allowance calls

*Instances (7)*:
```solidity
File: tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol

174:             IToken(olas).approve(l1ERC20Gateway, transferAmount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

191:                 IToken(olas).approve(target, amount);

290:             IToken(olas).approve(target, amount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

118:             IToken(olas).approve(target, amount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)

```solidity
File: tokenomics/contracts/staking/GnosisDepositProcessorL1.sol

65:             IToken(olas).approve(l1TokenRelayer, transferAmount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/OptimismDepositProcessorL1.sol

111:             IToken(olas).approve(l1TokenRelayer, transferAmount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/PolygonDepositProcessorL1.sol

68:             IToken(olas).approve(predicate, transferAmount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonDepositProcessorL1.sol)

### <a name="L-2"></a>[L-2] `decimals()` should be of type `uint8`

*Instances (1)*:
```solidity
File: tokenomics/contracts/Dispenser.sol

853:         uint256 bridgingDecimals

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

### <a name="L-3"></a>[L-3] Division by zero not prevented
The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.

*Instances (11)*:
```solidity
File: governance/contracts/VoteWeighting.sol

432:             weight = 1e18 * nomineeWeight / totalSum;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingActivityChecker.sol

58:             uint256 ratio = ((curNonces[0] - lastNonces[0]) * 1e18) / ts;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingActivityChecker.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

622:                     updatedReward = (eligibleServiceRewards[i] * lastAvailableRewards) / totalRewards;

633:                 updatedReward = (eligibleServiceRewards[0] * lastAvailableRewards) / totalRewards;

904:                     reward = (eligibleServiceRewards[i] * lastAvailableRewards) / totalRewards;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: tokenomics/contracts/Dispenser.sol

932:                     uint256 normalizedStakingAmount = stakingIncentive / (10 ** (18 - bridgingDecimals));

1221:             uint256 normalizedAmount = amount / (10 ** (18 - bridgingDecimals));

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

473:         uint256 _inflationPerSecond = getInflationForYear(0) / zeroYearSecondsLeft;

872:             totalIncentives = mapUnitIncentives[unitType][unitId].topUp + totalIncentives / sumUnitIncentives;

928:                     uint96 amount = uint96(amounts[i] / numServiceUnits);

1451:                     topUp += totalIncentives / sumUnitIncentives;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

### <a name="L-4"></a>[L-4] External call recipient may consume all transaction gas
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use `addr.call{gas: <amount>}("")` or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

*Instances (4)*:
```solidity
File: registries/contracts/staking/StakingFactory.sol

216:         (bool success, bytes memory returnData) = instance.call(initPayload);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: registries/contracts/staking/StakingNativeToken.sol

33:         (bool success, ) = to.call{value: amount}("");

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingNativeToken.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

161:             (bool success, bytes memory returnData) = stakingFactory.call(verifyData);

396:         (bool success, ) = msg.sender.call{value: amount}("");

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

### <a name="L-5"></a>[L-5] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (5)*:
```solidity
File: registries/contracts/staking/StakingBase.sol

278:     function _initialize(

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingNativeToken.sol

20:     function initialize(StakingParams memory _stakingParams) external {

21:         _initialize(_stakingParams);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingNativeToken.sol)

```solidity
File: registries/contracts/staking/StakingToken.sol

54:     function initialize(

60:         _initialize(_stakingParams);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingToken.sol)

### <a name="L-6"></a>[L-6] Signature use at deadlines should be allowed
According to [EIP-2612](https://github.com/ethereum/EIPs/blob/71dc97318013bf2ac572ab63fab530ac9ef419ca/EIPS/eip-2612.md?plain=1#L58), signatures used on exactly the deadline timestamp are supposed to be allowed. While the signature may or may not be used for the exact EIP-2612 use case (transfer approvals), for consistency's sake, all deadlines should follow this semantic. If the timestamp is an expiration rather than a deadline, consider whether it makes more sense to include the expiration timestamp as a valid timestamp, as is done for deadlines.

*Instances (7)*:
```solidity
File: governance/contracts/VoteWeighting.sol

228:             if (t > block.timestamp) {

243:             if (t > block.timestamp) {

268:             if (t > block.timestamp) {

283:             if (t > block.timestamp) {

503:         if (nextAllowedVotingTime > block.timestamp) {

542:         if (oldSlope.end > block.timestamp) {

658:         if (oldSlope.end > block.timestamp) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

### <a name="L-7"></a>[L-7] Possible rounding issue
Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator. Also, there is indication of multiplication and division without the use of parenthesis which could result in issues.

*Instances (4)*:
```solidity
File: governance/contracts/VoteWeighting.sol

432:             weight = 1e18 * nomineeWeight / totalSum;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

622:                     updatedReward = (eligibleServiceRewards[i] * lastAvailableRewards) / totalRewards;

633:                 updatedReward = (eligibleServiceRewards[0] * lastAvailableRewards) / totalRewards;

904:                     reward = (eligibleServiceRewards[i] * lastAvailableRewards) / totalRewards;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

### <a name="L-8"></a>[L-8] Loss of precision
Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator

*Instances (14)*:
```solidity
File: governance/contracts/VoteWeighting.sol

214:         timeSum = block.timestamp / WEEK * WEEK;

309:         uint256 nextTime = (block.timestamp + WEEK) / WEEK * WEEK;

424:         uint256 t = time / WEEK * WEEK;

432:             weight = 1e18 * nomineeWeight / totalSum;

489:         uint256 nextTime = (block.timestamp + WEEK) / WEEK * WEEK;

515:             slope: uint256(uint128(userSlope)) * weight / MAX_WEIGHT,

605:         uint256 nextTime = (block.timestamp + WEEK) / WEEK * WEEK;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)

```solidity
File: registries/contracts/staking/StakingBase.sol

622:                     updatedReward = (eligibleServiceRewards[i] * lastAvailableRewards) / totalRewards;

633:                 updatedReward = (eligibleServiceRewards[0] * lastAvailableRewards) / totalRewards;

904:                     reward = (eligibleServiceRewards[i] * lastAvailableRewards) / totalRewards;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

1126:         uint256 numYears = (block.timestamp - timeLaunch) / ONE_YEAR;

1136:             curInflationPerSecond = getInflationForYear(numYears) / ONE_YEAR;

1243:         numYears = (block.timestamp + curEpochLen - timeLaunch) / ONE_YEAR;

1252:             curInflationPerSecond = getInflationForYear(numYears) / ONE_YEAR;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

### <a name="L-9"></a>[L-9] Solidity version 0.8.20+ may not work on other chains due to `PUSH0`
The compiler for Solidity 0.8.20 switches the default target EVM version to [Shanghai](https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/#important-note), which includes the new `PUSH0` op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier [EVM](https://docs.soliditylang.org/en/v0.8.20/using-the-compiler.html?ref=zaryabs.com#setting-the-evm-version-to-target) [version](https://book.getfoundry.sh/reference/config/solidity-compiler#evm_version). While the project itself may or may not compile with 0.8.20, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.

*Instances (1)*:
```solidity
File: registries/contracts/utils/SafeTransferLib.sol

2: pragma solidity ^0.8.23;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/utils/SafeTransferLib.sol)

### <a name="L-10"></a>[L-10] Unsafe ERC20 operation(s)

*Instances (11)*:
```solidity
File: tokenomics/contracts/Dispenser.sol

420:             IToken(olas).transfer(depositProcessor, transferAmount);

456:                 IToken(olas).transfer(depositProcessor, transferAmounts[i]);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)

```solidity
File: tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol

174:             IToken(olas).approve(l1ERC20Gateway, transferAmount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

191:                 IToken(olas).approve(target, amount);

290:             IToken(olas).approve(target, amount);

443:             bool success = IToken(olas).transfer(newL2TargetDispenser, amount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

```solidity
File: tokenomics/contracts/staking/EthereumDepositProcessor.sol

112:                 IToken(olas).transfer(timelock, refundAmount);

118:             IToken(olas).approve(target, amount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)

```solidity
File: tokenomics/contracts/staking/GnosisDepositProcessorL1.sol

65:             IToken(olas).approve(l1TokenRelayer, transferAmount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/OptimismDepositProcessorL1.sol

111:             IToken(olas).approve(l1TokenRelayer, transferAmount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol)

```solidity
File: tokenomics/contracts/staking/PolygonDepositProcessorL1.sol

68:             IToken(olas).approve(predicate, transferAmount);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonDepositProcessorL1.sol)

### <a name="L-11"></a>[L-11] Unsafe solidity low-level call can cause gas grief attack
Using the low-level calls of a solidity address can leave the contract open to gas grief attacks. These attacks occur when the called contract returns a large amount of data.

So when calling an external contract, it is necessary to check the length of the return data before reading/copying it (using `returndatasize()`).

*Instances (2)*:
```solidity
File: registries/contracts/staking/StakingFactory.sol

216:         (bool success, bytes memory returnData) = instance.call(initPayload);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)

```solidity
File: tokenomics/contracts/staking/DefaultTargetDispenserL2.sol

161:             (bool success, bytes memory returnData) = stakingFactory.call(verifyData);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)

### <a name="L-12"></a>[L-12] Upgradeable contract not initialized
Upgradeable contracts are initialized via an initializer function rather than by a constructor. Leaving such a contract uninitialized may lead to it being taken over by a malicious user

*Instances (11)*:
```solidity
File: registries/contracts/staking/StakingBase.sol

116: error AlreadyInitialized();

278:     function _initialize(

283:             revert AlreadyInitialized();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)

```solidity
File: registries/contracts/staking/StakingNativeToken.sol

20:     function initialize(StakingParams memory _stakingParams) external {

21:         _initialize(_stakingParams);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingNativeToken.sol)

```solidity
File: registries/contracts/staking/StakingToken.sol

54:     function initialize(

60:         _initialize(_stakingParams);

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingToken.sol)

```solidity
File: tokenomics/contracts/Tokenomics.sol

113: error AlreadyInitialized();

409:     function initializeTokenomics(

423:             revert AlreadyInitialized();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)

```solidity
File: tokenomics/contracts/interfaces/IErrorsTokenomics.sol

119:     error AlreadyInitialized();

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/interfaces/IErrorsTokenomics.sol)

### <a name="L-13"></a>[L-13] A year is not always 365 days
On leap years, the number of days is 366, so calculations during those years will return the wrong value

*Instances (1)*:
```solidity
File: tokenomics/contracts/TokenomicsConstants.sol

15:     uint256 public constant ONE_YEAR = 1 days * 365;

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/TokenomicsConstants.sol)


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | `block.number` means different things on different L2s | 2 |
### <a name="M-1"></a>[M-1] `block.number` means different things on different L2s
On Optimism, `block.number` is the L2 block number, but on Arbitrum, it's the L1 block number, and `ArbSys(address(100)).arbBlockNumber()` must be used. Furthermore, L2 block numbers often occur much more frequently than L1 block numbers (any may even occur on a per-transaction basis), so using block numbers for timing results in inconsistencies, especially when voting is involved across multiple chains. As of version 4.9, OpenZeppelin has [modified](https://blog.openzeppelin.com/introducing-openzeppelin-contracts-v4.9#governor) their governor code to use a clock rather than block numbers, to avoid these sorts of issues, but this still requires that the project [implement](https://docs.openzeppelin.com/contracts/4.x/governance#token_2) a [clock](https://eips.ethereum.org/EIPS/eip-6372) for each L2.

*Instances (2)*:
```solidity
File: tokenomics/contracts/Tokenomics.sol

1026:         lastDonationBlockNumber = uint32(block.number);

1092:         if (lastDonationBlockNumber == block.number) {

```
[Link to code](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)
