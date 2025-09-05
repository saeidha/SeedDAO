# SeedDAO: Decentralized Autonomous Organization (DAO) with Token Governance

## Overview

This project provides a comprehensive and secure framework for a Decentralized Autonomous Organization (DAO) built on the Ethereum blockchain. The DAO empowers its members, who are holders of a specific ERC-20 governance token, to collectively manage a treasury and make decisions through a formal on-chain proposal and voting system. The goal is to replace traditional, hierarchical organizational structures with a transparent, community-owned, and censorship-resistant model of governance.

The core of this system is the DAO.sol smart contract, which acts as the governance layer and treasury. It allows members to create proposals, vote on them using their token-weighted influence, and execute passed proposals, which can trigger transactions and interactions with other smart contracts. By encoding the rules of the organization into a smart contract, it ensures that all actions are executed exactly as approved by the voters, creating a trustless environment. This implementation includes critical governance features like voting delays, periods, quorums, and a timelock for executing proposals, ensuring a robust and decentralized decision-making process suitable for developers and communities aiming to build truly decentralized applications.

## Features

- **Token-Based Governance**: Voting power is directly proportional to the amount of governance tokens a member holds at a specific point in the past (snapshot-based). This is crucial for preventing governance attacks, such as using flash loans to acquire a large number of tokens to sway a vote, as the voting power is based on a historical balance.

- **Full Proposal Lifecycle**: Manages proposals through a clearly defined lifecycle: Pending, Active, Succeeded, Defeated, Queued, Executed, and Canceled. The state of any proposal can be queried at any time, providing full transparency into the decision-making process.

- **On-Chain Treasury**: The DAO contract can securely hold and manage a treasury of ETH and various ERC-20 tokens. These funds can be allocated for community grants, developer bounties, operational expenses, or strategic investments, all through successful proposals. This is more secure than a multisig wallet as it removes the single point of failure associated with key holders.

- **Configurable Governance Parameters**: The DAO owner (typically the deployer, who may later renounce ownership to the DAO itself) can set key parameters like the `votingPeriod`, `proposalThreshold`, and `quorumPercentage`. These parameters allow the governance model to be tailored to the specific needs and size of the community.

- **Quorum Requirement**: A proposal must receive a minimum percentage of the total token supply's votes (in 'For' or 'Against' votes) to be considered valid. This ensures that decisions have sufficient community participation and prevents a small minority from passing proposals during periods of low voter turnout. For example, with a 10% quorum, at least 10% of all tokens must participate in the vote for it to be valid.

- **Execution Timelock**: A mandatory delay (`executionDelay`) between when a proposal passes and when it can be executed. This feature acts as a crucial security measure, giving members a window to react to decisions they strongly disagree with, potentially by selling their tokens (a concept known as "rage quitting") before the action is implemented.

- **Double-Vote Prevention**: The contract ensures that each member can only cast one vote per proposal. This is enforced by storing a receipt for each vote, guaranteeing the integrity of the final vote count.

- **Gas-Efficient Voting**: The system is designed to be as gas-conscious as possible. While all votes are on-chain transactions, the logic is optimized to minimize computational complexity and cost for the voters.

- **Comprehensive View Functions**: A wide array of public view functions allows users and decentralized application frontends to easily query the state of the DAO, inspect the details of specific proposals, check a member's voting power at a historical block, and view treasury balances.

## Core Concepts

### 1. The Governance Token

An external ERC-20 token that supports vote snapshots (compliant with the ERC20Votes standard) is used to represent membership and voting rights. This standard is critical because it maintains a historical record of each user's balance at every block. When a vote begins, the DAO contract queries this historical data to determine a user's voting power, locking it in for the duration of the vote. This snapshot mechanism is a cornerstone of secure on-chain governance. The health and decentralization of the DAO are also directly tied to the distribution of this token; a wider, more equitable distribution leads to a more robust and community-driven organization.

### 2. The Proposal Lifecycle

A proposal is an on-chain action that the DAO can take, such as sending funds from the treasury or calling a function on another contract.

- **Creation**: A member holding a minimum number of governance tokens (the `proposalThreshold`) can submit a new proposal. The proposal includes the target contract address(es), the value(s) of ETH to send, the function calldata (the encoded function signature and arguments), and a human-readable description. For example, to transfer 100 USDC tokens, the target would be the USDC contract address, the value would be 0, and the calldata would be the encoded `transfer(address, uint256)` function call.

- **Pending**: After creation, a proposal enters a `votingDelay` period (measured in blocks). This is a brief window before voting begins, allowing members to discuss the proposal off-chain (e.g., on forums) and for dApp frontends to index and display the new proposal.

- **Active (Voting)**: Once the delay passes, the `votingPeriod` (measured in blocks) starts. During this time, token holders can cast their votes: For, Against, or Abstain. An 'Abstain' vote contributes to meeting the quorum but does not count towards the 'For' or 'Against' tally, allowing members to signal participation without taking a side.

- **State Check**: After the voting period ends, the proposal's state is determined by the `state()` function.
  - **Defeated**: If the quorum is not met or 'Against' votes are greater than or equal to 'For' votes.
  - **Succeeded**: If the quorum is met and 'For' votes are strictly greater than 'Against' votes.

- **Queued (Timelock)**: A succeeded proposal must be manually moved to the Queued state by anyone calling the `queue()` function. This action starts the `executionDelay` timelock (measured in seconds), which is the final safeguard before execution.

- **Execution**: After the timelock delay has passed, anyone can call the `execute()` function to enact the proposal's on-chain actions. This is a powerful feature, as it means the DAO does not rely on the proposer or any single entity to finalize its decisions.

- **Cancellation**: A proposal can be canceled by the proposer at almost any time before it is executed. This is useful if a proposal is found to be flawed or is no longer relevant after it has been created.

### 3. Treasury Management

The DAO contract itself acts as a decentralized treasury. It can receive ETH (via a `receive()` function) and any ERC-20 tokens. All funds are locked and can only be moved or spent by a successful governance proposal that has passed the full lifecycle. This programmatic control over the treasury is a significant security enhancement over traditional systems, as it removes human error and the need to trust individuals with private keys. The treasury can be diversified to hold various assets, spreading risk and enabling a wider range of activities as dictated by the DAO's members.

## Usage

1. **Deployment**: First, deploy your ERC20Votes governance token contract. Then, deploy the DAO.sol contract, providing the address of the deployed token and setting the initial governance parameters (voting delay/period, quorum, etc.) in the constructor.

2. **Fund Treasury**: Send ETH directly to the deployed DAO contract address via a standard transaction. To send ERC-20 tokens, use the respective token's transfer function, with the DAO contract address as the recipient.

3. **Create Proposal**: A qualified token holder (with tokens >= `proposalThreshold`) calls the `propose()` function. This requires carefully formulating the transaction details, especially the calldata, which can be done using tools like Ethers.js or an Etherscan calldata encoder.

4. **Vote**: During the active voting period, token holders connect their wallets to a dApp and call the `castVote()` function with their choice. This is an on-chain transaction and will require a gas fee.

5. **Queue & Execute Proposal**: Once a proposal has succeeded, it requires two more transactions to be finalized. First, anyone can call `queue()` to begin the timelock. Second, after the timelock duration has passed, anyone can call `execute()` to run the proposal's transaction and enact the DAO's decision.

## Development & Testing

This project is designed to be developed and tested with modern Ethereum development frameworks like Foundry or Hardhat. These tools provide the necessary infrastructure for compiling, deploying, and rigorously testing smart contracts. The included test suite, TestDAO.sol, provides comprehensive coverage of all functions and proposal lifecycle states, serving as a critical component for ensuring the contract's security and correctness.