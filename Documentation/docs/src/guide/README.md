# Venture DAO Framework

## Contents

- Overview
- Proposal
- Architecture
- Quickstart

## Overview
VentureDAO is a new modular, low cost DAO framework. The framework aims to improve DAOs by fixing the:

- Lack of modularity: which has created challenges both in terms of extending, managing, and upgrading DAOs;
- Rigid voting and governance mechanisms: which limit the ability to experiment with additional forms of governance;
- High costs: especially for onchain voting;
- Single token DAO structures: which make it difficult to divide up economic and governance rights and create teams or sub-groups;

## Architecture

The main design goal is to limit access to the smart contracts according at layer boundaries. The External World (i.e. RPC clients) can access the core contracts only via Adapters, never directly. Every adapter contains all the necessary logic and data to update/change the state of the DAO in the DAORegistry Contract. The Core Contract tracks all the state changes of the DAO, and an Adapter tracks only the state changes in its own context. Extensions enhance the DAO capabilities and simplify the Core Contract code. The information always flows from the External World to the Core Contracts, never the other way around. If a Core Contract needs external info, it must be provided by an Adapter and/or an Extension instead of calling External World directly.

There are five main Venture in the Tribute architecture outlined further below.


### Core Contracts

The core contracts serve as the spine for the Venture DAO framework and act as a DAO registry, creating a digital version of "division of corporations." These contracts compose the DAO itself, and make it cheaper and easier to deploy a DAO. These contracts directly change the DAO state without the need of going through an adapter or extension (described further below). A core contract never pulls information directly from the external world. For that we use Adapters and Extensions, and the natural information flow is always from the external world to the core contracts.

There are three core contracts as part of the Venture DAO framework, including a:

- <code>DaoRegistry</code>: tracks the state changes of the DAO, only adapters with proper Access Flags can alter the DAO state.
- <code>CloneFactory</code>: creates a clone of the DAO based on its address.
- <code>DaoFactory</code>: creates, initializes, and adds adapter configurations to the new DAO, and uses the CloneFactory to reduce the DAO creation transaction costs.

### Adapters and Extensions
Once a DAO is created using the above core contracts, they can be extended and modified with adapters and extensions. Adapters and extensions make it easy to assemble a DAO like lego blocks, by adding to a DAO narrowly-defined, tested, and extensible smart contracts created for specific purposes. Adapters and extensions make DAOs more modular, upgradeable, and also enable us to work together to build robust DAO tooling. They can be added to a VentureDAO via a DAO vote.

#### Adapters
Creating an adapter is straight forward and should save developers engineering time. Each adapter needs to be configured with the Access Flags in order to access the Core Contracts, and/or Extensions. Otherwise the Adapter will not able to pull/push information to/from the DAO.

- <code>DistributeFund</code>: let a project rasing funds from the DAO through a voting process.
- <code>GPKick</code>: Kick out a GP member of the DAO through a voting process.
- <code>GPDaoOnboarding</code>: register a new GP member through a voting process.
- <code>Managing</code>: enhances the DAO capabilities by adding/updating the DAO Adapters through a voting process.

Please note:
- Adapters do not keep track of the state of the DAO. An adapter might use storage to control its own state, but ideally any DAO state change must be propagated to the DAORegistry Core Contract.
- Adapters just execute smart contract logic that changes the state of the DAO by calling the DAORegistry. They also can compose complex calls that interact with External World, other Adapters or even Extensions, to pull/push additional information.
- The adapter must follow the rules defined by the Template Adapter.
If you want to contribute and create an Adapter, please checkout this: How to create an Adapter.

#### Extensions
Extensions are conceived to isolate the complexity of state changes from the DAORegistry contract, and to simplify the core logic. Essentially an Extension is similar to an Adapter, but the main difference is that it is used by several adapters and by the DAORegistry - which end up enhancing the DAO capabilities and the state management without cluttering the DAO core contract.

- <code>FundingPool</code>: A pool to manages the funds.
- <code>GPDao</code>: to manages GP members.

#### Access Control Layer
The Access Control Layer (ACL) is implemented using Access Flags to indicate which permissions an adapter must have in order to access and modify the DAO state. The are 3 main categories of Access Flags:
- <code>MemberFlag</code>: EXISTS.
- <code>ProposalFlag</code>: EXISTS, SPONSORED, PROCESSED.
- <code>AclFlag</code>: REPLACE_ADAPTER, SUBMIT_PROPOSAL, UPDATE_DELEGATE_KEY, SET_CONFIGURATION, ADD_EXTENSION, REMOVE_EXTENSION, NEW_MEMBER.

The Access Flags of each adapter must be provided to the DAOFactory when the daoFactory.addAdapters function is called passing the new adapters. These flags will grant the access to the DAORegistry contract, and the same process must be done to grant the access of each Adapter to each Extension (function daoFactory.configureExtension).

The Access Flags are defined in the DAORegistry using the modifier hasAccess. For example, a function with the modifier hasAccess(this, AclFlag.REPLACE_ADAPTER) means the adapter calling this function needs to have the Access Flag REPLACE_ADAPTER enabled, otherwise the call will revert. In order to create an Adapter with the proper Access Flags one needs to first map out all the functions that the Adapter will be calling in the DAORegistry and Extensions, and provide these Access Flags using the DAO Factory as described above.

You can find more information about the purpose of each access flag at DAO Registry - Access Flags.