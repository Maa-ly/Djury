# NFTField - Dispute Resolution Market with Pari-Mutuel Betting

## Overview

NFTField is a decentralized dispute resolution platform that combines community-driven dispute resolution with a pari-mutuel betting system. Users can create disputes, submit evidence, vote on outcomes, and bet on dispute results using KWN stablecoin. The system mints NFTs for resolved disputes and allows winners to accrue value through betting pools.

## Architecture

### Core Contracts

1. **DisputeMarket** - Main dispute resolution contract (upgradeable)
2. **PariMutuelBetting** - Pari-mutuel betting system for disputes
3. **DisputeStorage** - Storage layout and data structures
4. **DisputeEvents** - Event definitions
5. **DisputeErrors** - Custom error definitions

### Key Features

- **Dispute Resolution**: Community voting on disputes with evidence submission
- **NFT Minting**: Winners receive NFTs representing resolved disputes
- **Pari-Mutuel Betting**: Users bet on dispute outcomes using KWN
- **Value Accrual**: Winning NFTs accrue KWN from betting pools
- **Multi-Outcome Support**: Creator wins, Respondent wins, or Tie

## How It Works

### 1. Dispute Creation & Resolution

```
User creates dispute → Evidence submission → Voting period → Resolution → NFT minting
```

**Dispute Lifecycle:**
- **Pending**: Dispute created, evidence being collected
- **Active**: Dispute is open for evidence and voting
- **Voting**: Community votes on outcome
- **Resolved**: Winner determined, NFT minted

**Voting System:**
- Only users who submitted evidence can vote
- Votes determine winner (Creator vs Respondent)
- Ties result in NFT minted to contract (admin can transfer)

### 2. Pari-Mutuel Betting System

**Three Betting Outcomes:**
- **Outcome 0**: Creator wins
- **Outcome 1**: Respondent wins  
- **Outcome 2**: Tie

**Betting Process:**
1. Admin creates betting pool for a dispute
2. Users bet KWN on their preferred outcome
3. Betting closes when dispute ends
4. After resolution, winners split the pool minus fees

**Fee Structure:**
- **Protocol Fee**: 1% of total pool (configurable)
- **NFT Rake**: 3% of total pool credited to winner's NFT
- **Tie Slash**: 4% of refunds when pools are voided

**Payout Calculation:**
```
Winner Payout = (User Stake / Winning Side Total) × Payout Pool
Payout Pool = Total Staked - NFT Rake - Protocol Fee
```

### 3. NFT Value Accrual

- Winning NFTs automatically accrue KWN from betting pools
- NFT owners can withdraw accrued KWN
- NFT value increases with each dispute win
- Ties result in voided pools with partial refunds

## Smart Contract Details

### DisputeMarket

**Key Functions:**
- `createDispute()` - Create new dispute with evidence
- `submitEvidence()` - Add evidence to dispute
- `startVoting()` - Begin voting period
- `castVote()` - Vote on dispute outcome
- `resolveDispute()` - Finalize dispute and mint NFT

**Access Control:**
- `ADMIN_ROLE` - Full administrative access
- `MODERATOR_ROLE` - Dispute moderation
- `RESOLVER_ROLE` - Dispute resolution

### PariMutuelBetting

**Key Functions:**
- `createPool()` - Create betting pool for dispute
- `bet()` - Place bet on outcome
- `resolve()` - Resolve betting pool after dispute
- `claim()` - Claim winnings or refunds
- `withdrawNftAccrued()` - Withdraw KWN from NFT

**Pool States:**
- **Open**: Accepting bets
- **Resolved**: Winners determined, payouts available
- **Voided**: Pool cancelled, refunds with slash

## Deployment

### Environment Variables

```bash
PRIVATE_KEY=your_private_key
KWN=0x... # KWN stablecoin contract address
TREASURY=0x... # Treasury address for fees
TIE_SLASH_BPS=400 # Optional: tie slash percentage (default 4%)
```

### Deploy Commands

```bash
# Deploy to testnet
forge script script/Deploy.s.sol --rpc-url kaiatestnet --broadcast

# Deploy to mainnet
forge script script/Deploy.s.sol --rpc-url basechain --broadcast
```

### Contract Addresses

After deployment, the script outputs:
- DisputeMarket address
- PariMutuelBetting address

## Usage Examples

### Creating a Dispute

```solidity
// Create dispute with evidence
disputeMarket.createDispute({
    respondent: 0x...,
    title: "Contract Breach",
    description: "Service provider failed to deliver",
    category: DisputeCategory.Business,
    priority: Priority.High,
    requiresEscrow: true,
    escrowAmount: 0.1 ether,
    customPeriod: 0, // Use default
    evidenceDescriptions: ["Email correspondence", "Contract terms"],
    evidenceHashes: ["QmHash1", "QmHash2"],
    evidenceSupportsCreator: [true, true]
});
```

### Betting on Dispute

```solidity
// Approve KWN spending
kwn.approve(bettingPool, 1000e18);

// Bet on creator winning
bettingPool.bet(disputeId, 0, 1000e18);

// Bet on respondent winning  
bettingPool.bet(disputeId, 1, 500e18);

// Bet on tie
bettingPool.bet(disputeId, 2, 200e18);
```

### Claiming Winnings

```solidity
// After dispute resolution
bettingPool.claim(disputeId);

// Withdraw NFT accrued KWN
bettingPool.withdrawNftAccrued(tokenId, recipient);
```

## Testing

### Run Tests

```bash
# All tests
forge test

# Specific test file
forge test --match-path test/BettingPool.t.sol

# Verbose output
forge test -vvv

# Gas reporting
forge test --gas-report
```

### Test Coverage

```bash
# Generate coverage report
forge coverage --report lcov
```

## Security Features

- **Reentrancy Protection**: All external calls protected
- **Access Control**: Role-based permissions
- **Pausable**: Emergency pause functionality
- **Upgradeable**: DisputeMarket can be upgraded
- **Input Validation**: Comprehensive parameter checks
- **Safe Math**: Built-in overflow protection

## Economic Model

### Fee Distribution

- **Protocol Treasury**: 1% of betting pools
- **NFT Winners**: 3% of betting pools
- **Bettors**: 96% of betting pools (minus their side's losses)

### Incentive Alignment

- **Evidence Submitters**: Must submit evidence to vote
- **Voters**: Influence dispute outcomes
- **Bettors**: Financial incentive for accurate predictions
- **Dispute Winners**: NFT value increases with betting activity

## Network Support

- **Base Chain**: Primary deployment target
- **Kaiatestnet**: Test deployment
- **Ethereum**: Compatible but not primary target

## Development

### Prerequisites

- Foundry (forge, cast, anvil)
- Node.js 16+
- Solidity 0.8.20+

### Setup

```bash
# Clone repository
git clone <repo-url>
cd NFTField-

# Install dependencies
forge install

# Build contracts
forge build

# Run tests
forge test
```

### Contract Verification

```bash
# Verify on block explorer
forge verify-contract <contract-address> src/Contract.sol:Contract \
    --chain-id <chain-id> \
    --etherscan-api-key <api-key>
```

## Contributing

1. Fork the repository
2. Create feature branch
3. Add tests for new functionality
4. Ensure all tests pass
5. Submit pull request

## License

MIT License - see LICENSE file for details

## Support

For questions and support:
- Open an issue on GitHub
- Check the documentation
- Review the test files for usage examples

---

**Note**: This is a complex financial system. Always test thoroughly on testnets before mainnet deployment. The betting system involves real money and should be used responsibly.

  
