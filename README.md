# Olas audit details
- Total Prize Pool: $90,500 in USDC
  - HM awards: $72,000 in USDC
  - QA awards: $3,000 in USDC
  - Judge awards: $9,000 in USDC
  - Validator awards: $6,000 in USDC
  - Scout awards: $500 in USDC
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2024-05-olas/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts May 28, 2024 20:00 UTC
- Ends June 18, 2024 20:00 UTC

## Automated Findings / Publicly Known Issues

The 4naly3er report can be found [here](https://github.com/code-423n4/2024-05-olas/blob/main/4naly3er-report.md).

_Note for C4 wardens: Anything included in this `Automated Findings / Publicly Known Issues` section is considered a publicly known issue and is ineligible for awards._

The known issues (some of them intended by design) that are not in scope for this audit are outlined in the following:
- https://github.com/code-423n4/2024-05-olas/blob/main/governance/docs/Vulnerabilities_list_governance.pdf
- https://github.com/code-423n4/2024-05-olas/blob/main/registries/docs/Vulnerabilities_list_registries.pdf
- https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/docs/Vulnerabilities_list_tokenomics.pdf


Additionally, the following are not in scope for this audit.

- All vulnerabilities mentioned in [governance audits folder](https://github.com/code-423n4/2024-05-olas/blob/main/governance/audits), [registries audits folder](https://github.com/code-423n4/2024-05-olas/blob/main/registries/audits), [tokenomics audits folder](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/audits)
- All vulnerabilities mentioned in [governance docs folder](https://github.com/code-423n4/2024-05-olas/blob/main/governance/docs), [registries docs folder](https://github.com/code-423n4/2024-05-olas/blob/main/registries/docs), [tokenomics docs folder](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/docs)
- All vulnerabilities mentioned in [governance test folder](https://github.com/code-423n4/2024-05-olas/blob/main/governance/test), [registries test folder](https://github.com/code-423n4/2024-05-olas/blob/main/registries/test), [tokenomics test folder](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/test)
- All vulnerabilities mentioned in the comments on the contracts code [governance contracts folder](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts), [registries contracts folder](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts), [tokenomics contracts folder](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts)
- All vulnerabilities found in the inherited source code from [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) and [Solmate](https://github.com/transmissions11/solmate)
- All vulnerabilities found in the inherited source code for the bridging contracts.
- All vulnerabilities found in code based on or inspired by [Maple Finance](https://github.com/maple-labs), [Curve DAO](https://github.com/curvefi/curve-dao-contracts), [Uniswap Lab](https://github.com/Uniswap/v2-core), [PaulRBerg](https://github.com/paulrberg/prb-math), [Jeiwan](https://github.com/Jeiwan/zuniswapv2), [Safe Ecosystem](https://github.com/safe-global/safe-contracts) and that are already reported to one of those projects.


# Overview

This audit is focused on smart contracts related to Olas staking mechanism. Olas on-chain protocol can be divided in three main parts: governance, registries, and tokenomics. Here is an overview of these parts. 

The **governance** is designed to assume various control points to steer the Olas protocol. The governance token, veOLAS is the virtualized representation of OLAS locked and used a similar approach to [veCRV](https://curve.readthedocs.io/dao-vecrv.html), where votes are weighted depending on the time OLAS is locked other than the amount of locked OLAS.

The VoteWeighting contract enables Olas DAO members (via veOLAS) to vote on staking programs, assigning weights according to their preferences. It adopts a model similar to the [Curve Gauge Controller](https://curve.readthedocs.io/dao-gauges.html#dao-gauges-controller), maintains a list of gauges and their associated weights.  Modifications from the original Curve Gauge Controller include granting anyone the ability to add staking contracts by removing ownership control on this functionality, and eliminating additional categorization by contract type. For more details on VotingWeight, see [Olas staking smart contracts](https://github.com/code-423n4/2024-05-olas/blob/main/governance/docs/StakingSmartContracts.pdf) and [Olas staking whitepaper](https://staking.olas.network/poaa-whitepaper.pdf).

The list of the governance contracts in scope for the current audit can be found in the Scope section below.

**Registries** allow developer of code in form of agents, components, or services to register and manage their code on-chain. The code existing off-chain will be uniquely represented on-chain by means of NFTs. A summary of registries is provided [here](https://github.com/code-423n4/2024-05-olas/blob/main/registries/docs/AgentServicesFunctionality.pdf). The registry contracts related to [Olas staking](https://staking.olas.network/poaa-whitepaper.pdf) collectively form a robust system for managing staking services within an Olas ecosystem. The StakingFactory serves as a gateway for deploying staking contracts, offering flexibility through customizable verification logic. Staking contracts, designed for compatibility, provide standardized functionalities while allowing variation in activity checks tailored to specific use cases. The StakingActivityCheck contract optimistically ensures that stakers meet predefined activity criteria, contributing to the fair distribution of rewards (see [Olas staking smart contracts](https://github.com/code-423n4/2024-05-olas/blob/main/governance/docs/StakingSmartContracts.pdf) for more details).

The list of the registries contracts in scope for the current audit can be found in the Scope section below.

The **tokenomics** provides the following pimitives: Staking, developer rewards, and bonding (cf. [Olas tokenomics paper](https://www.autonolas.network/documents/whitepaper/Autonolas_Tokenomics_Core_Technical_Document.pdf) and [Olas staking whitepaper](https://staking.olas.network/poaa-whitepaper.pdf) for more details). 

Tokenomics contracts related to [Olas staking](https://staking.olas.network/poaa-whitepaper.pdf) introduce a robust and decentralized framework for allocating OLAS emissions to staking programs beyond the boundaries of Ethereum to
various networks such as Gnosis, Polygon, Arbitrum, Solana, and more. More details can be found in [Olas staking smart contracts](https://github.com/code-423n4/2024-05-olas/blob/main/governance/docs/StakingSmartContracts.pdf).

## Links

- **Previous audits:**  
  - https://github.com/code-423n4/2024-05-olas/blob/main/governance/audits  
  - https://github.com/code-423n4/2024-05-olas/blob/main/registries/audits
  - https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/audits
- **Documentation:**
  - [Olas staking whitepaper](https://staking.olas.network/poaa-whitepaper.pdf)
  - [Olas staking smart contracts](https://github.com/code-423n4/2024-05-olas/blob/main/governance/docs/StakingSmartContracts.pdf).
  - [Autonolas whitepaper](https://www.autonolas.network/documents/whitepaper/Whitepaper%20v1.0.pdf) 
  The following are relevant for governance related contracts: 
  - [Summary of governance model](https://github.com/code-423n4/2024-05-olas/blob/main/governance/docs/Governance_process.pdf) 
  - [Cross-chain governance design](https://github.com/code-423n4/2024-05-olas/blob/main/governance/docs/governace_bridge.pdf)
  The following are relevant for registries related contracts: 
  - [Summary of registries design](https://github.com/code-423n4/2024-05-olas/blob/main/registries/docs/AgentServicesFunctionality.pdf) 
  - [Definitions and data structures](https://github.com/code-423n4/2024-05-olas/blob/main/registries/docs/definitions.md) 
  The following are relevant for tokenomics related contract:
  - [Summary of tokenomics model](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/docs/Autonolas_tokenomics_audit.pdf) 
  - [Autonolas tokenomics paper](https://www.autonolas.network/documents/whitepaper/Autonolas_Tokenomics_Core_Technical_Document.pdf)
- **Website:** https://olas.network/  
- **Twitter:**  [@autonolas](https://x.com/autonolas)  
- **Discord:**  https://discord.gg/Dh6UqUuV 

---


# Scope

## Files in scope

| Contract                                                                                                                                                                        | SLOC        | Purpose                                                                                                                                  | Libraries used                                                                                          |
|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Governance contracts (1)                                                                                                                                                        |             |                                                                                                                                          |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [governance/contracts/VoteWeighting.sol](https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol)                                           | 427         | The contract enables Olas DAO members (via veOLAS) to vote on staking programs, assigning weights according to their preferences.        |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| Registries contracts (8)                                                                                                                                                        |             |                                                                                                                                          |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [registries/contracts/staking/StakingBase.sol](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingBase.sol)                               | 518         | Base abstract smart contract for staking a service by its owner                                                                          | [`solmate/*`](https://github.com/transmissions11/solmate)                                               |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [registries/contracts/staking/StakingFactory.sol](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingFactory.sol)                         | 159         | Smart contract for staking factory                                                                                                       |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [registries/contracts/staking/StakingNativeToken.sol](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingNativeToken.sol)                 | 22          | Smart contract for staking a service secured with the native network token                                                               |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [registries/contracts/staking/StakingProxy.sol](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingProxy.sol)                             | 30          | Smart contract for staking proxy                                                                                                         |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [registries/contracts/staking/StakingToken.sol](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingToken.sol)                             | 58          | Smart contract for staking a service secured whit an ERC20 token                                                                         |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [registries/contracts/staking/StakingVerifier.sol](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingVerifier.sol)                       | 133         | Smart contract for service staking contracts verification                                                                                |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [registries/contracts/staking/StakingActivityChecker.sol](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/staking/StakingActivityChecker.sol)         | 28          | Smart contract for performing a service staking activity check                                                                           |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [registries/contracts/utils/SafeTransferLib.sol](https://github.com/code-423n4/2024-05-olas/blob/main/registries/contracts/utils/SafeTransferLib.sol)                           | 41          | This contract provides a library with safe methods for transferring ERC-20 tokens                                                        |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| Tokenomics contracts (16)                                                                                                                                                       |             |                                                                                                                                          |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [tokenomics/contracts/staking/DefaultDepositProcessorL1.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol)   | 112         | Smart contract for sending tokens and data via arbitrary bridge from L1 to L2 and processing data received from L2                       |                                                                                                         |
| -----------                                                                                                                                                                     |             |                                                                                                                                          |                                                                                                         |
| [tokenomics/contracts/staking/DefaultTargetDispenserL2.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol)     | 253         | Smart contract for processing tokens and data received on L2, and data sent back to L1                                                   |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [tokenomics/contracts/staking/EthereumDepositProcessor.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/EthereumDepositProcessor.sol)     | 87          | Smart contract for processing tokens and data on L1                                                                                      |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol) | 98          | Smart contract for sending tokens and data via Arbitrum bridge from L1 to L2 and processing data received from L2                        |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | ------------------------------------------------------------------------------------------------------------------                       | ------------------------------------------------------------------------------------------------------- |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| [tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol)   | 33          | Smart contract for processing tokens and data received on Arbitrum L2, and data sent back to L1                                          |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [tokenomics/contracts/staking/GnosisDepositProcessorL1.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisDepositProcessorL1.sol)     | 48          | Smart contract for sending tokens and data via Gnosis bridge from L1 to L2 and processing data received from L2.                         |                                                                                                         |
| -----------                                                                                                                                                                     |             |                                                                                                                                          |                                                                                                         |
| [tokenomics/contracts/staking/GnosisTargetDispenserL2.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/GnosisTargetDispenserL2.sol)       | 50          | Smart contract for processing tokens and data received on Gnosis L2, and data sent back to L1                                            |                                                                                                         |
| -----------                                                                                                                                                                     |             |                                                                                                                                          |                                                                                                         |
| [tokenomics/contracts/staking/OptimismDepositProcessorL1.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol) | 69          | Smart contract for sending tokens and data via Optimism bridge from L1 to L2 and processing data received from L2.                       |                                                                                                         |
| -----------                                                                                                                                                                     |             |                                                                                                                                          |                                                                                                         |
| [tokenomics/contracts/staking/OptimismTargetDispenserL2.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/OptimismTargetDispenserL2.sol)   | 45          | Smart contract for processing tokens and data received on Optimism L2, and data sent back to L1                                          |                                                                                                         |
| -----------                                                                                                                                                                     |             |                                                                                                                                          |                                                                                                         |
| [tokenomics/contracts/staking/PolygonDepositProcessorL1.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonDepositProcessorL1.sol)   | 58          | Smart contract for sending tokens and data via Polygon bridge from L1 to L2 and processing data received from L2                         | [`fx-portal/*`](https://github.com/0xPolygon/fx-portal/tree/296ac8d41579f98d3a4dfb6d41737fae272a30ba)   |
| -----------                                                                                                                                                                     |             |                                                                                                                                          |                                                                                                         |
| [tokenomics/contracts/staking/PolygonTargetDispenserL2.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/PolygonTargetDispenserL2.sol)     | 34          | Smart contract for processing tokens and data received on Polygon L2, and data sent back to L1                                           | [`fx-portal/*`](https://github.com/0xPolygon/fx-portal/tree/296ac8d41579f98d3a4dfb6d41737fae272a30ba)   |
| -----------                                                                                                                                                                     |             |                                                                                                                                          |                                                                                                         |
| [tokenomics/contracts/staking/WormholeDepositProcessorL1.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeDepositProcessorL1.sol) | 78          | Smart contract for sending tokens and data via Wormhole bridge from L1 to L2 and processing data received from L2                        | [`@wormhole-solidity-sdk/*`](https://github.com/wormhole-foundation/wormhole-solidity-sdk)              |
| -----------                                                                                                                                                                     |             |                                                                                                                                          |                                                                                                         |
| [tokenomics/contracts/staking/WormholeTargetDispenserL2.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/WormholeTargetDispenserL2.sol)   | 88          | Smart contract for processing tokens and data received via Wormhole on L2, and data sent back to L1                                      | [`@wormhole-solidity-sdk/*`](https://github.com/wormhole-foundation/wormhole-solidity-sdk)              |
| -----------                                                                                                                                                                     |             |                                                                                                                                          |                                                                                                         |
| [tokenomics/contracts/Tokenomics.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol)                                                 | 733         | Smart contract implementing the tokenomics model for code incentives, discount factor bonding mechanism regulations, and staking points. | [`@prb-math/*`]](https://github.com/PaulRBerg/prb-math)                                                 |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [tokenomics/contracts/TokenomicsConstants.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/TokenomicsConstants.sol)                               | 60          | Smart contract with tokenomics constants for annual inflation supplies                                                                   |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [tokenomics/contracts/Dispenser.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol)                                                   | 644         | Smart contract for distributing dev rewards and claim staking emissions                                                                  |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| Tokenomics interfaces (3)                                                                                                                                                       |             |                                                                                                                                          |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [tokenomics/contracts/interfaces/IDonatorBlacklist.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/interfaces/IDonatorBlacklist.sol])            | 4           | DonatorBlacklist interface                                                                                                               |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [tokenomics/contracts/interfaces/IErrorsTokenomics.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/interfaces/IErrorsTokenomics.sol)             | 31          | Errors interface                                                                                                                         |                                                                                                         |
| -----------                                                                                                                                                                     | ----------- | -----------                                                                                                                              | -----------                                                                                             |
| [tokenomics/contracts/interfaces/IBridgeErrors.sol](https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/interfaces/IBridgeErrors.sol)                     | 23          | Bridge Errors interface                                                                                                                  |                                                                                                         |
| -----------                                                                                                                                                                     |             |                                                                                                                                          |                                                                                                         |
| TOTAL                                                                                                                                                                   |  3964           |                                                                                                                                          |                                                                                                         

## Scoping Q &amp; A

### General questions


| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| ERC20 used by the protocol              |       Any (all possible ERC20s)             |
| Test coverage                           | Governance Functions: 99.43% - Registries Functions: 99.46% - Tokenomics Functions: 99.32%                            |
| ERC721 used  by the protocol            |            Some contracts in scope interacts with Solmate ERC721, however such ERC721 contract is not scope.             |
| ERC777 used by the protocol             |           None                |
| ERC1155 used by the protocol            |              None            |
| Chains the protocol will be deployed on | Ethereum, Arbitrum, Base, Optimism, Polygon, OtherGnosis, Celo, Solana

  

### ERC20 token behaviors in scope

| Question                                                                                                                                                   | Answer |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| [Missing return values](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#missing-return-values)                                                      |   Yes  |
| [Fee on transfer](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#fee-on-transfer)                                                                  |  Yes  |
| [Balance changes outside of transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#balance-modifications-outside-of-transfers-rebasingairdrops) | Yes    |
| [Upgradeability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#upgradable-tokens)                                                                 |   Yes  |
| [Flash minting](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#flash-mintable-tokens)                                                              | Yes    |
| [Pausability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#pausable-tokens)                                                                      | Yes    |
| [Approval race protections](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#approval-race-protections)                                              | Yes    |
| [Revert on approval to zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-approval-to-zero-address)                            | Yes    |
| [Revert on zero value approvals](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-approvals)                                    | Yes    |
| [Revert on zero value transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                    | Yes    |
| [Revert on transfer to the zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-transfer-to-the-zero-address)                    | Yes    |
| [Revert on large approvals and/or transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-large-approvals--transfers)                  | Yes    |
| [Doesn't revert on failure](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure)                                                   |  Yes   |
| [Multiple token addresses](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                          | Yes    |
| [Low decimals ( < 6)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#low-decimals)                                                                 |   Yes  |
| [High decimals ( > 18)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#high-decimals)                                                              | Yes    |
| [Blocklists](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#tokens-with-blocklists)                                                                | Yes    |

### External integrations (e.g., Uniswap) behavior in scope:


| Question                                                  | Answer |
| --------------------------------------------------------- | ------ |
| Enabling/disabling fees (e.g. Blur disables/enables fees) | No   |
| Pausability (e.g. Uniswap pool gets paused)               |  No   |
| Upgradeability (e.g. Uniswap gets upgraded)               |   No  |


### EIP compliance checklist
None


# Additional context

## Main invariants

Here some examples.

1. Only DAO members can cast their vote on votingWeigh 
2. Only executed DAO vote allow to sync information form L1 to L2 and vice-versa
3. In the contract in scope, only dispenser contracts has the manager rights to mint OLAS via Treasury
4. OLAS token transfer can happen only from L1 to L2 



## Attack ideas (where to focus for bugs)
Here some examples.
1. Issues arising from cross-chain interactions
2. Issues arising from incorrect tokenomics calculation
3. Minting that exceeds the global inflation curve
4. Attack to distribute more rewards than expected
5. Issues arising from ability to abuse staking factory contracts



## All trusted roles in the protocol

- DAO members decisions are always assumed to be honest
- DAO executed vote are always assumed to be honest

✅ SCOUTS: Please format the response above 👆 using the template below👇

| Role                                | Description                       |
| --------------------------------------- | ---------------------------- |
| Owner                          | Has superpowers                |
| Administrator                             | Can change fees                       |

## Describe any novel or unique curve logic or mathematical models implemented in the contracts:

Olas staking is a novel staking mechanism that can spawn desirable autonomous AI agent economies in crypto and beyond. The smart contracts in scope of this audit are essential to enable Olas staking. Details on Olas staking can be found in https://staking.olas.network/poaa-whitepaper.pdf. 

A brief overview of the tokenomics model can be found here https://github.com/valory-xyz/autonolas-tokenomics/blob/pre-c4a/docs/Autonolas_tokenomics_audit.pdf. For more details, see the tokenomics paper  https://www.autonolas.network/documents/whitepaper/Autonolas_Tokenomics_Core_Technical_Document.pdf. Details on

An overview of the governance process can be found here https://github.com/valory-xyz/autonolas-governance/blob/pre-c4a/docs/Governance_process.pdf. 

A brief overview of registries can be found here https://github.com/valory-xyz/autonolas-registries/blob/pre-c4a/docs/AgentServicesFunctionality.pdf . Here, the protocol withepaper https://www.autonolas.network/documents/whitepaper/Whitepaper%20v1.0.pdf.  



## Running tests

## 
This repository will follows the standard [`Hardhat`](https://hardhat.org/tutorial/) development process.
- The code is written on Solidity starting from version `0.8.18`.
- The standard versions of Node.js along with Yarn are required to proceed further (confirmed to work with Yarn `1.22.19` and npm `10.1.0` and node `v18.6.0`);
- [`Foundry`](https://book.getfoundry.sh/) is required to run the foundry tests.

### Install the dependencies
The project has submodules to get the dependencies. Make sure you run `git clone --recursive` or init the submodules yourself.
```
git clone https://github.com/code-423n4/2024-05-olas
git submodule update --init --recursive
```
The dependency list is managed by the `package.json` file, and the setup parameters are stored in the `hardhat.config.js` file.
Simply run the following command to install the project:
```
yarn install
```

### Core components
The contracts, deploy scripts, regular scripts and tests are located in the following folders respectively:
```
contracts
scripts
test
```

### Compile the code and run
Compile the code:
```
npm run compile
```
Run tests with Hardhat:
```
npx hardhat test
```
Run tests with Foundry:
```
forge test --hh -vv
```
> [!NOTE]
> Forge tests don't run by default, 

 For them to run in the corresponding folder (registries / tokenomics) one needs to go in the folder and initialize an empty  git repo with the `git init` command, i.e.
```sh
cd registries
git init
forge test --hh -vvv
```
Run tests coverage
```
npx hardhat coverage
```
![Screenshot from 2024-05-27 23-52-57](https://github.com/code-423n4/2024-05-olas/assets/65364747/def2d356-7503-4a73-8b6f-50e1578bac2c)
![Screenshot from 2024-05-28 00-05-33](https://github.com/code-423n4/2024-05-olas/assets/65364747/0c703406-1a61-48c7-be3f-f903bb3fd05f)
![Screenshot from 2024-05-28 00-15-47](https://github.com/code-423n4/2024-05-olas/assets/65364747/9e2a9928-7b95-4fba-973b-588fab75076a)


## Miscellaneous
Employees of OLAS and employees' family members are ineligible to participate in this audit.
