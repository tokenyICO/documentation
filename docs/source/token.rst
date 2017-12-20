Token Features
==============

Tokens are the currency within your ecosystem. A proper set of functionalities in your token is therefore needed to make it work efficiently. This document proposes some basic features you should consider to have in your token contracts.

As more logic can be used in token contracts to implement more security features like timeouts, spending limits, multisig, vaults etc. Although, the more logic in the smart contract the bigger the attack surface and the more likely that bugs are introduced, that risk undermining the security features.

*Notice that this document is not describing the features your ICO tokens should have to provide a utility value to your Contributors.*

ERC20
-----

The ERC-20 defines a common list of rules for all Ethereum tokens to follow, meaning that this particular token empowers developers of all types to accurately predict how new tokens will function within the larger Ethereum system. The impact that ERC-20 therefore has on developers is massive, as projects do not need to be redone each time a new token is released. Rather, they are designed to be compatible with new tokens, provided those tokens adhere to the rules. Developers of new tokens have by-and-large observed the ERC-20 rules, meaning that most of the tokens released through Ethereum initial coin offerings are ERC-20 compliant.

ERC223
------

ERC223 is built on top of ERC20 and provides the following advantages mentioned below.

Eliminates the problem of lost tokens which happens during the transfer of ERC20 tokens to a contract (when people mistakenly use the instructions for sending tokens to a wallet). ERC223 allows users to send their tokens to either wallet or contract with the same function transfer, thereby eliminating the potential for confusion and lost tokens.

Allows developers to handle incoming token transactions, and reject non-supported tokens (not possible with ERC20).

Energy savings- The transfer of ERC223 tokens to a contract is a one step process rather than two steps process (for ERC20), and this means two times less gas and no extra blockchain bloating.


Upgradable
----------

Allows future modifications in your token contracts. Can be useful in case of changing requirements, or for bug fixes.

Owner Assigned
--------------

Owner will be able to mint and assign tokens to accounts even after the crowdsale ends.

Pausable
--------

Basic ERC20 features like transfer, transferFrom and approve can be paused by owner. It is useful in case of bug fixes.

Burnable
--------
 
On the basis of token functionality owners or users can be allowed to burn their tokens using this feature.
