Crowdsale Features
==================

This document focuses on some recent adoptions becoming popular in the blockchain community while writing smart contracts for ICOs.


In use in many successful ICOs they should result in a higher probability to succeed. However I would advise you to only adopt features which are essential to your ICO. As more logic can be used in smart contracts to implement more security features like circuit breakers, upgrades, multisig etc. Although, the more logic in the smart contract the bigger the attack surface and the more likely that bugs are introduced, that risk undermining the security features.

*Notice that this document is not describing the features your ICO contracts should have to provide a utility value to your Contributors.*

Capped
------

**Min cap** (also known as Soft cap) is the minimum financial goal of a crowdsale. In Layman’s terms, it’s the minimum amount at startup, which is deemed sufficient to carry on with a project.


**Max cap** (also known as Hard cap) is the maximum desired financial goal from an ICO. It’s the amount that the project needs to develop or improve their product. The team usually believes this amount will cover the successful development of the project until the moment it becomes profitable.


Still most projects set a very high cap that is unlikely to be achieved. Only a few famous projects like `Status 
<https://status.im/>`_ or `Brave 
<https://brave.com/>`_ browser successfully reached their hard cap. You can find a list of complete/uncomplete ICOs on `ICOdrops 
<https://icodrops.com/category/ended-ico/>`_.


Soft cap is the minimal amount required by your project, to make it viable, in order to continue. If you do not reach that amount during your ICO then you should allow your investors to refund their money using a push/ pull mechanism.

Refundable
----------

In case the soft cap decided at the beginning of your crowdsale is not reached, you need to refund your investor's money. Open-zeppelin does it by creating a refund vault, which stores money while the crowdsale is in progress. It refunds money if the crowdsale fails, otherwise forwards it if successful.


This is done using a pull mechanism, while keeping the Re-Entrancy problem into consideration to avoid attacks, from malicious contributors.

Finalizable
-----------

Finalizable contract is derived from Open-Zeppelin, it allows owners to add logic of extra work once the crowdsale is finished. It is always used once the crowdsale is finished to add some extra finalization work to the contract. It has an internal finalization function which can be overridden to add finalization logic. The overriding function should call super.finalization() to ensure the chain of finalization is executed entirely.

Destructible
------------

A crowdsale with a Destructible option is usually advised against, especially since the `Parity Hack <https://medium.com/chain-cloud-company-blog/parity-multisig-hack-again-b46771eaa838>`_. A better alternative to transfer your funds after crowdsale end is finalizable option discussed above. But still if your functionality demands it, Tokeny can create a crowdsale for you which will destruct on it's end and will transfer all it funds to owner wallet.

Extendable
----------

Crowdsale proprietors will have the capacity to change start and end time of their crowdsale, however it is advisable only when milestone strategy is not being utilized. `Bancor <https://www.bancor.network/>`_ used this feature in their crowdsale contracts for proper control over crowdsale parameters.

KYC Based
---------

In order to comply with `SEC's <https://www.sec.gov/>`_ recommendations, you should have a KYC proof strategy in place for your crowdsale. You can have different strategies to implement this feature, whitelisting is a commonly used solution. Although, it could be inconvenient, as most of the crowdsale organizers are not registered before acrowdsale begins.

We can introduce the following two mappings (reserveTokens, kycDone) and three functions (confirmKYC, allocateTokens and register) in our KYC smart contracts to solve this problem more efficiently. It will have the following workflows:

**Contributor completed their KYC before buying tokens during crowdsale:** The oracle service will update the mapping kycDone against contributors address to true. While buying the we will check the value of mapping kycDone and the workflow will proceed, as it would have  normally.

**Contributor's KYC process is incomplete and he buys tokens:** When the contributor sends ethers to the crowdsale, value of kycDone will be false, so no tokens will be assigned to him, but they will be registered(using register function) inside mapping reserveTokens. When contributor completes his kyc process oracle will call two functions one after the other, namely confirmKYC (setting kycDone to true for contributor) and allocateTokens (distributing tokens to contributor).

*A refund option can be added for contributors in case they cannot complete KYC uptill crowdsale end.*

Milestone Based
---------------

Milestones are points of reference or timeframes amid which your crowdsale will be active, although they can be separated by gaps between them. Every milestone can have different properties, for example, its own token swap rate, cap and so forth.

Owner Assigned
--------------

Should not be used unless necessary, because it promotes centralization, as crowdsale owner will be able to allocate tokens at will. But it can be very useful during bug bounty programs and share distribution. This is one tradeoff you should consider before using Owner asiigned features. though most crowdsales being implemented these days are already using it. It can also help you to accept multiple alt coins like Litecoin, Dash and Ripple during your crowdsale. `BrickBlock <https://www.brickblock.io/>`_ and `Bancor <https://www.bancor.network/>`_ are examples of ICOs who used this feature.


Pausable
--------

Crowdsale owners will be able to stop investors from buying tokens, in case of malicious attack on their platform or some bug report. After resolving these problems the crowdsale could be continued. It has become a very popular functionality and should be included in every smart contract as `recommended by ConsenSys <https://consensys.github.io/smart-contract-best-practices/software_engineering/#circuit-breakers-pause-contract-functionality>`_.

Upgradable
----------

This feature solves a major problem of crowdsale owners- *How to handle bug fixes?* Code should be changed if mistakes are found or if upgrades should be made. It is useless to find a bug, yet have no real way to manage it. 

Outlining a viable upgrade framework for smart contracts is a region of active research, and we won't be able to cover everything in this document. Be that as it may, there are two essential methodologies that are most normally utilized. The easier of the two is to have a registry contract that holds the address of the most recent variant of the contract. A more consistent approach for contract clients is to have a contract that advances calls and information onto the most recent form of the contract. 

Whatever the procedure, it's essential to have modularization and good separation between code components, with the goal that code changes don't break usefulness, orphan data, or require substantial expenses to port. Specifically, it is generally gainful to isolate complex rationale from your data storage, with the goal that you don't need to reproduce the majority of the information keeping in mind the end goal to change the usefulness. 

It's also critical to host a secure way for parties to choose to update the code. Dependent upon your contract, code changes may need to be approved by a single trusted party, a group of members, or a vote of the full set of stakeholders. If this process can take some time, you will want to consider if there are other ways to react more quickly in case of an attack, such as an emergency stop or circuit-breaker.
