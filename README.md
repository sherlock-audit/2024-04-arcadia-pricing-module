
# Arcadia Pricing Module contest details

- Join [Sherlock Discord](https://discord.gg/MABEWyASkp)
- Submit findings using the issue page in your private contest repo (label issues as med or high)
- [Read for more details](https://docs.sherlock.xyz/audits/watsons)

# Q&A

### Q: On what chains are the smart contracts going to be deployed?
Base
___

### Q: If you are integrating tokens, are you allowing only whitelisted tokens to work with the codebase or any complying with the standard? Are they assumed to have certain properties, e.g. be non-reentrant? Are there any types of <a href="https://github.com/d-xo/weird-erc20" target="_blank" rel="noopener noreferrer">weird tokens</a> you want to integrate?
Please raise issues regarding the underlying tokens that can currently be used: WETH, DAI, COMP, USDBC, USDC, CBETH, RETH, STG, wstETH

We also want to receive issues regarding minimal implementations of ERC20 Tokens, which besides a standard implementation have one of the following "weird behaviours":

- **Revert on Zero Value**
- **Revert on Non-Zero to Non-Zero Approval**
- **Doesn’t Revert on Failure**
- **Blacklisting** - This may cause liquidations to not end, but in that case the protocol takes ownership of the Account to manually resolve the situation after cutOffTime.
- **Token upgradeability** - Although unpredictable, if it still adheres to the token standard EIP after upgrade, it should be OK.
- **Over 18 decimals** - We check internally in the asset modules so that no asset can be added as collateral which has more than 18 decimals. This has to be checked in any future asset modules as well.
- **Pausability** - If a token is ‘paused’ (ie not transferable), it may lead to similar issues as being blacklisted.
- **Flash-mintable tokens**
- **Callback tokens (ERC777)**
- **non-bool returning tokens**
___

### Q: Are the admins of the protocols your contracts integrate with (if any) TRUSTED or RESTRICTED? If these integrations are trusted, should auditors also assume they are always responsive, for example, are oracles trusted to provide non-stale information, or VRF providers to respond within a designated timeframe?
The protocols we integrate with are:

AeroPool.sol:
- Solidly fork
- Almost trustless
- Admin can change fee, but that does not impact LPs
- RESTRICTED

Gauge.sol:
- Owner should ensure the balance of reward token of the contract is sufficient.
- Worst case anyone can top up the balance if owner is non-responsive
- Apart from the mentioned top-up issue RESTRICTED

Slipstream (CLPool.sol and NonfungiblePositionManager.sol):
- Uniswap V3 fork
- Almost trustless
- Admin can change fee, but that does not impact LPs
- RESTRICTED

Although not directly in scope, Chainlink and admins of the contracts for primary assets are TRUSTED.
 
___

### Q: Are there any protocol roles? Please list them and provide whether they are TRUSTED or RESTRICTED, or provide a more comprehensive description of what a role can and can't do/impact.
Owner Protocol (core team):
- Can add stable pools to the AerodromePoolAM.sol.
- Can set baseURIs (the NFTs represent LP positions, there is no value in the 'image' shown). 
- Can initialise or activate contracts.
- TRUSTED

Risk Manager
- Can set the riskVariables (collateral and liquidation factors, maxExposures) for the Creditor to which it's linked.
- TRUSTED
___

### Q: For permissioned functions, please list all checks and requirements that will be made before calling the function.
addAsset() of a stable AeroPool to AerodromePoolAM:
Owner will first check that the total supply of both tokens is smaller than 15511800964 * 10 ** decimals (and will always remain smaller).


___

### Q: Is the codebase expected to comply with any EIPs? Can there be/are there any deviations from the specification?
The WrappedAerodromeAM and StakedAerodromeAM are optionally compliant with ERC721. Any extension to ERC721 (enumerable, metadata, ...) is not in scope.
___

### Q: Are there any off-chain mechanisms or off-chain procedures for the protocol (keeper bots, arbitrage bots, etc.)?
Not in scope of the asset modules of this audit.
(In the protocol there are off-chain mechanisms e.g. with arbitrage bots for the liquidations, but those where audited in our previous Sherlock audit).
___

### Q: Are there any hardcoded values that you intend to change before (some) deployments?
No, some addresses will be passed on deployment to the module (via deployscript, not in scope): registry, aerodromeFactory, aerodromeVoter, nonFungiblePositionManager. Risk variables can be changed in comparison to (out of scope) deployscripts.
___

### Q: If the codebase is to be deployed on an L2, what should be the behavior of the protocol in case of sequencer issues (if applicable)? Should Sherlock assume that the Sequencer won't misbehave, including going offline?
In the protocol there are mitigations implemented, but those where audited in our previous Sherlock audit.
For the asset modules in scope of this audit, there should be no direct consequences if the Sequencer goes down.

If there are additional issues, not covered by the current mitigations, that would be a valid issue.
___

### Q: Should potential issues, like broken assumptions about function behavior, be reported if they could pose risks in future integrations, even if they might not be an issue in the context of the scope? If yes, can you elaborate on properties/invariants that should hold?
Yes.
___

### Q: Please discuss any design choices you made.
AerodromePoolAM.sol:
- When calculating the Trusted Reserves, we work with the idealised AMM bonding curves and ignore fees (k = r0 * r1 and k = x³y + y³x).
- We do lose some precision in the maths with _getTrustedReservesStable() (to avoid overflows), this should however always lead to under-estimating the actual amounts).

WrappedAerodromeAM.sol:
- "fee0PerLiquidity" and "fee1PerLiquidity" can overflow, what matters is the delta in FeePerLiquidity between two interactions. We assume that no users waits with claiming fees until fee0PerLiquidity did a full cycle (in that case unclaimed fees will be lost).
- The total amount wrapped is stored on the contract and not fetched via a balanceOf(). Discrepancies between both (if tokens are not properly wrapped but directly transferred) will always underestimate the actual claimable amount of fees.
___

### Q: Please list any known issues/acceptable risks that should not result in a valid finding.
- Some tokens can blacklist users or pause operations.
- Id’s of ERC721 should not exceed uint96 (→ if they can exceed uint96 they should be wrapped in a new contract before they can be deposited).
- Balances of ERC20 or ERC1155 or the USD value of derived assets (with 18 decimals) are assumed to never exceed a uint112.
- If pool tokens are transferred without depositing before any position is minted, the pool can have non zero fees balances while totalWrapped_ is 0. In this case the fees are not accounted for and will be lost.
- Slipstream is not yet live on Base. Current implementation of SlipstreamAM.sol is based on the version deployed on Optimism. Before actually deploying the AM, we will verify that code is 100% identical.
- Any issue related to EVM versions and L2 compatibility.
- Any issue previously found or mentioned in one of the audits.
___

### Q: We will report issues where the core protocol functionality is inaccessible for at least 7 days. Would you like to override this value?
No
___

### Q: Please provide links to previous audits (if any).
This is an Update Contest of our codebase (where we add additional asset modules):
https://audits.sherlock.xyz/contests/137

Other audits of the protocol (the asset modules of this contest were NOT in scope for these audits):
https://github.com/arcadia-finance/arcadia-finance-audits/tree/main/audits-v2
___

### Q: Please list any relevant protocol resources.
Docs: https://docs.arcadia.finance/
Whitepaper: https://github.com/arcadia-finance/whitepapers/blob/main/main.pdf
Website: https://arcadia.finance/
Twitter: https://twitter.com/ArcadiaFi
___

### Q: Additional audit information.
SlipstreamAM.sol is a fork of the already audited UniswapV3AM.sol (only 8 lines of code and 1 library difference).

To install a local repo:
- install Accounts V2 (https://github.com/arcadia-finance/accounts-v2.git)
- add RPC_URL=<rpc url to a base mainnet (or fork)> in your .env
gh repo clone arcadia-finance/accounts-v2
cd accounts-v2
git checkout audit/sherlock
git pull
forge install
forge build
forge test
___



# Audit scope


[accounts-v2 @ c0b9c92c210aac1b759c78215dbf40d40e85bf66](https://github.com/arcadia-finance/accounts-v2/tree/c0b9c92c210aac1b759c78215dbf40d40e85bf66)
- [accounts-v2/src/asset-modules/Aerodrome-Finance/AerodromePoolAM.sol](accounts-v2/src/asset-modules/Aerodrome-Finance/AerodromePoolAM.sol)
- [accounts-v2/src/asset-modules/Aerodrome-Finance/StakedAerodromeAM.sol](accounts-v2/src/asset-modules/Aerodrome-Finance/StakedAerodromeAM.sol)
- [accounts-v2/src/asset-modules/Aerodrome-Finance/WrappedAerodromeAM.sol](accounts-v2/src/asset-modules/Aerodrome-Finance/WrappedAerodromeAM.sol)
- [accounts-v2/src/asset-modules/Slipstream/SlipstreamAM.sol](accounts-v2/src/asset-modules/Slipstream/SlipstreamAM.sol)


