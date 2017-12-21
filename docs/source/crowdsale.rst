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
<https://brave.com/>`_ browser successfully reached their hard cap. You can find a list of complete/incomplete ICOs on `ICOdrops 
<https://icodrops.com/category/ended-ico/>`_.


Soft cap is the minimal amount required by your project, to make it viable, in order to continue. If you do not reach that amount during your ICO then you should allow your investors to refund their money using a push/ pull mechanism.

A simple implementation of capping functionality in smart contracts is portrayed in the following contract, where `Crowdsale <https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/crowdsale/Crowdsale.sol>`_ is a standard contract of zeppelin solidity: ::

    contract CappedCrowdsale is Crowdsale {
        using SafeMath for uint256;

        uint256 public cap;

        function CappedCrowdsale(uint256 _cap) public {
            require(_cap > 0);
            cap = _cap;
        }

        // overriding Crowdsale#hasEnded to add cap logic
        // @return true if crowdsale event has ended
        function hasEnded() public constant returns (bool) {
            bool capReached = weiRaised >= cap;
            return super.hasEnded() || capReached;
        }

        // overriding Crowdsale#validPurchase to add extra cap logic
        // @return true if investors can buy at the moment
        function validPurchase() internal returns (bool) {
            bool withinCap = weiRaised.add(msg.value) <= cap;
            return super.validPurchase() && withinCap;
        }
    }

Refundable
----------

In case the soft cap decided at the beginning of your crowdsale is not reached, you need to refund your investor's money. Open-zeppelin does it by creating a refund vault, which stores money while the crowdsale is in progress. It refunds money if the crowdsale fails, otherwise forwards it to the crowdsale wallet if successful. You can see the sample code `here <https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/crowdsale/RefundableCrowdsale.sol>`_.


Refund to the contributor is done using a pull mechanism, while keeping the Reentrancy problem into consideration to avoid attacks, from malicious contributors.

Finalizable
-----------

Finalizable contract is derived from Open-Zeppelin, it allows owners to add logic of extra work once the crowdsale is finished. It is always used once the crowdsale is finished to add some extra finalization work to the contract. It has an internal ``finalization`` function which can be overridden to add finalization logic. The overriding function should call ``super.finalization()`` to ensure the chain of finalization is executed entirely. You can find a simple implementation by zeppelin solidity below, though it could be used for much more. ::

    function finalize() onlyOwner public {
        require(!isFinalized);
        require(hasEnded());

        finalization();
        Finalized();

        isFinalized = true;
    }

    function finalization() internal {}


Destructible
------------

A crowdsale with a Destructible option is usually advised against, especially since the `Parity Hack <https://medium.com/chain-cloud-company-blog/parity-multisig-hack-again-b46771eaa838>`_. A better alternative to transfer your funds after crowdsale end is finalizable option discussed above. But still if your functionality demands it, Tokeny can create a crowdsale for you which will destruct on its end and will transfer all its funds to owner wallet. It should contain following two functions to achieve this: ::

    function destroy() public onlyOwner {
        selfdestruct(owner);
    }

    function destroyAndSend(address _recipient) public onlyOwner {
        selfdestruct(_recipient);
    }

It uses ``selfdestruct`` function provided by zeppelin-solidity, which will destroy the contract deployed on the ethereum blockchain. It can also be used to delete previous versions of your contracts from ethereum blockchain. Here ``destroy`` will remove the crowdsale contract from ethereum blockchain and will transfer its funds to ``owner`` address. While, ``destroyAndSend`` will transfer everything to the ``_recipient``, on owner's consent.

Extendable
----------

Crowdsale proprietors will have the capacity to change start and end time of their crowdsale, however it is advisable only when milestone strategy is not being utilized. `Bancor <https://www.bancor.network/>`_ used this feature in their crowdsale contracts for proper control over crowdsale parameters. A simple implementation will look as follows: ::

    function setStartTime(uint256 _newStartTime) public onlyOwner {
        require(startTime > now);
        require(_newStartTime > now);
        require(_newStartTime < endTime);
        startTime = _newStartTime;
    }

    function setEndTime(uint256 _newEndTime) public onlyOwner {
        require(endTime > now);
        require(_newEndTime > now);
        require(_newEndTime > startTime);
        endTime = _newEndTime;
    }

Here ``startTime`` and ``endTime`` are currently set timestamps, governing crowdsale start and end.

KYC Based
---------

In order to comply with regulators, you should have a KYC proof strategy in place for your crowdsale. You can have different strategies to implement this feature, whitelisting is a commonly used solution. Although, it could be inconvenient, as most of the crowdsale organizers are not registered before a crowdsale begins. Following contract can be used as an alternative solution, where `CappedCrowdsale <https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/crowdsale/CappedCrowdsale.sol>`_, `RefundableCrowdsale <https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/crowdsale/RefundableCrowdsale.sol>`_ are standard zeppelin-solidity contracts: ::

    contract KYCBasedCrowdsale is CappedCrowdsale, RefundableCrowdsale {
        mapping (address => uint)  public reserveTokens;
        mapping (address => bool) public kycDone;

        event Register(address sender, address beneficiary, uint tokens);

        modifier kycComplete(address _beneficiary) {
            require(_beneficiary != 0x0);
            require(kycDone[_beneficiary]);
            _;
        }
        
        function KYCBasedCrowdsale (
            uint256 _startTime,
            uint256 _endTime,
            uint256 _rate,
            uint256 _goal,
            uint256 _cap,
            address _wallet,
            address _token
        )   public
            CappedCrowdsale(_cap)
            FinalizableCrowdsale()
            RefundableCrowdsale(_goal)
            Crowdsale(_startTime, _endTime, _rate, _wallet, _token)
        {
            //As goal needs to be met for a successful crowdsale
            //the value needs to less or equal than a cap which is limit for accepted funds
            require(_goal <= _cap);
        }

        function buyTokens(address _beneficiary) public payable {
            require(_beneficiary != 0x0);
            require(validPurchase());

            uint256 weiAmount = msg.value;

            // calculate token amount to be created
            uint256 tokens = weiAmount.mul(rate);

            // update state
            weiRaised = weiRaised.add(weiAmount);

            if (kycDone[_beneficiary]) token.mint(_beneficiary, tokens);
            else register(_beneficiary, tokens);
            
            TokenPurchase(msg.sender, _beneficiary, weiAmount, tokens);
            forwardFunds();
        }
        
        function confirmKYC(address _beneficiary) public onlyOwner {
            require(_beneficiary != 0x0);
            kycDone[_beneficiary] = true;
        }

        function allocateTokens(address _beneficiary) public onlyOwner kycComplete(_beneficiary) {
            uint tokens = reserveTokens[_beneficiary];
            reserveTokens[_beneficiary] = 0;
            token.mint(_beneficiary, tokens);
        }

        function register(address _beneficiary, uint _tokens) internal {
            reserveTokens[_beneficiary] = reserveTokens[_beneficiary].add(_tokens);
            Register(msg.sender, _beneficiary, _tokens);
        }
    }

We have introduced the following two mappings- ``reserveTokens``, ``kycDone``; and three functions- ``confirmKYC``, ``allocateTokens`` and ``register`` in our KYC smart contracts to solve this problem more efficiently. It will have the following workflows:

**Contributor completed their KYC before buying tokens during crowdsale:** The oracle service will update the mapping ``kycDone`` against contributors address to true. While buying we will check the value of ``kycDone``, if it holds true the tokens will be assigned to the contributor instantly.

**Contributor's KYC process is incomplete and he buys tokens:** When the contributor sends ethers to the crowdsale, value of ``kycDone`` will be false, so no tokens will be assigned to him, but they will be registered, using ``register`` function, inside mapping ``reserveTokens``. When contributor completes his KYC process oracle will call two functions one after the other, namely ``confirmKYC`` (setting ``kycDone`` to true for contributor) and ``allocateTokens`` (distributing tokens to contributor).

*A refund option can be added for contributors in case they cannot complete KYC uptill crowdsale end.*

Milestone Based
---------------

Milestones are points of reference or timeframes amid which your crowdsale will be active, although they can be separated by gaps between them. Every milestone can have different properties, for example, its own token swap rate, cap and so forth.

A simple implementation of Milestone strategy will be as follows: ::

    contract MilestoneStrategy {
        using SafeMath for uint;

        uint public constant MAX_MILESTONE = 10;

        // How many active milestones we have
        uint public milestoneCount;

        /**
        * Define pricing schedule using milestones.
        */
        struct Milestone {
            // UNIX timestamp when this milestone kicks in
            uint time;

            // Milestone index
            uint index;

            // How many tokens per ether you will get after this milestone has been passed
            uint price;

            //how many wei can be raised     between two milestones
            uint weiCap;
        }

        // Store milestones in a fixed array, so that it can be seen in a blockchain explorer
        Milestone[10] public milestones;

        Milestone previousMilestone;
        Milestone currentMilestone;
        
        modifier onlyAfterFirstMilestone {
            require(now > milestones[0].time.sub(1));
            _;
        }

        /// @dev Contruction, creating a list of milestones
        /// @param _milestones uint[] milestones Pairs of (time, price, weiCap)
        function MilestoneStrategy(uint[] _milestones) public {
            // Need to have tuples, length check
            assert(_milestones.length <= MAX_MILESTONE*3);
            assert(_milestones.length%3 == 0);

            milestoneCount = _milestones.length / 3;

            // uint is initialized by compiler with 0
            uint lastTimestamp;

            // uint is initialized by compiler with 0
            for (uint i; i < milestoneCount; ++i) {
                // No invalid steps
                assert(_milestones[i*3] > lastTimestamp);
                // price must be greater that or equal to 0
                assert(_milestones[i*3 + 1] >= 0);
                // weiCap must be greater that or equal to 0
                assert(_milestones[i*3 + 2] >= 0);

                milestones[i].index = i;
                milestones[i].time = _milestones[i*3];
                milestones[i].price = _milestones[(i*3) + 1];
                milestones[i].weiCap = _milestones[(i*3) + 2];

                lastTimestamp = milestones[i].time;
            }

            // Last milestone price and weiCap must be zero, terminating the crowdale
            assert(milestones[milestoneCount-1].price == 0);
            assert(milestones[milestoneCount-1].weiCap == 0);

            //set initialMilestone
            previousMilestone = milestones[0];
        }

        /// @dev Get the current price.
        /// @return The current price or 0 if we are outside milestone period
        function getCurrentPrice() public constant returns (uint result) {
            return getCurrentMilestone().price;
        }

        /// @dev Get the current weiCap.
        /// @return The current weiCap or 0 if we are outside milestone period
        function getCurrentWeiCap() public constant returns (uint result) {
            return getCurrentMilestone().weiCap;
        }

        function isNewMilestone() internal returns (bool) {
            currentMilestone = getCurrentMilestone();
            
            if (currentMilestone.index > previousMilestone.index) {
                previousMilestone = currentMilestone;
                return true;
            }
            return false;
        }

        /// @dev Get the current milestone or bail out if we are not in the milestone periods.
        /// @return {[type]} [description]
        function getCurrentMilestone()
            internal
            constant
            onlyAfterFirstMilestone
            returns (Milestone)
        {
            for (uint8 i; i < milestoneCount; i++) {
                if (now < milestones[i].time) 
                    return milestones[i-1];
            }
        }
    }

Owner Assigned
--------------

Should not be used unless necessary, because it promotes centralization, as crowdsale owner will be able to allocate tokens at will. But it can be very useful during bug bounty programs and share distribution. This is one tradeoff you should consider before using Owner assigned features. though most crowdsales being implemented these days are already using it. It can also help you to accept multiple alt coins like Litecoin, Dash and Ripple during your crowdsale. `BrickBlock <https://www.brickblock.io/>`_ and `Bancor <https://www.bancor.network/>`_ are examples of ICOs who used this feature.

Here is an example of ``assignTokens``, ``confirmTokens`` and ``cancelTokens`` functions which can be used to implement this strategy in a simple manner: ::

    function assignTokens(
        address _beneficiary,
        uint _weiAmount,
        uint _indexOfTransaction
    )   public
        onlyOwner
        whenNotPaused
        returns (bool)
    {
        require(_beneficiary != 0x0);
        require(now >= startTime && now <= endTime);

        uint tokens = _weiAmount.mul(rate);

        // update state
        weiRaised = weiRaised.add(_weiAmount);

        return TokenyTestToken(token).assign(_beneficiary, tokens, _indexOfTransaction);
        TokenPurchase(msg.sender, _beneficiary, _weiAmount, tokens);
    }

    function confirmTokens(address _beneficiary, uint _indexOfTransaction)
        public
        onlyOwner
        whenNotPaused
        returns (bool)
    {
        require(endTime + 7 days > now); //should confirm within 7 days
        return TokenyTestToken(token).confirm(_beneficiary, _indexOfTransaction);
    }

    // weiAmount must be checked at backend to equal actual contribution
    function cancelTokens(address _beneficiary, uint _weiAmount, uint _indexOfTransaction)
        public
        onlyOwner
        whenNotPaused
        returns (bool)
    {
        require(endTime + 7 days > now); //should cancel within 7 days

        // update state
        weiRaised = weiRaised.sub(_weiAmount);
        return TokenyTestToken(token).cancel(_beneficiary, _indexOfTransaction);
    }

Here ``TokenyTestToken`` is a token contract with functions ``assign``, ``cancel`` and ``confirm``. modifiers ``whenNotPaused`` and ``onlyOwner`` ensure that crowdsale is not paused and only owner is able to access these functions, they also use `SafeMath <https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/math/SafeMath.sol>`_ library to avoid integer underflow/overflow.
Assigning and confirming logic is separated to deal with cancellation of bank payments. Suppose someone sends the crowdsale owner 150000 USD and asks for tokens, then owner can assign him tokens telling that his contribution has been noted in the blockchain, when the contribution reaches in his account (which could take upto 7 days) he can confirm user's contribution by distributing him tokens, otherwise he could cancel his contribution.

Pausable
--------

Crowdsale owners will be able to stop investors from buying tokens, in case of malicious attack on their platform or some bug report. After resolving these problems the crowdsale could be continued. It has become a very popular functionality and should be included in every smart contract as `recommended by ConsenSys <https://consensys.github.io/smart-contract-best-practices/software_engineering/#circuit-breakers-pause-contract-functionality>`_. To do this we need to simply override a ``buyTokens``  function and add a when not paused modifier to it. Simplified functions will look as follows: ::

    bool public paused = false;

    /**
    * @dev Modifier to make a function callable only when the contract is not paused.
    */
    modifier whenNotPaused() {
        require(!paused);
        _;
    }

    // Override buyTokens of Crowdsale
    function buyTokens(address _beneficiary) public payable whenNotPaused {
        super.buyTokens(_beneficiary);
    } 

Upgradable
----------

This feature solves a major problem of crowdsale owners- *How to handle bug fixes?* Code should be changed if mistakes are found or if upgrades should be made. It is useless to find a bug, yet have no real way to manage it. 

Outlining a viable upgrade framework for smart contracts is a region of active research, and we won't be able to cover everything in this document. Be that as it may, there are two essential methodologies that are most normally utilized. The easier of the two is to have a registry contract that holds the address of the most recent variant of the contract. A more consistent approach for contract clients is to have a contract that advances calls and information onto the most recent form of the contract. 

Whatever the procedure, it's essential to have modularization and good separation between code components, with the goal that code changes don't break usefulness, orphan data, or require substantial expenses to port. Specifically, it is generally gainful to isolate complex rationale from your data storage, with the goal that you don't need to reproduce the majority of the information keeping in mind the end goal to change the usefulness. 

It's also critical to host a secure way for parties to choose to update the code. Dependent upon your contract, code changes may need to be approved by a single trusted party, a group of members, or a vote of the full set of stakeholders. If this process can take some time, you will want to consider if there are other ways to react more quickly in case of an attack, such as an emergency stop or circuit-breaker.
