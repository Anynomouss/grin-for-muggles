>**Disclaimer:** <br/>This is docucement explores a **non-interactive user experience** for Grin transactions using a **relay buddy system**. This method is not proven, created for fun, and the author does advocate or endorse this proposal. I hope the reader will enjoy reading this proposal and will appreciate it as a (to the authors knowledge) first example of using cut-through before broadcasting to achieve some benefits; skipping the last round of interaction. Note the trick with a temporary output can also be used to creating a 1 step interaction transactions without a relay being involved. If combined with an address book, this would create a transaction method with few disadvantages. However, the method would still require two kernels and would not be compatible with PayJoins that remove directionality in the transaction graph. Although the authors finds this idea entertaining enough to to share, I frimly believe the future of Grin is in its interactivity, e.g.  fully interactive uniform transactions using PayJoins and optional CoinSwap. This proposal only serves for eductiona/fun purposes.

# Discussion on Bulleting Boards systems
This proposal is a 6th option added in the [Discussion on Bulletin Board Systems on the grin forum.](https://forum.grin.mw/t/grin-bulletin-board-discussing-four-options-and-select-one-for-bounty/9822]). Note that Grin is by default interactive crypto, the discusion on Bulletin Board Systems does not aim to provide 1 step or non-interactive transactions, but aims to find/explore user friendly ways to buffer and facilitate interaction between users. This proposal simply aims to share one such possible solutions. After inception I red Marek's Slate Store proposal and recognize it to be fairly similar with the exception that the last round of communication is kipped [[REF](https://github.com/grinventions/ideas/blob/main/SlateStore.md)].

## Transaction Relay-buddy system
This document proposes a method which I call the transaction relay ***Buddy System*** to allow a users (**A, Alice**) to send a transaction via an intermediary (**B, Bob the Buddy**) to a receiver (**C, Charlie**). The transaction flow is **request-RSR-SRS** and requires 6 steps of interaction. Most notable, the last round of interaction with the Sender is skipped using a temporary output, creating a 'send and forget' kind of user experience. The objective is to faciliate transaction between Alice and Charlie while Alice is offline with no need for Alice to manually interfer in the transaction process. In other words, it should all be magic behind the scenes.

0) Alice contacts Charlie over tor, but Charlie is offline
1) Alice send a **request** to *Bob, the relay-buddy*.
2) Bob creates a payment request including his output with the **relay-fee (RSR)** and sends it to Alice.
3) Alice adds the outputs for Charlie using Charlies **known Pubkey(s)*** and creates her partial signature for all outputs [change-output,relay-fee,charlies-output]
4) Bob waits till Charlie comes online to continues the transaction. When Charlie comes online, Bob sends the transaction in SRS mode to Charlie
5) Charlie receives the transaction, signs for it - and combines it with a self spend where he replaces the output from Alice with a new outputs for himself and sends it back to Bob. Charlie signs both the initial RSR transaction and the new SRS transaction (two kernels) and sends the result to Bob.  
6) Bob finalizes the two transactions that are now aggregated by signing the second kernel and broadcasting the transctions to the network to earn his relay fee.

**Note**  
* *A publickly known pubkey is included in the additional address information of Charlie, Alice will use this PubKey to create the output for Charlie and provide her partical signature for the excess value. She does however not have Charlie's PrivKey, so she cannot provide Charlies signature.
* The main invention here is in step 5, using cut-through to replace an undesired output since it contains Charlies known PubKey. The magic here is that the output can be  dropped thanks to **cut-through** and therefore will be known to no one except Alice, Bob and Charlie. What is achieved by using this temporary output is that Alice and Charlie never need to interact.
* Since the transaction contains outputs from both Alice,Bob and Charlie, none of the parties can cheat, since they do not know the PrivKey's to sign the excess value with from the other party.
* This idea [originates from 2021](https://forum.grin.mw/t/an-open-discussion-on-non-interactive-transactions/8510/48?u=anynomoushttps://forum.grin.mw/t/an-open-discussion-on-non-interactive-transactions/8510/48?u=anynomous) and is **a first example of how using cut-through before broadcasting a transaction can be usefull**. 
* The relay buddy system requires additional public transfer information to be presented by the receiver, e.g. a wrapper around the slatepack address. See more details later in this document on *Transaction transfer information*.

***Advantages:***
* Generic solution: Does not reuse public keys and only the intermediary (Bob) can deduce Alice and Charlie interacted.
* Decentralised, any wallet can be a relay with minimal overhead or extra code since in fact the relay is just two normal transactions aggregated.
* No new cryptography is needed, no side chain, no extra layers, no merged mining, little complexity.
* Relayed transactions are indistinguishable from a normal transaction and as such are not identifiable as being relayed.

***Disadvantages:***
* **Major disadvantage: :man_facepalming:** Bob know the PubKey from Charlie, and can bruteforce/deduce the value send by Alice to Charlie. He can in theory Dox the value to outsiders by undoing cut-through and publishing the output to the mempool, so anyone can bruteforce find the value there if they have Charlies PubKey. This is unlikely, there is no incentive to do so for Bob, but it still kind of ruins the magic of this trick.
* The relay is not private (opposed to the [Contract Wall](https://gist.github.com/phyro/1046022377fcb1886a1b4f6500f23773)" solution by Phyro)  
* The **relay-buddy system** can requires an extra transaction relay wrapper information to be implemented or alternatively, something like an address book with a share PubKeys from friends. Note that a friends-book would allow the receiver to share a specific PubKey for each friend, meaning Bob would not be able to deduce the value or Dox it to the mempool.
* For payment proofs to work, both kernels need to be published. I think there is no way to cheat here, again since the transaction contains outputs for all parties, no one can cheat and the only way to self spend the temporary output to Charlie is by publishing the kernel for the first transaction. 
* It is unclear how robust this system can be, it is untested, purely hypothetical and unproven. Moreover, their mind be more logical solutions, for example, mobile wallet with Nostr integration, or integration with any other decentralised buffered messaging protocol.

  
As mentioned above, the **transaction relay-buddy** system, requires additional information, namely a known PubKey for the receiver. There are two logical solutions: 

1) An **address-book** that contains information for the sender such as [name - address - [buddy tor addres, PubKey for relay]]. I will provide an example of such information in the next section
2) A **transaction transfer information system**. The transaction transfer information is a wrapper with additional information besides the slatepack address of the receiver. In this wrapper, the Receiver puts information such as the preferred methods/protocols to be used to receive a transaction. The methods can simply be provided in the order of preference, e.g. first try method 1, if that fails, method 2, etc.. One of the methods that is put in this wrapper is the information to contact the relay-buddy. When the receiver is offline, the Sender can use the information to contact the relay-buddy, who will handle the transaction for the receiver

# Transaction transfer information:
Example of how transfer protocol information could potentially look like as JSON-LD. Information can be presented as QR code to the user without needing to worry about space restrictions [[REF](https://www.qrcode.com/en/about/version.htmlhttps://www.qrcode.com/en/about/version.html)].<br/>
`[{`  <br/>
`"method-1":{"address": grin......},`<br/>
`"method-2": {"relay": grin42.....,relay-PubKey},`<br/>
`"method-3":{"email":"tomelvisjedusor@protonmail.com"}`<br/>
`}]`<br/>

## Insentives
For this to work it is essential that:
* A relay buddy gets a fee. I firmly belive any system only works with a proper insentive distribution.
* Sender builds outputs for the Receiver so the transaction contains an output for both Sender, Relay/intermediary and the Receiver
* The relay-buddy does need to broadcast both kernels for payment-proofs to work. 


## Security considerations
* The use of a known/public to all PubKey from Charlie is undesired. Using an address book would solve this issue. 
* Not sure yet how to do payment proof with 3 parties (according to Tromp ok as long as both kernels are published)
* Intermediare/relay can deny having build a transaction, but would not gain any funds, nor will funds be lost.
* Intermediary knows the address of the Receiver, optionally use a second tor address for this, not the slatepac address for extra privacy.
* Someone can pretend to be the receiver comming online, but cannot steal funds since he/she does not have the pubkey to get the `value` and `index` from the rangeproof, needed to spend that output. Nor can any messages be decrypted by that user since he does not have the privatekey for that tor address.
* Receiver must drop outputs build by the sender, replacing it with an equal value output with a non-fixed public-key, so basically replacing it with a normal transaction
* There must some time involved before a transaction is dropped, or in case of a trusted buddy, perhaps a message being send to the Receiver to go online to receive funds.
* How about spam to an intermediary. Can Alice and Charlie collude to spamm Bob, but Charlie will never finalize any transaction? Should not be a problem since each transaction requires new outputs to be on-chain. This should serve as a spam protection. Additionally, a maximum amount of transactions can be set by bob to be stored for his receiver friend.
* Can Bob screw over Charlie by not performing cut-through and publishing his outputs that do contain the public-keys online? I think yes, what would be the disinsentive? Should Charlie scan and detect this? Bob sees multiple outputs with the same blinding factor/pub-key being re-used.   
* Both Alice and Bob need access to the same pubkey information.
* Can a random attacker anoy Bob enough to stop him from relaying transactions for Charlie? Yes, in theory, but Bob can simply put a maximum to the number of transaction to accept from a single Sender.


## References
Bulletin Board System discussion on Grin forum: https://forum.grin.mw/t/grin-bulletin-board-discussing-four-options-and-select-one-for-bounty/9822/14
<br/>
Transfer protocol: https://forum.grin.mw/t/what-is-the-most-critical-problem-of-grin/9424/62<br/>
Contract wall: https://gist.github.com/phyro/1046022377fcb1886a1b4f6500f23773
Simple explenation transact: https://forum.grin.mw/t/eliminating-finalize-step/7621 <br/>
Payment proof Kurt: https://forum.grin.mw/t/eliminating-finalize-step/7621/22<br/>
https://forum.grin.mw/t/eliminating-finalize-step/7621/76?u=anynomous<br/>
https://github.com/mimblewimble/grin-rfcs/pull/59#issuecomment-679970246<br/>
https://medium.com/blockstream/insecure-shortcuts-in-musig-2ad0d38a97da<br/>
Asynchronous transactions e.g. Grinbox https://github.com/mimblewimble/grin-rfcs/pull/25<br/>
https://gist.github.com/DavidBurkett/32e33835b03f9101666690b7d6185203<br/>
Discussion on KeyBas: keybase://chat/grincoin#general/63744<br/>
