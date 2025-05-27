# ENS

[![Build Status](https://travis-ci.org/ensdomains/ens-contracts.svg?branch=master)](https://travis-ci.org/ensdomains/ens-contracts)

For documentation of the ENS system, see [docs.ens.domains](https://docs.ens.domains/).

## npm package

This repo doubles as an npm package with the compiled JSON contracts

```js
import {
  BaseRegistrar,
  BaseRegistrarImplementation,
  BulkRenewal,
  ENS,
  ENSRegistry,
  ENSRegistryWithFallback,
  ETHRegistrarController,
  FIFSRegistrar,
  LinearPremiumPriceOracle,
  PriceOracle,
  PublicResolver,
  Resolver,
  ReverseRegistrar,
  StablePriceOracle,
  TestRegistrar,
} from '@ensdomains/ens-contracts'
```

## Importing from solidity

```
// Registry
import '@ensdomains/ens-contracts/contracts/registry/ENS.sol';
import '@ensdomains/ens-contracts/contracts/registry/ENSRegistry.sol';
import '@ensdomains/ens-contracts/contracts/registry/ENSRegistryWithFallback.sol';
import '@ensdomains/ens-contracts/contracts/registry/ReverseRegistrar.sol';
import '@ensdomains/ens-contracts/contracts/registry/TestRegistrar.sol';
// EthRegistrar
import '@ensdomains/ens-contracts/contracts/ethregistrar/BaseRegistrar.sol';
import '@ensdomains/ens-contracts/contracts/ethregistrar/BaseRegistrarImplementation.sol';
import '@ensdomains/ens-contracts/contracts/ethregistrar/BulkRenewal.sol';
import '@ensdomains/ens-contracts/contracts/ethregistrar/ETHRegistrarController.sol';
import '@ensdomains/ens-contracts/contracts/ethregistrar/LinearPremiumPriceOracle.sol';
import '@ensdomains/ens-contracts/contracts/ethregistrar/PriceOracle.sol';
import '@ensdomains/ens-contracts/contracts/ethregistrar/StablePriceOracle.sol';
// Resolvers
import '@ensdomains/ens-contracts/contracts/resolvers/PublicResolver.sol';
import '@ensdomains/ens-contracts/contracts/resolvers/Resolver.sol';
```

## Accessing to binary file.

If your environment does not have compiler, you can access to the raw hardhat artifacts files at `node_modules/@ensdomains/ens-contracts/artifacts/contracts/${modName}/${contractName}.sol/${contractName}.json`

## Contracts

## Registry

The ENS registry is the core contract that lies at the heart of ENS resolution. All ENS lookups start by querying the registry. The registry maintains a list of domains, recording the owner, resolver, and TTL for each, and allows the owner of a domain to make changes to that data. It also includes some generic registrars.

### ENS.sol

Interface of the ENS Registry.

### ENSRegistry

Implementation of the ENS Registry, the central contract used to look up resolvers and owners for domains.

### ENSRegistryWithFallback

The new implementation of the ENS Registry after [the 2020 ENS Registry Migration](https://docs.ens.domains/ens-migration-february-2020/technical-description#new-ens-deployment).

### FIFSRegistrar

Implementation of a simple first-in-first-served registrar, which issues (sub-)domains to the first account to request them.

### ReverseRegistrar

Implementation of the reverse registrar responsible for managing reverse resolution via the .addr.reverse special-purpose TLD.

### TestRegistrar

Implementation of the `.test` registrar facilitates easy testing of ENS on the Ethereum test networks. Currently deployed on Ropsten network, it provides functionality to instantly claim a domain for test purposes, which expires 28 days after it was claimed.

## EthRegistrar

Implements an [ENS](https://ens.domains/) registrar intended for the .eth TLD.

These contracts were audited by ConsenSys Diligence; the audit report is available [here](https://github.com/ConsenSys/ens-audit-report-2019-02).

### BaseRegistrar

BaseRegistrar is the contract that owns the TLD in the ENS registry. This contract implements a minimal set of functionality:

- The owner of the registrar may add and remove controllers.
- Controllers may register new domains and extend the expiry of (renew) existing domains. They can not change the ownership or reduce the expiration time of existing domains.
- Name owners may transfer ownership to another address.
- Name owners may reclaim ownership in the ENS registry if they have lost it.
- Owners of names in the interim registrar may transfer them to the new registrar, during the 1 year transition period. When they do so, their deposit is returned to them in its entirety.

This separation of concerns provides name owners strong guarantees over continued ownership of their existing names, while still permitting innovation and change in the way names are registered and renewed via the controller mechanism.

### EthRegistrarController

EthRegistrarController is the first implementation of a registration controller for the new registrar. This contract implements the following functionality:

- The owner of the registrar may set a price oracle contract, which determines the cost of registrations and renewals based on the name and the desired registration or renewal duration.
- The owner of the registrar may withdraw any collected funds to their account.
- Users can register new names using a commit/reveal process and by paying the appropriate registration fee.
- Users can renew a name by paying the appropriate fee. Any user may renew a domain, not just the name's owner.

The commit/reveal process is used to avoid frontrunning, and operates as follows:

1.  A user commits to a hash, the preimage of which contains the name to be registered and a secret value.
2.  After a minimum delay period and before the commitment expires, the user calls the register function with the name to register and the secret value from the commitment. If a valid commitment is found and the other preconditions are met, the name is registered.

The minimum delay and expiry for commitments exist to prevent miners or other users from effectively frontrunning registrations.

### SimplePriceOracle

SimplePriceOracle is a trivial implementation of the pricing oracle for the EthRegistrarController that always returns a fixed price per domain per year, determined by the contract owner.

### StablePriceOracle

StablePriceOracle is a price oracle implementation that allows the contract owner to specify pricing based on the length of a name, and uses a fiat currency oracle to set a fixed price in fiat per name.

## Resolvers

Resolver implements a general-purpose ENS resolver that is suitable for most standard ENS use cases. The public resolver permits updates to ENS records by the owner of the corresponding name.

PublicResolver includes the following profiles that implements different EIPs.

- ABIResolver = EIP 205 - ABI support (`ABI()`).
- AddrResolver = EIP 137 - Contract address interface. EIP 2304 - Multicoin support (`addr()`).
- ContentHashResolver = EIP 1577 - Content hash support (`contenthash()`).
- InterfaceResolver = EIP 165 - Interface Detection (`supportsInterface()`).
- NameResolver = EIP 181 - Reverse resolution (`name()`).
- PubkeyResolver = EIP 619 - SECP256k1 public keys (`pubkey()`).
- TextResolver = EIP 634 - Text records (`text()`).
- DNSResolver = Experimental support is available for hosting DNS domains on the Ethereum blockchain via ENS. [The more detail](https://veox-ens.readthedocs.io/en/latest/dns.html) is on the old ENS doc.

## Developer guide

### Prettier pre-commit hook

This repo runs a husky precommit to prettify all contract files to keep them consistent. Add new folder/files to `prettier format` script in package.json. If you need to add other tasks to the pre-commit script, add them to `.husky/pre-commit`

### How to setup

```
git clone https://github.com/ensdomains/ens-contracts
cd ens-contracts
bun i
```

### How to run tests

```
bun run test
```

### How to publish

```
bun run pub
```

### Deployment

```
NODE_OPTIONS='--experimental-loader ts-node/esm/transpile-only' bun run hardhat --network <network_name> deploy
```

Full list of available networks for deployment is [here](hardhat.config.cts#L38).

### Release flow

1. Create a `feature` branch from `staging` branch
2. Make code updates
3. Ensure you are synced up with `staging`
4. Code should now be in a state where you think it can be deployed to production
5. Create a "Release Candidate" [release](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases) on GitHub. This will be of the form `v1.2.3-RC0`. This tagged commit is now subject to our bug bounty.
6. Have the tagged commit audited if necessary
7. If changes are required, make the changes and then once ready for review create another GitHub release with an incremented RC value `v1.2.3-RC0` -> `v.1.2.3-RC1`. Repeat as necessary.
8. Deploy to testnet. Open a pull request to merge the deploy artifacts into
   the `feature` branch. Create GitHub release of the form `v1.2.3-testnet` from the commit that has the new deployment artifacts.
9. Get someone to review and approve the deployment and then merge. You now MUST merge this branch into `staging` branch.
10. If any further changes are needed, you can either make them on the existing feature branch that is in sync or create a new branch, and follow steps 1 -> 9. Repeat as necessary.
11. Make a deployment to ethereum mainnet from `staging`. Create a GitHub release of the form `v1.2.3` from the commit that has the new deployment artifacts.
12. Open a PR to merge into `main`. Have it reviewed and merged.

### Cherry-picked release flow

Certain changes can be released in isolation via cherry-picking, although ideally we would always release from `staging`.

1. Create a new branch from `mainnet`.
2. Cherry-pick from `staging` into new branch.
3. Deploy to ethereum mainnet, tag the commit that has deployment artifacts and create a release.
4. Merge into `mainnet`.

### Emergency release process

1. Branch from `main`, make fixes, deploy to testnet (can skip), deploy to mainnet
2. Merge changes back into `main` and `staging` immediately after deploy
3. Create GitHub releases, if you didn't deploy to testnet in step 1, do it now

### Notes

- Deployed code should always match source code in mainnet releases. This may not be the case for `staging`.
- `staging` branch and `main` branch should start in sync
- `staging` is intended to be a practice `main`. Only code that is intended to be released to `main` can be merged to `staging`. Consequently:
  - Feature branches will be long-lived
  - Feature branches must be kept in sync with `staging`
  - Audits are conducted on feature branches
- All code that is on `staging` and `main` should be deployed to testnet and mainnet respectively i.e. these branches should not have any undeployed code
- It is preferable to not edit the same file on different feature branches.
- Code on `staging` and `main` will always be a subset of what is deployed, as smart contracts cannot be undeployed.
- Release candidates, `staging` and `main` branch are subject to our bug bounty
- Releases follow semantic versioning and releases should contain a description of changes with developers being the intended audience

## Web3 Security Researcher Guidelines

Guideline 1: Protocol Understanding and Documentation

Understand the repristory and how the smart contracts work. As a Web3 security researcher, your primary task is to deeply understand a given protocol. This involves:

Comprehensive Analysis: Conduct a thorough examination of the Solidity smart contracts and accompanying documentation. Architectural Overview: Provide a high-level overview of the protocol, detailing its components, architecture, and overall functionality. Smart Contract Breakdown: For each smart contract or component: Clearly explain its purpose and function within the protocol. Generate a diagram illustrating its functionality and interactions with other smart contracts. Use established diagramming conventions (e.g., Mermaid syntax) for clarity. User Flow Analysis: Describe the typical user flows within the protocol. Create diagrams illustrating these user flows, highlighting key steps and interactions. Glossary Creation: Compile a glossary of terms used within the protocol, presented as a Markdown table with clear definitions.

Guideline 2: Vulnerability Discovery

As a Web3 security researcher specializing in Solidity smart contracts, your primary goal is to proactively identify vulnerabilities that could lead to critical exploits. Emulate the rigorous mindset of a security auditor, with a focus on uncovering weaknesses related to:

Vulnerability Focus:

Prioritize identifying vulnerabilities that could result in:

"Critical
Direct theft of any user NFTs, whether at-rest or in-motion, other than unclaimed royalties

Critical
Manipulation of governance voting result deviating from voted outcome and resulting in a direct change from intended effect of original results

Critical
Direct theft of any user funds, whether at-rest or in-motion, other than unclaimed yield

Critical
Permanent freezing of funds

Critical
Permanent freezing of NFTs

Critical
Unauthorized minting of NFTs

Critical
Predictable or manipulable RNG that results in abuse of the principal or NFT

Critical
Unintended alteration of what the NFT represents (e.g. token URI, payload, artistic content)

Critical
Protocol insolvency

Critical
Theft of treasury funds

Critical
Permanent freezing of treasury funds

Critical
Retrieve sensitive data/files from a running server, such as: Access tokens"

Direct Theft of Funds: Vulnerabilities enabling the unauthorized transfer of user funds, whether at rest or in motion (excluding unclaimed yield). Permanent Freezing of Funds: Mechanisms that could permanently lock user funds, rendering them inaccessible. Theft of Unclaimed Yield: Exploits allowing unauthorized access to and theft of unclaimed yield. Permanent Denial-of-Service (DoS): Vulnerabilities leading to a permanent inability to use core contract functionalities (excluding volumetric attacks). Governance Manipulation: Exploits that could manipulate governance voting outcomes. Illegitimate Minting: Vulnerabilities enabling the unauthorized minting of protocol-native assets. Incorrect Behavior Analysis:

Investigate potential causes of incorrect smart contract behavior that could lead to unintended functionality, including but not limited to:

Financial Exploits: Stealing or loss of funds due to flawed logic. Unauthorized transactions executed without proper authorization. Transaction manipulation to benefit an attacker. Price manipulation through contract vulnerabilities. Fee payment bypass, allowing users to avoid paying intended fees. Balance manipulation, leading to incorrect accounting of assets. Contract Logic and Execution Flaws: Flaws in contract execution flows, leading to unexpected state changes. Cryptographic flaws, such as weak randomness or signature vulnerabilities. Attacks on logic where the code's behavior deviates from the intended business description. Reentrancy vulnerabilities, allowing recursive calls to exploit state inconsistencies. Integer overflow and underflow vulnerabilities, leading to incorrect calculations. Missing Checks: Missing or insufficient input validation. Lack of proper access control mechanisms. Absence of necessary state checks before critical operations. Cross-Contract Analysis:

Actively identify cross-contract vulnerabilities arising from interactions between multiple contracts, creating exploitable conditions. Pay close attention to:

Unexpected interactions between contracts. State inconsistencies across contracts. Vulnerabilities that can be amplified through cross-contract calls. Advanced Analysis:

When a potential vulnerability is identified:

In-Depth Verification: Perform a thorough analysis, including cross-contract analysis, to confirm the vulnerability's legitimacy and potential impact. Mitigation Assessment: Evaluate existing protections or mitigations that might reduce the vulnerability's severity or likelihood of exploitation. Exploit Scenario: Develop a detailed exploit scenario demonstrating how the vulnerability could be exploited in a real-world attack. Your analysis should consider all possible attack vectors and edge cases to provide a comprehensive security assessment.

Guideline 3: Vulnerability Verification

As a Web3 security researcher/auditor, your primary responsibility upon identifying a potential vulnerability is to rigorously verify its exploitability with absolute certainty. We require a comprehensive assessment, leaving no room for assumptions or hypotheses. This demands a meticulous, line-by-line analysis to definitively determine if a genuine vulnerability exists. Your verification process must include:

Codebase Mastery: Demonstrate an in-depth understanding of the relevant codebase, including the intricacies of function logic and data flow.

Exploit Path Confirmation: Based on your expert understanding of the codebase, confirm with 100% certainty whether a realistic exploit path exists, considering the current state of the code and any potential dependencies. Detail the exact steps required to execute a successful exploit.

Mitigation Awareness: Conduct a thorough investigation and provide detailed documentation of all existing protections, mitigations, or countermeasures that could potentially hinder or prevent the exploitation of the identified vulnerability. Explain precisely how these measures impact the exploitability.
