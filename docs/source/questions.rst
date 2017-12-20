Questions for ICO Organizers
============================

Your idea is worth millions of dollars, and an ICO might provide you with a platform to 
raise them. But before you approach a solidity developer to realise your dream, you should
be prepared to ask yourself several questions, in order to have an efficient and healthy plan.
In many cases asking the right questions is a very important job, as it can reduce planning
time drastically and can help you proceed to the required solution. This document will help
you understand basic questions you must ask yourself before finalizing a crowdsale or a token
strategy. Their answers will shape the smart contracts and the way you’ll handle your investment
plan.

Crowdsaale Questions
--------------------

Your crowdsale would be handling people’s money, moreover it will be collecting money for
you. Therefore, you should plan properly before proceeding. We at Tokeny, are there to help
you through that:

* **When will your crowdsale begin?** This might sound like avery basic question but it is among the most important ones, and their are many factors you should consider before finalizing it:
    * *Time to establish your idea in the market.*
    * *Cryptocurrency market stability.*
    * *Status of your idea prototype.*
* **What should be the duration of your crowdsale?** Another important question reflecting your preparation, and available investors. It should be a duration apt for you, to raise an amount which will be sufficient to proceed with your project.
* **Do you want the duration of your crowsale to be dependent on timestamp of a block or block number?** In ethereum blockchain the essence of a block is realized by tracking the latest block. Therefore crowdsale organizers can set the duration using either block timestamp or block number.Although, we would recommend using block timestamps for its proximity with human dates.
* **Do you want a presale?** It can be used to attract special investors, who show early trust in your idea and are rewarded with subsidized token swap rates. If yes, answer the following: 
    * *When will it begin?*
    * *What will be it’s duration?*
    * *What are the special conditions, for instance- bonus, minimum amount, etc.*
* **Do you want your crowdsale to be divided into milestones?** Milestones are points of reference or timeframes amid which your crowdsale will be active, although they can be separated by gaps between them. Every milestone can have different properties, for example, its own token swap rate, cap and so forth. You ought to likewise consider the accompanying inquiries below.
    * *What determines a milestones price intervals, bonus intervals?*
    * *How many milestones do you need?*
    * *When will each milestone start?*
    * *What will be the duration of each milestone?*
    * *Will there be a cap for each milestone or an overall cap?*
* **Do you want your crowdsale token swap rate to be based on fiat currencies (USD/EUR) or ether (ETH)?** We recommend ether because crowdsale ether to fiat exchange rates are subjected to change during the duration of a crowdsale.
* **Do you want a Cap?** A Cap helps you choose when to stop and when to surrender. It likewise regulates control and demonstrates that you're no tightwad, at last building a relationship of trust with your customers. This is one of those inquiries that you should reply with a "yes". There are different inquiries related with capping you ought to consider questions mentioned below.
    * *What should be your minimum ether/fiat cap?*
    * *What should be your maximum ether/fiat cap?*
    * *Do you want to accept multiple contributions from an investor or do you want a limit on number of investments from a single investor?*
    * *Do you want an individual min/max cap for each investor? If yes, how much?*
    * *Do you need a fixed token supply (max token cap) or you want them to be created every time someone buys your tokens, until your ether/fiat cap is reached? If latter is your choice, just decide the token swap rate considering the max ether/fiat cap and the duration of your crowdsale (also include, presale and milestone strategy, if any). Otherwise, answer the questions below.*
        * *What should be the total supply of your tokens? (Or max token cap)*
        * *What should be the swap rate of your tokens? Decide this on the basis of maximum ether/fiat cap and total supply of your tokens.*
        * *If required total supply is not reached during the crowdsale, i.e. some tokens are left even after crowdsale ends, what should happen to them? Should they be sent to some reserve or they’re to be distributed among crowdsale owners. Another strategy is Airdrop, where left over tokens will be distributed among token holders according to their fair share after crowdsale.*
* **What will be your refund policy if cap is not reached?** Usually ethers are transferred to a vault contract for safe keeping until min/max cap is reached. In the event that cap isn't achieved contributor's can go to the crowdsale contract and claim their tokens utilizing a pull mechanism. ICO organizers can likewise select a separate procedure as indicated by their necessities.
* **Would new tokens be available to buy after crowdsale?** This can be useful in cases where you have a fixed total supply and you were unable to sell all of them during your crowdsale. But f the limit has already been exhausted, it is not a good idea to do so.
* **Do you want a referral policy during your crowdsale?** Your contributors will have a referral code, when somebody joins on your platform utilizing this referral code, the referrer will have the capacity to claim reward tokens. It has the accompanying inquiries related with it.
    * *What should be the percentage of tokens assigned for referrals?*
    * *How many referrals should be allowed for a single user?*
* **Do you want your crowdsale to have a KYC logic implemented before your crowdsale begins or you want your contributors to complete their KYC before they can begin transferring their tokens?** Non KYC contributors won’t be able to participate in your crowdsale. Special care should be taken to comply with industry norms in this case.
* **Do you need an owner assigned token mechanism for your crowdsale?** Should not be used unless necessary, because it promotes centralization, as crowdsale owner will be able to allocate tokens at will. But it can be very useful during bug bounty programs and share distribution. this is one tradeoff you should consider before using.
* **Any other special functionality for your crowdsale?** There may be some special functionality of your crowdsale that could be unique in itself, you can't leave anything. So consider it assiduously.
* **What will be your initial share distribution scheme?** Have you saved a few tokens/ethers for venture proprietors and diverse groups related with your projects? This functionality should be a part of crowdsale contracts and therefore should be known in advance.  
* **How do you vision the ether to be handled after the crowdsale?** 
    * *Straightaway transfer to a private account.*
    * *Handle using a multisig on the blockchain.*
* **Owner of the crowdsale should be single account or multisig?** Multisig or multisignature require another client or clients to sign an transaction before it can be communicated onto the blockchain, this is a decent practice and is prescribed over single account usage.

Token Questions
---------------

Once you’re done planning your crowdsale, you will need to start thinking about your tokens, here are some basic questions to help you do so.

* **What is the name of your token?** It ought to be something appealing, however more imperatively it ought to be one of a kind. Because of the expanding number of crowdsales you should watch that your token name isn't as of now being used.
* **What is the symbol for your token?** A token symbol is generally a three letter word derived from the token name itself, eg BTC from Bitcoin. Though it is not a rigid rule and the imperative thing about it, similar to token name is its uniqueness.
* **How many decimal points you want to track for your tokens?** Typically favoured value is 18, since ether has 18 decimal units. Be that as it may, it can totally rely upon the cost of your token and which sub units, ought to be accessible to exchange later.
* **You want to allow only high level purchase, low level purchase or both?**
	* *High level purchase: Only the one who sends ether to the contract will be able to buy tokens.*
	* *Low level purchase: Investors will be able to buy tokens for some other account, sending ethers on their behalf.*
* **Do you want token transfer and other basic ERC20 functions to be active during the crowdsale?** If your answer is no, they will remain inactive until the crowdsale is finished or some other time/block limit is reached.
* **Your token should be ERC20 or ERC223?** ERC20 is the widely popular standard for tokens, which helps in buying selling and trading them. ERC223 is built on top of ERC20 and provides the following advantages mentioned below.
	* *Eliminates the problem of lost tokens which happens during the transfer of ERC20 tokens to a contract (when people mistakenly use the instructions for sending tokens to a wallet). ERC223 allows users to send their tokens to either wallet or contract with the same function transfer, thereby eliminating the potential for confusion and lost tokens.*
	* *Allows developers to handle incoming token transactions, and reject non-supported tokens (not possible with ERC20).*
	* *Energy savings- The transfer of ERC223 tokens to a contract is a one step process rather than two steps process (for ERC20), and this means two times less gas and no extra blockchain bloating.*
* **Any other special functionality of tokens?** Tokens can be utilized for some different procedures like voting, betting and so on. It relies upon your prerequisite that what else your token would do.
* **What will be the vesting scheme of your tokens?** Would you like to release every one of your tokens at once or would you like to discharge them slowly, at different timestamps? Assume you release 40 percent amid presale and 60 percent amid your crowdsale. You can likewise utilize this methodology on token holders and discharge their tokens in a vested way.
