# Contracts

## Core

### Registry
The DAO Registry contract is the identity of the DAO. This is the contract address that every adapter usually interacts with.

The scope of the registry is to manage the following:

- The adapter registry - which adapter is being used by this DAO and which access it has to the DAO state.
- The extension registry - which extension is part of the DAO and the adapter's access to it.



> Each non-constant function in the DAO Registry has an access control modifier hasAccess linked to it, to make sure the caller has the right to call it.


The DaoRegistry.sol contract tracks the state of the DAO for 1) Adapter and Extension access, 2) State of Proposals, 3) Membership status. If an Adapter needs to access the DAO Registry, it must be registered to the DAO with the correct access flags.

#### DAO State
The DAO can be in one of the following states:

- CREATION: when the DAO is being deployed via initializeDao, but is not ready to be used.
- READY: when the function finalizeDao has been called, and is now ready to be used.

> Once the DaoState is changed to READY, then the only way to add additional Adapters is via proposal process using the Managing Adapter.

#### Access Flags
The are two main categories of Access Flags for the DAO Registry Contract:

1. ProposalFlag

- EXISTS: right to check if a given proposal id already exists in the DAO storage.
- SPONSORED: right to check if a given proposal id was already sponsored.
- PROCESSED: right to check if a griven proposal id was already processed.

2. AclFlag

- REPLACE_ADAPTER: right to add/remove/replace an adapter, function dao.replaceAdapter.
- SUBMIT_PROPOSAL: right to submit a proposal, function dao.submitProposal.
- UPDATE_DELEGATE_KEY: right to update the delegate key of a member, function dao.updateDelegatedKey.
- SET_CONFIGURATION: right to set custom configurations parameters, function dao.setConfiguration.
- ADD_EXTENSION: right to add an extension, function dao.addExtension.
- REMOVE_EXTENSION: right to remove an extension, function dao.removeExtension.
- NEW_MEMBER: right to add a new potential member to the DAO, function dao.potentialNewMember.


#### Structs

<h3>Proposal</h3>
<p>
The structure to track all the proposals in the DAO and their state (EXISTS, SPONSORED, PROCESSED).
</p>
<h3>AdapterEntry</h3>
<p>
When an Adapter is added to DaoRegistry via the function replaceAdapter, a bytes32 id and a uint256 acl are parameters assigned to the Adapter for use in identifying the Adapter.
</p>

<h3>ExtensionEntry</h3>
<p>
When an Extension is added to DaoRegistry via addExtenstion a bytes32 id and a uint256 acl are parameters assigned to the Extension for use in identifying the Extension.
</p>

<h3> Storage</h3>

<h4>public state</h4>
<p>
Dao state. This is used to know if the DAO is currently being set up or if it is already running. Useful to configure it.
</p>

<h4>public proposals</h4>
<p>
Mapping of all the proposals for the DAO. Each proposal has an adapterAddress (which adapter created it) and flags to define its state.
</p>
<h4>public adapters</h4>
<p>
Mapping of all the adapters. bytes32 is the keccak256 of their name and address.
</p>
<h4>public inverseAdapters</h4>
<p>
Mapping of adapter details. For each address, we can get its id (keccak256(name)) and its acl (access control, which function in the DAO it has access to).
</p>
<h4>public extensions</h4>
<p>
Mapping of each extension. Like for adapters, the key here is keccak256(name) (e.g., keccak256("bank"))
</p>
<h4>public inverseExtensions</h4>
<p>
Mapping of extension details. For each extension address, you get its id (keccak256(name)) and a mapping from adapter address => access control. Access control for each extension is centralized in the DaoRegistry to avoid each extension implementing its own ACL system.
</p>
<h4>public mainConfiguration</h4>
<p>
Generic configuration mapping from key (keccak256(name)) to any type that can be encoded in 256 bytes (does not need to be uint, could be bytes32 too).
</p>
<h4>public addressConfiguration</h4>
<p>
Since addresses are not encoded in 256 bytes, we need a separate configuration mapping for this type.
</p>

#### Functions

>The constructor function is non-existent, because this is a Cloneable contract. See, https://eips.ethereum.org/EIPS/eip-1167

##### initialize
Initializes the DAO by creating the initial members who are 1) the DAO creator passed to the function, 2) the account passed to the function which paid for the transaction to create the DAO, and 3) the DaoFactory calling this function.

##### finalizeDao
Mark the DAO as finalized. After that, changes can only be made through adapters.

##### setConfiguration
Set a generic configuration entry for the DAO. Only adapters with access to this function can do it.

##### setAddressConfiguration
Set an address configuration entry for the DAO. Only adapters with access to this function can do it.

##### getConfiguration
Get the generic config entry by passing the keccak256(config name).

##### getAddressConfiguration
Get the address config entry by passing the keccak256(config name).

##### addExtension
Add a new extension to the registry. It first checks if the extension id is already used and reverts if it is the case. It then adds the extension to the DAO and initializes it.

##### removeExtension
Removes the extension by extension id. It reverts if no extension has been registered for that id (keccak256(name)).

##### setAclToExtensionForAdapter
Sets the access control for a particular adapter (by address) to a specific extension. Both adapter and extension need to be already registered to the DAO.

##### replaceAdapter
Adds, removes or replaces an adapter om the DAO registry. It also sets the access control. The adapter can be added only if the adapter id is not already in use. To remove an adapter from the DAO just set the address to 0x0.

##### isExtension
Checks whether the address is registered as an extension in the DAO.

##### isAdapter
Checks whether the address is registered as an adapter in the DAO.

##### hasAdapterAccess
Checks whether the adapter has access to a certain flag in the DAO.

##### hasAdapterAccessToExtension
Checks whether a certain adapter has access to a certain extension in the DAO.

##### getAdapterAddress
Returns the adapter address registered for this adapterId and reverts if not found.

The reason we revert here is to avoid the need to check everywhere that the return value is 0x0 when we want to use an adapter.

##### getExtensionAddress
Returns the extension address registered for this extensionId and reverts if not found.

The reason we revert here is to avoid the need to check everywhere that the return value is 0x0 when we want to use an extension.

##### submitProposal
Creates a proposal entry for the DAO. It checks that the proposal was not previously created.

##### sponsorProposal
Marks an existing proposal as sponsored. saves which voting adapter is being used for this proposal. Checks that the proposal has not been sponsored yet. Checks that the proposal exists. Checks that the adapter that sponsors the proposal is the one that submitted it. Checks that the proposal has not been processed yet. Checks that the member sponsoring the proposal is an active member.

##### processProposal
Marks an existing proposal as processed. Checks that the proposal has not been processed already and that it exists.

##### _setProposalFlag
Internal utility function to set a flag to a proposal. It checks that the proposal exists and that the flag has not been already set.

##### getProposalFlag
Helper function to get the flag value for a proposal.

### Factory
The DaoFactory uses the CloneFactory to let you create a cost effective DaoRegistry and initialize and configure it properly. It also serves as a registry of created DAOs to help others find a DAO by name.

#### Structs

##### Adapter
Struct defined to handle the addition and configuration of new adapters using the functions addAdapters, updateAdapter, and configureExtension.

#### Storage

##### public daos
Maps the DAO address to a given sha3(daoName).

##### public addresses
Maps the sha3(daoName) of a DAO to an address.

##### public identityAddress
The address of the identityDao address that is being used to clone the DAO.

#### Functions
##### createDao
Creates and initializes a new DaoRegistry with the DAO creator and the transaction sender. Enters the new DaoRegistry in the DaoFactory state. The daoName must not be in use.

##### getDaoAddress
Returns the DAO address based on its name.

##### addAdapters
Adds adapters and sets their ACL for DaoRegistry functions. A new DAO is instantiated with only the Core Modules enabled, to reduce the call cost. This call must be made to add adapters. The message sender must be an active member of the DAO. The DAO must be in CREATION state.

##### configureExtension
Configures extension to set the ACL for each adapter that needs to access the extension. The message sender must be an active member of the DAO. The DAO must be in CREATION state.

##### updateAdapter
Removes an adapter with a given ID from a DAO, and adds a new one of the same ID. The message sender must be an active member of the DAO. The DAO must be in CREATION state.




## Extension

<h1> FundingPool</h1>
<p>The Fundingpool extension manages the funds of the lp of the DAO. The funds can be any ERC20 token.

On top of that, it implements balance checkpoints so it is possible to retrieve prior balance at a certain block number. The balance is managed for the lp address.

The <code> availableTokens </code> and <code> availableInteralTokens </code>, are tokens that have been whitelisted for use within the DAO. A token goes from <code>tokens</code> or <code>internalTokens</code> to <code>availableTokens</code> and <code>availableInternalTokens</code> respectively when the function <code>registerPotentialNewToken</code> or <code>registerPotentialNewInternalToken</code> is called.</p>

<h2> Access Flags</h2>

- <code>ADD_TO_BALANCE</code>: right to add balance to the bank.
- <code>SUB_FROM_BALANCE</code>: right to subtract balance from the bank.
- <code>WITHDRAW</code>: right to withdraw the funds to an external wallet.
- <code>REGISTER_NEW_TOKEN</code>: right to register a new external token.
- <code>DISTRIBUTE_FUNDS</code>: right to distribute external token from funding pool.
- <code>UPDATE_GP_BALANCE</code>: right to update the gp balance.

<h2> Storage</h2>
<h3>public tokens</h3>
<p>List of all the external tokens that are whitelisted in the funding pool.</p>
<h3>public availableTokens</h3>
<p>Same as the list of tokens but accessible in a random way.</p>
<h3>public checkpoints</h3>
<p>Checkpoint counts for each token / lp.
</p>
<h2>Functions</h2>
<h3>initialize</h3>
<p>This function can be called only once and only by the creator of the DAO.
</p>
<h3>withdraw</h3>
<p>This function is only accessible if you have extension access WITHDRAW.
The function subtracts from the account the amount and then transfers the actual tokens to the account address.
</p>
<h3>addToBalance</h3>
<p>Adds to the lp balance the amount in token. This also updates the checkpoint for this lp / token.
</p>
<h3>subtractFromBalance</h3>
<p>Subtracts from the lp balance the amount in token. This also updates the checkpoint for this lp / token.
</p>
<h3>subtractAllFromBalance</h3>
<p>Subtracts from all the lp balance the amount in token. This also updates the checkpoint for this lp / token.
</p>
<h3>balanceOf</h3>
<p>Returns the balance of a certain account for a certain token.
<h3>getPriorAmount</h3>
<p>Returns the balance of a certain account for a certain token at a certain point in time (block number).
</p>
<h3>_createNewAmountCheckpoint</h3>
<p>Internal function to create a new amount checkpoint (if a balance has been updated).
</p>
<h3>_newInvestor</h3>
<p>Internal function to record lp (if is first time to deposit).
</p>
<h3>_removeInvestor</h3>
<p>Internal function to remove lp (if his balance is zero).
</p>
<h3>totalSupply</h3>
<p>return funding pool token amount.
</p>
<h3>ifInRedemptionPeriod</h3>
<p>check if in redemption period by given time.
</p>
<h3>getInvestors</h3>
<p>return all lps.
</p>
<h3>processFundRaising</h3>
<p>process fund raise when time window is close.
</p>
<h3>distributeFunds</h3>
<p>send token from funding pool to given address.
</p>
<h3>updateTotalGPsBalance</h3>
<p>update gp balance when gp join / leaving.
</p>

<h1> GPDao</h1>
The GPDao extension manages GP (register / remove).
<h2> Storage</h2>
<h3>private _generalPartners</h3>
<p>List of all the GP in the DAO.</p>
<h2>Functions</h2>
<h3>initialize</h3>
<p>This function can be called only once and only by the creator of the DAO. register the contract creator as first GP.
</p>
<h3>registerGeneralPartner</h3>
<p>register new GP.
</p>
<h3>removeGneralPartner</h3>
<p>remove GP from DAO.
</p>
<h3>isGeneralPartner</h3>
<p>check if GP.
</p>
<h3>getAllGPs</h3>
<p>return all the GP.
</p>




## Adapter 


<h2>DistributeFund</h2>
<p>DistributeFund Adapter manages proposals that project raise fund from the pool.
if the proposal passes, the funds are released to the project and the project tokens are released to the lp.
</p>
<h3>Workflow</h3>
<p>Submit a proposal and check:
</p>
<p>

- if caller is a GP member
- if fund raise success
- if fund raise window close
- if stop vesting time > start vesting time
- if requeset fund amount > 0
- if trading off project token amount > 0

</p>

<p>
If all the requirements pass, then the proposal is subitted to registry and the adapter stores the proposal data.
</p>
<p>
Once the voting period ends, anyone can process the proposal. The proposal is processed only if:
</p>

<p>

- it has not been processed already
- the proposal has been sponsored
- the voting has passed

</p>

<h3>Access Flags</h3>
<h4>DaoRegistry</h4>

<p>

- SUBMIT_PROPOSAL

</p>

<h3>Dependencies</h3>
<h4>DaoRegistry</h4>
<h4>GPVoting</h4>
<h4>FundingPool</h4>

<h3>Structs</h3>
<h4>Distribution</h4>
<p>

- <code>tokenAddr</code>: The project token address.
- <code>tradingOffTokenAmount</code>: The release project token amount.
- <code>requestedFundAmount</code>: The USDT amount project team requesting.
- <code>fullyReleasedDate</code>: The stop vesting time of project token.
- <code>lockupDate</code>: The start vesting time of project token.
- <code>recipientAddr</code>: The project team address.
- <code>proposer</code>: The proposer address.
- <code>status</code>: The proposal state.
- <code>inQueueTimestamp</code>: The time of proposal submitted.
- <code>proposalStartVotingTimestamp</code>: The time of proposal start voting.
- <code>proposalStopVotingTimestamp</code>: The time of proposal stop voting.
- <code>proposalExecuteTimestamp</code>: The time of proposal stop executing.

</p>
<h3>Storage</h3>
<h4>distributions</h4>
<p>All the proposals handled by the adapter per DAO.
</p>
<h4>projectTeamLockedTokens</h4>
<p>All the project token escrow by the adapter per DAO.
</p>
<h4>ongoingDistributions</h4>
<p>proposal processing currently.
</p>
<h4>proposalQueue</h4>
<p>All the proposals waiting in a queue.
</p>
<h3>Functions</h3>
<h4>submitProposal</h4>
<h4>processProposal</h4>

<h2>FundingPool</h2>
<p>FundingPool adapter call function from FundingPool extension.
</p>

<h3>Access Flags</h3>
<h4>FundingPoolExtension</h4>
<p>

- WITHDRAW
- ADD_TO_BALANCE

</p>
<h3>Dependencies</h3>
<h4>DaoRegistry</h4>
<h4>FundingPool Extension</h4>
<h3>Functions</h3>
<h4>withdraw</h4>
<h4>deposit</h4>
<h4>balanceOf</h4>
<h4>lpBalance</h4><h4>gpBalance</h4><h4>getFundRaisingMaxAmount</h4><h4>getMinInvestmentForLP</h4>
<h4>getFundRaisingTarget</h4><h4>gpBalance</h4><h4>getFundRaiseWindowOpenTime</h4><h4>getFundRaiseWindowCloseTime</h4>
<h4>getFundStartTime</h4><h4>getFundEndTime</h4><h4>getRedemptDuration</h4>

<h2>GPDao</h2>
<p>GPDao adapter call function from GPDao extension</p>
<h3>Access Flags</h3>
<h4>GPDao Extension</h4>

<p>

- REMOVE_GP

</p>
<h3>Dependencies</h3>
<h4>DaoRegistry</h4>
<h4>GPDao Extension</h4>
<h3>Functions</h3>
<h4>gpQuit</h4>
<p>let GP quit out by himself</p>


<h2>GPKick</h2>
<p>GPKick Adapter manages proposals that kick GP from DAO.
if the proposal passes, the target GP will kick out GP member.
</p>
<h3>Workflow</h3>

<p>Submit a proposal and check:
</p>
<p>

- if caller is a GP member
- if gp to kick is a valid address and is not the summonor of the DAO
- if proposer and gp to kick not the same one

</p>

<p>
If all the requirements pass, then the proposal is subitted to registry and the adapter stores the proposal data.
</p>
<p>
Once the voting period ends, anyone can process the proposal. The proposal is processed only if:
</p>

<p>

- it has not been processed already
- the proposal has been sponsored
- the voting has passed

</p>

<h3>Access Flags</h3>
<h4>DaoRegistry</h4>

<p>

- SUBMIT_PROPOSAL

</p>
<h4>GPDao Extension</h4>

<p>

- REMOVE_GP

</p>

<h3>Dependencies</h3>
<h4>DaoRegistry</h4>
<h4>GPVoting</h4>
<h4>GPDao Extension</h4>

<h3>Structs</h3>
<h4>GPKickProposal</h4>
<p>

- <code>gpToKick</code>: gp to kick address(cant be proposer)
- <code>state</code>: proposal state
- <code>creationTime</code>: proposal start voting time
- <code>stopVoteTime</code>: proposal stop voting time

</p>
<h3>Storage</h3>
<h4>kickProposals</h4>
<p>All the proposals handled by the adapter per DAO.
</p>
<h3>Functions</h3>
<h4>submitProposal</h4>
<h4>processProposal</h4>

<h3>Events</h3>
<h4>ProposalCreated</h4>
<p>When a proposal submitted. Event emitted by the adapter.
</p>
<p>

- <code> 
    event ProposalCreated(
        bytes32 proposalId,
        address gpToKick,
        uint256 creationTime,
        uint256 stopVoteTime
    );</code>

</p>

<h4>ProposalProcessed</h4>
<p>When a proposal processed. Event emitted by the adapter.
</p>
<p>

- <code>event ProposalProcessed(
        bytes32 proposalId,
        IGPOnboardingVoting.VotingState voteRelsult,
        ProposalState state,
        uint128 allVotingWeight,
        uint128 nbYes,
        uint128 nbNo
    );</code>

</p>

<h2>GPOnboarding</h2>
<p>GPOnboarding Adapter manages proposals that add someone as GP member.
if the proposal passes, the applicant will register to GP member.
</p>
<h3>Workflow</h3>

<p>Submit a proposal and check:
</p>
<p>

- if caller is a GP member

</p>

<p>
If all the requirements pass, then the proposal is subitted to registry and the adapter stores the proposal data.
</p>
<p>
Once the voting period ends, anyone can process the proposal. The proposal is processed only if:
</p>

<p>

- it has not been processed already
- the proposal has been sponsored
- the voting has passed

</p>

<h3>Access Flags</h3>
<h4>DaoRegistry</h4>

<p>

- SUBMIT_PROPOSAL

</p>
<h4>GPDao Extension</h4>

<p>

- REGISTER_NEW_GP

</p>

<h3>Dependencies</h3>
<h4>DaoRegistry</h4>
<h4>GPVoting</h4>
<h4>GPDao Extension</h4>

<h3>Structs</h3>
<h4>ProposalDetails</h4>
<p>

- <code>id</code>: proposal Id
- <code>applicant</code>: address who want to be GP
- <code>creationTime</code>: proposal start voting time
- <code>stopVoteTime</code>: proposal stop voting time
- <code>state</code>: proposal state

</p>
<h3>Storage</h3>
<h4>proposals</h4>
<p>All the proposals handled by the adapter per DAO.
</p>
<h3>Functions</h3>
<h4>submitProposal</h4>
<h4>processProposal</h4>

<h3>Events</h3>
<h4>ProposalCreated</h4>
<p>When a proposal submitted. Event emitted by the adapter.
</p>
<p>

- <code> event ProposalCreated(
        bytes32 proposalId,
        address applicant,
        uint256 creationTime,
        uint256 stopVoteTime
    );
    </code>

</p>

<h4>ProposalProcessed</h4>
<p>When a proposal processed. Event emitted by the adapter.
</p>
<p>

- <code>event ProposalProcessed(
        bytes32 proposalId,
        IGPOnboardingVoting.VotingState voteRelsult,
        ProposalState state,
        uint128 allVotingWeight,
        uint128 nbYes,
        uint128 nbNo
    );</code>

</p>


<h2>Managing</h2>
<p>The Managing adapter handles the proposal creation, sponsorship and processing of a new adapter/extension including its initial configuration, and permissions.</p>
<p>An adapter/extension can be added, removed or replaced in the DAO registry. In order to remove an adapter/extension one must pass the address 0x0 with the adapter/extension id that needs to be removed. To add a new adapter/extension one most provide the adapter/extension id, address and access flags. The address of the new adapter/extension can not be a reserved address, and the id must be a known id as defined in the <code>DaoConstants.sol</code>. The replace adapter/extension operation removes the adapter/extension from the registry based on the id parameter, and also adds a new adapter/extension using the same id but with a new address.</p>

<p>The Managing adapter also allows one set custom ACL flags to adapters that need to communicate with different extensions.
> It is important to indicate which operation type will be performed. Set 1 to updateType when you are adding/removing an Adapter, and set 2 when you are adding/removing an Extension.
</p>

<h3>Workflow</h3>

<p>Submit a proposal and check:
</p>
<p>

- if caller is a GP member
- if keys and values have equal length
- if adapter/extension address is valid
- if the access flags don't overflow
- if adapter/extension address is not reserved

</p>

<p>
If all the requirements pass, then the proposal is subitted to registry and the adapter stores the proposal data.
</p>
<p>
Once the voting period ends, anyone can process the proposal. The proposal is processed only if:
</p>

<p>

- it has not been processed already
- the proposal has been sponsored
- the voting has passed
- the updateType is 1 (adapter) or 2 (extension)
- the extension AclFlags, and address are valid

</p>

<h3>Access Flags</h3>
<h4>DaoRegistry</h4>

<p>

- SUBMIT_PROPOSAL
- REPLACE_ADAPTER
- ADD_EXTENSION
- REMOVE_EXTENSION

</p>

<h3>Dependencies</h3>
<h4>DaoRegistry</h4>
<h4>GPVoting</h4>

<h3>Structs</h3>
<h4>ProposalDetails</h4>
<p>

- <code>adapterOrExtensionId</code>: The id of the adapter/extension to add, remove or replace.
- <code>adapterOrExtensionAddr</code>: The contract address of the adapter/extension.
- <code>flags</code>: The DAO ACL for the new adapter.
- <code>keys</code>: The configuration keys for the adapter.
- <code>values</code>: The values to set for the adapter configuration.
- <code>extensionAddresses</code>: The list of extension addresses that the adapter interacts with.
- <code>extensionAclFlags</code>: The list of ACL flags that the adapter needs to interact with each extension.

</p>


<h3>Storage</h3>
<h4>proposals</h4>
<p>All the proposals handled by the adapter per DAO.
</p>

<h3>Functions</h3>
<h4>submitProposal</h4>
<h4>processProposal</h4>

<h3>Events</h3>
<h4>AdapterRemoved</h4>
<p>When an adapter is removed from the regitry. Event emitted by the DAO Registry.
</p>
<p>

- <code>event AdapterRemoved(bytes32 adapterId);</code>

</p>

<h4>AdapterAdded</h4>
<p>When a new adapter is added to the registry. Event emitted by the DAO Registry.
</p>
<p>

- <code>event AdapterAdded(bytes32 adapterId, address adapterAddress, uint256 flags);</code>

</p>
<h4>ConfigurationUpdated</h4>
<p>When a new configuration is stored in the registry. Event emitted by the DAO Registry.
</p>
<p>

- <code>event ConfigurationUpdated(bytes32 key, uint256 value);</code>

</p>


