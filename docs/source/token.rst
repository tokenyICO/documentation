Token Features
==============

Tokens are the currency within your ecosystem. A proper set of functionalities in your token is therefore needed to make it work efficiently. This document proposes some basic features you should consider to have in your token contracts.

As more logic can be used in token contracts to implement more security features like timeouts, spending limits, multisig, vaults etc. Although, the more logic in the smart contract the bigger the attack surface and the more likely that bugs are introduced, that risk undermining the security features.

*Notice that this document is not describing the features your ICO tokens should have to provide a utility value to your Contributors.*

ERC20
-----

The ERC-20 defines a common list of rules for all Ethereum tokens to follow, meaning that this particular token empowers developers of all types to accurately predict how new tokens will function within the larger Ethereum system. The impact that ERC-20 therefore has on developers is massive, as projects do not need to be redone each time a new token is released. Rather, they are designed to be compatible with new tokens, provided those tokens adhere to the rules. Developers of new tokens have by-and-large observed the ERC-20 rules, meaning that most of the tokens released through Ethereum initial coin offerings are ERC-20 compliant.

Here's a basic interface describing function signatures of ERC20 contracts: ::

    contract ERC20 {
        uint256 public totalSupply;
        event Approval(address indexed owner, address indexed spender, uint256 value);
        event Transfer(address indexed from, address indexed to, uint256 value);
        function balanceOf(address who) public constant returns (uint256);
        function transfer(address to, uint256 value) public returns (bool);
        function allowance(address owner, address spender) public constant returns (uint256);
        function transferFrom(address from, address to, uint256 value) public returns (bool);
        function approve(address spender, uint256 value) public returns (bool);
    }

Unsigned integer type variable ``totalSupply``, is accessible to all, and it displays the number of tokens in circulation. Function ``balanceOf`` takes in ethereum addresses and tells you the number of tokens are held by that address. Function ``transfer`` is used to transfer tokens from one address to other. Lastly, functions ``allowance``, ``approve`` and ``transferFrom`` are used to authorize some other ethereum address to utilize your tokens on your behalf. This ethereum address could be a smart contract or just another account, ``approve`` is used to allow some ``spender`` to use a particular ``value`` of his tokens. Similarly, ``allowance`` can be used to check ``spender's`` allowance for a particular ``owner``, and ``transferFrom`` can be used by ``spender`` to transfer tokens.

ERC223
------

ERC223 is built on top of ERC20 and provides the following advantages mentioned below.

**Eliminates the problem of lost tokens** which happens during the transfer of ERC20 tokens to a contract (when people mistakenly use the instructions for sending tokens to a wallet). ERC223 allows users to send their tokens to either wallet or contract with the same function transfer, thereby eliminating the potential for confusion and lost tokens.

Allows developers to **handle incoming token transactions, and reject non-supported tokens** (not possible with ERC20).

**Energy savings**- The transfer of ERC223 tokens to a contract is a one step process rather than two steps process (for ERC20), and this means two times less gas and no extra blockchain bloating.

Here's a simple ERC223 implementation: ::

    contract ERC223Token {
        using SafeMath for uint;

        mapping(address => uint) internal balances; // List of user balances.
        event Transfer(address indexed from, address indexed to, uint value, bytes indexed data);
        
        /**
        * @dev Transfer the specified amount of tokens to the specified address.
        *      Invokes the `tokenFallback` function if the recipient is a contract.
        *      The token transfer fails if the recipient is a contract
        *      but does not implement the `tokenFallback` function
        *      or the fallback function to receive funds.
        *
        * @param _to    Receiver address.
        * @param _value Amount of tokens that will be transferred.
        * @param _data  Transaction metadata.
        */
        function transfer(address _to, uint _value, bytes _data) public {
            // Standard function transfer similar to ERC20 transfer with no _data .
            // Added due to backwards compatibility reasons .
            uint codeLength;

            assembly {
                // Retrieve the size of the code on target address, this needs assembly .
                codeLength := extcodesize(_to)
            }

            balances[msg.sender] = balances[msg.sender].sub(_value);
            balances[_to] = balances[_to].add(_value);
            if (codeLength > 0) {
                ERC223ReceivingContract receiver = ERC223ReceivingContract(_to);
                receiver.tokenFallback(msg.sender, _value, _data);
            }
            Transfer(msg.sender, _to, _value, _data);
        }
        
        /**
        * @dev Transfer the specified amount of tokens to the specified address.
        *      This function works the same with the previous one
        *      but doesn't contain `_data` param.
        *      Added due to backwards compatibility reasons.
        *
        * @param _to    Receiver address.
        * @param _value Amount of tokens that will be transferred.
        */
        function transfer(address _to, uint _value) public {
            uint codeLength;
            bytes memory empty;

            assembly {
                // Retrieve the size of the code on target address, this needs assembly .
                codeLength := extcodesize(_to)
            }

            balances[msg.sender] = balances[msg.sender].sub(_value);
            balances[_to] = balances[_to].add(_value);
            if (codeLength > 0) {
                ERC223ReceivingContract receiver = ERC223ReceivingContract(_to);
                receiver.tokenFallback(msg.sender, _value, empty);
            }
            Transfer(msg.sender, _to, _value, empty);
        }

        /**
        * @dev Returns balance of the `_owner`.
        *
        * @param _owner   The address whose balance will be returned.
        * @return balance Balance of the `_owner`.
        */
        function balanceOf(address _owner) public view returns (uint balance) {
            return balances[_owner];
        }
    }

Standard ``ERC223ReceivingContract`` must have a function tokenFallback, the sample interface would be like this: ::

    contract ERC223ReceivingContract { 
        /**
        * @dev Standard ERC223 function that will handle incoming token transfers.
        *
        * @param _from  Token sender address.
        * @param _value Amount of tokens.
        * @param _data  Transaction metadata.
        */
        function tokenFallback(address _from, uint _value, bytes _data) public;
    }

ERC223 checks for bytecode length of the address tokens are being transferred to, and if it is a contract, it looks for a function with the signature of ``tokenFallback``, hence solving the problem of tokens stuck in contracts.


Upgradable
----------

Allows future modifications in your token contracts. Can be useful in case of changing requirements, or for bug fixes.

Upgradable token contracts are divided into two types of contracts- ``UpgradeAgent`` and ``UpgradeToken``.

``UpgradeAgent`` contract is inspired by `Lunyr <https://lunyr.com/>`_. It is used to transfer tokens to a new contract. ``UpgradeAgent`` itself can be the token contract or a middleman contract doing the heavy lifting.

While ``UpgradeToken`` contract provides a token upgrade mechanism where the users can opt-in amount of tokens to the next smart contract revision. It has the the following upgrade states:

* **NotAllowed:** The child contract has not reached a condition where the upgrade can begin.
* **WaitingForAgent:** Token allows upgrade, but we don't have a new agent yet.
* **ReadyToUpgrade:** The agent is set, but not a single token has been upgraded yet.
* **Upgrading:** Upgrade agent is set and the balance holders can upgrade their tokens.

If any new version of the token contracts arrives, the UpgradeToken contract can be used by token holders to claim new version of tokens.

Although this is just one strategy in Tokeny's arsenal for handling upgrades in your token contracts. We can also use registry contracts to point to the latest version of the contract, and your contributors need not worry about a thing. 

Owner Assigned
--------------

Owner will be able to mint and assign tokens to ethereum based accounts. This functionality can be restricted according to required usage, in order to decrease centralization introduced due to its usage. This functionality should only be used when necessary. Although it can be used to assign tokens after the crowdsale end, and therefore has a very large usage set. You can handle bug bounties, large USD investments, share distribution etc.

Here's a very simple smart contract implementation, where `MintableToken <https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/token/MintableToken.sol>`_ is a famous zeppelin-solidity token: ::

    contract OwnerAssignedToken is MintableToken {
        event Canceled(address indexed canceledContributor, uint256 value);
        event Assigned(address indexed contributor, uint256 value);
        enum TransactionState { YET_TO_START, PENDING, CONFIRMED, CANCELED }

        struct Contribution {
            uint time;
            uint tokens;
            TransactionState transactionState;
        }

        mapping(address => mapping(uint => Contribution)) public contributions;

        modifier validRequest(Contribution contribution) {
            require(contribution.transactionState == TransactionState.PENDING);
            require((contribution.time + 7 days) > now && now >= contribution.time);
            _;
        }

        function cancel(address _contributorAddress, uint _indexOfTransaction)
            public
            onlyOwner
            validRequest(contributions[_contributorAddress][_indexOfTransaction])
            returns (bool)
        {
            require(_contributorAddress != 0x0);

            Contribution storage contribution = contributions[_contributorAddress][_indexOfTransaction];
            contribution.transactionState = TransactionState.CANCELED;

            Canceled(_contributorAddress, contribution.tokens);
            
            return true;
        }

        function assign(address _contributorAddress, uint _amount, uint _indexOfTransaction)
            public
            onlyOwner
            returns (bool)
        {
            require(_contributorAddress != 0x0);

            Contribution storage contribution = contributions[_contributorAddress][_indexOfTransaction];
            assert(contribution.transactionState == TransactionState.YET_TO_START);

            contribution.time = now;
            contribution.transactionState = TransactionState.PENDING;
            contribution.tokens = _amount;

            Assigned(_contributorAddress, _amount);

            return true;
        }

        function confirm(address _contributorAddress, uint _indexOfTransaction)
            public
            onlyOwner
            validRequest(contributions[_contributorAddress][_indexOfTransaction])
            returns (bool)
        {
            require(_contributorAddress != 0x0);

            Contribution storage contribution = contributions[_contributorAddress][_indexOfTransaction];

            contribution.transactionState = TransactionState.CONFIRMED;
            totalSupply = totalSupply.add(contribution.tokens);
            balances[_contributorAddress] = balances[_contributorAddress].add(contribution.tokens);
            
            Mint(_contributorAddress, contribution.tokens);
            Transfer(0x0, _contributorAddress, contribution.tokens);

            return true;
        }
    }

Pausable
--------

Basic ERC20 features like transfer, transferFrom and approve can be paused by owner. It is useful in case of bug fixes or attack on your ecosystem. It could be included in your project's emergency protocols to fight malicious attacks on your system. `Here <https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/token/PausableToken.sol>`_ you can find a simple implementation by zeppelin-solidity.

Burnable
--------
 
On the basis of token functionality owners or users can be allowed to burn their tokens using this feature. If misused it can prove to be harmful for your token ecosystem, therefore proper checks are always necessary while exploring this feature. A simple function using burn feature will be as follows: ::

    function burn(uint256 _value) public {
        require(_value > 0);
        require(_value <= balances[msg.sender]);
        // no need to require value <= totalSupply, since that would imply the
        // sender's balance is greater than the totalSupply, which *should* be an assertion failure

        address burner = msg.sender;
        balances[burner] = balances[burner].sub(_value);
        totalSupply = totalSupply.sub(_value);
        Burn(burner, _value);
    }

