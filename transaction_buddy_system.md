**Disclaimer:** <br/>This is docucement explores a **non-interactive user experience** for Grin transactions using a **relay buddy system**. This method is not proven, created for fun, and the author does advocates this solution.  

# Discussion on Bulleting Boards systems
This proposal can be seen as a 5th option in the [Discussion on Bulletin Board Systems on the grin forum.](https://forum.grin.mw/t/grin-bulletin-board-discussing-four-options-and-select-one-for-bounty/9822]). Note that Grin is by default interactive crypto, the discusion on Bulletin Board Systems does not aim to provide 1 step or non-interactive transactions, but aims to find/explore user friendly ways to buffer and facilitate interaction between users.

## Transaction Relay-buddy system
This document proposes a method which I call the transaction relay ***Buddy System*** to allow a users (**A, Alice**) to send a transaction via an intermediary (**B, Bob**) to another user (**C, Charlie**). The transaction flow is **request-RSR-SRS** and requires 6 steps of interaction. The objective is to faciliate transaction between Alice and Charlie while Alice is offline.

0) Alice contacts Charlie over tor, but Charlie is offline
1) Alice send a **request** to Bob, the intermediary/*relay-buddy*.
2) Bob creates a payment request **(RSR)** including his own output/**relay-fee**
3) Alice adds the outputs for Charlie using his known Pubkey(s) and creates her partial signature
4) Bob wiats till Charlie comes online to continues the transaction. When Charlie comes online, Bob sends the transaction in SRS mode to Charlie
5) Charlie receives the transaction - creates new outputs to replace the ones made by and signed for by Alice. These old outputs can be dropped thanks to **cut-through** so they do not need to appear on chain. Charlie signs both the innitial RSR transaction and the new SRS transaction (two kernesl) and sends the result to Bob.  
6) Bob finalizes the two transactions that are now aggregated by signing the second kernel and broadcast the transction with both kernels to earn his relay fee.

**Note**
* Bob cannot broadcast the transaction without Charlie, since he does not know the partial excess to sign for Charlies outputs.
* The outputs created by Alice for Charlie, will never appear on-chain thanks to cutt-through, unless Bob does include these outputs on purpose, in which case miners will will drop the intermediate outputs.
This idea [originates from 2021](https://forum.grin.mw/t/an-open-discussion-on-non-interactive-transactions/8510/48?u=anynomoushttps://forum.grin.mw/t/an-open-discussion-on-non-interactive-transactions/8510/48?u=anynomous). 
* The relay buddy system requires additional public transfer information to be presented by the receiver, e.g. a wrapper around the slatepack address. See more details later in this document on *Transaction transfer information*.

***Advantages:***
* Generic solution: Does not reuse public keys and only the intermediary (Bob) can deduce Alice and Charlie interacted.
* Decentralised, any wallet can be a relay with minimal overhead or extra code since in fact the relay is just two normal transactions aggregated.
* No new cryptography is needed, no side chain, no extra layers, no merged mining, little complexity.
* Relayed transactions are indistinguishable from a normal transaction and as such are not identifiable as being relayed.

***Disadvantages:***
* **Major disadvantage: :man_facepalming:** Who ever knows the PubKey from Charlie, can bruteforce/deduce the value send by Alice to Charlie.
* The relay is not private (opposed to the [Contract Wall](https://gist.github.com/phyro/1046022377fcb1886a1b4f6500f23773)" solution by Phyro)  
* The **relay-buddy system** can be decentralised, but does require extra transaction relay wrapper information to be implemented.
* For payment proofs to work, both kernels need to be published. This requires Bob to be trusted. Perhaps an extra proof needss to be generated between Alice and Bob, to make sure that Bob and Charlie cannot 'collude' by Bob not publishing the second kernel, allowing Charlie to deny having received funds.
* It is unclear how robust this system can be, it is untested, purely hypothetical and unproven

  
As mentioned above, the **transaction relay-buddy** system, requires the implementation of a `transaction transfer information system`. The transaction transfer information is a wrapper with additional information besides the slatepack address of the receiver. In this wrapper, the Receiver puts additional information such as the preferred methods in order of his/her preference to receive a transaction. One of these methods could be a relay buddy/intermediary. When the receiver is offline, the Sender can use the information in the `transaction transfer information` to send to the transaction intermediary. As said before, the intermediary can be a  buddy, central relay provider o a random wallet/node via tor but that last option is not considered for now since it requires more security cosiderations as well as decentralised way to determine which node to use ass buddy.

# Transaction transfer information:
Example of how transfer protocol information could potentially look like as JSON-LD. Information can be presented as QR code to the user without needing to worry about space restrictions [[REF](https://www.qrcode.com/en/about/version.htmlhttps://www.qrcode.com/en/about/version.html)].<br/>
`[{`  <br/>
`"method-1":{"address": grin......},`<br/>
`"method-2": {"relay": grin42.....,relay-PubKey},`<br/>
`"method-3":{"email":"tomelvisjedusor@protonmail.com"}`<br/>
`}]`<br/>

## Insentives
For this to work it is essential that:
* A relay buddy gets fee. Crypto only works with proper insentive distribution.
* Sender builds outputs for the Receiver so the Intermediary cannot create outputs for himself
* Relay-buddy does broadcast both kernels for payment-proofs to work. Again, we need some sort of extra insentive/proof to avoid a relay from cheating the sender.



## Security considerations
* The use of a known/public to all PubKey from Charlie is still a problem.
* Not sure yet how to do payment proof with 3 parties (according to Tromp ok as long as both kernels are published)
* Intermediare can deny having build a transaction, but would not gain any funds, nor will funds be lost.
* Intermediary knows the address of the Receiver, optionally use a second tor address for this, not the slatepac address for extra privacy.
* Someone can pretend to be the receiver comming online, but cannot steal funds since he/she does not have the pubkey to get the `value` and `index` from the rangeproof, needed to spend that output. Nor can any messages be decrypted by that user since he does not have the privatekey for that tor address.
* Receiver must drop outputs build by the sender, replacing it with an equal value output with a non-fixed public-key, so basically replacing it with a normal transaction
* There must some time involved before a transaction is dropped, or in case of a trusted buddy, perhaps a message being send to the Receiver to go online to receive funds.
* How about spam to an intermediary. Can Alice and Charlie collude to spamm Bob, but Charlie will never finalize any transaction? Should not be a problem since each transaction requires new outputs to be on-chain. This should serve as a spam protection. Additionally, a maximum amount of transactions can be set by bob to be stored for his receiver friend.
* Can Bob screw over Charlie by not performing cut-through and publishing his outputs that do contain the public-keys online? I think yes, what would be the des-insentive? Should charlie scan and detect this? Bob sees multiple outputs with the same blinding factor/pub-key being re-used.   
* Both Alice and Bob need access to the same pubkey information.
* Can a random attacker anoy Bob enough to stop him from relaying transactions for Charlie? Yes, in theory, but Bob can simply put a maximum to the number of transaction to accept from a single Sender.


## References
Bulletin Board System discussoin on Grin forum: https://forum.grin.mw/t/grin-bulletin-board-discussing-four-options-and-select-one-for-bounty/9822/14
<br/>
Transfer protocol: https://forum.grin.mw/t/what-is-the-most-critical-problem-of-grin/9424/62<br/>
Contract wall: https://gist.github.com/phyro/1046022377fcb1886a1b4f6500f23773
Simple explenation transact: https://forum.grin.mw/t/eliminating-finalize-step/7621 <br/>
Payment proof Kurt: https://forum.grin.mw/t/eliminating-finalize-step/7621/22<br/>
https://forum.grin.mw/t/eliminating-finalize-step/7621/76?u=anynomous<br/>
https://github.com/mimblewimble/grin-rfcs/pull/59#issuecomment-679970246<br/>
https://medium.com/blockstream/insecure-shortcuts-in-musig-2ad0d38a97da<br/>
Asynchrnous transactions e.g. Grinbox https://github.com/mimblewimble/grin-rfcs/pull/25<br/>
https://gist.github.com/DavidBurkett/32e33835b03f9101666690b7d6185203<br/>
Discussion on KeyBas: keybase://chat/grincoin#general/63744<br/>