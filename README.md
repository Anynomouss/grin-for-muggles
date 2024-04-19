[WARNING THIS IS WORK IN PROGRESS AND MIGHT CONTAIN WRONG INFORMATION]  
# Grin for muggles
This documentation aims to explain Grin and mimblewimble from a user viewpoint.
The outline of the document follows a 'User Story'  by answering common questions from a user. This document serves as an Add-On to the [official documentation](https://docs.grin.mw/wiki/table-of-contents/https://docs.grin.mw/wiki/table-of-contents/). Where possible I will simplify while at the same time being **explicit about the information flow between [users] - [nodes] - [wallets]**. 
This documentation is self serving, and represents my own struggles as a muggle, visual thinker, programmer and aspiring wizard, to understand Grin. Since I have a background in Bitcoin, I will make many comparisons with Bitcoin; highlighting where Grin is different from Bitcoin. I will in this document speak about Grin. Note that grin is a (the purest) *mimblewimble* implementation. Therefore ***this document describes mimblewimle but more specifically describes its  implementation in Grin.***.

**Outline of this document:**

 1) How does my Grin wallet work?
 2) how does my wallet create a transaction?
 3) How does my transaction end up on chain?
 4) How does my wallet knows which outputs belong to it? 
 
If this explanation does not work for you and you prefer a purely conceptual high level description, I highly recommend reading [this explanation by Phyro](https://phyro.github.io/what-is-grin/), it helped me a lot. Reading about Grin from multiple angles helped me greatly on my journey to understand Grin. If your brain gets tired while reading, simply skip forward to read the "SUMMARY" at the end of each section.<br/> <br/>

***

# How does my Grin wallet work?
Grin wallets are generated the same way as most Crypto wallets. Grin follows the BIP32 standard in deriving a master seed from a mnemonic seed phrase (BIP39) and deriving children keys as a [Hierarchically Deterministic (HD)](https://learnmeabitcoin.com/technical/keys/hd-wallets/derivation-paths/https://learnmeabitcoin.com/technical/keys/hd-wallets/derivation-paths/https://learnmeabitcoin.com/technical/keys/hd-wallets/derivation-paths/https://learnmeabitcoin.com/technical/keys/hd-wallets/derivation-paths/) wallet. The process for deriving BIP32 HD wallets is that you start with a randomly generated number called a (**seed**), which is presented to the users as a list of words called the **mnemonic seed phrase** (12-24 words). From this seed phrase a master key as well as children keys are generated. HD wallets can be best understood by visualizing them as a tree:

        BIP32:
        Master  / Account/ Purpose / index                           Derivation_path
       
                              / --3  (spend)                         /m/0/0/3
                             / ---2  (spend)                         /m/0/0/2
                            0 ----1  (spend)                         /m/0/0/1
                           /    
    seed  -> m/ ---- 0 ---- |
                           \    
                            1 ---- 1 (change)                        /m/0/1/1
                             \ --- 2 (change)                        /m/0/1/2
                              \ -- 3 (change)                        /m/0/1/3
                    
 

  
The most important thing to understand about your Grin wallet is that all keys are Hierarchically Deterministic (HD). This means that as long as you have the **seed(phrase)**, you can restore your wallet and derive the complete tree of derived keys. Losing funds after a recovery from the seed phrase is therefore impossible! Your wallet seed is stored on your computer in a file called ***"wallet.seed"***.
> **WARNING:**  
*Grin-wallet* stores the seed itself. This means that the mnemonic seed phrase should not be reused for other wallets.<br/><br/>


**Is my Grin wallet different from my Bitcoin wallet?**  
There are two important differences when using your Grin wallet compared to Bitcoin. These differences have to do with how Grin transaction data is stored. First, grin nodes can freely "forget" transaction data from transactions that are spend. This is great for keeping the blockchain lightweight by pruning unneeded data, but it also means your wallet cannot recover information about spend transactions from the blockchain. Second, Grin is amazing for privacy since all outputs in a transaction can be aggregated as 1) transaction (Dandelion, MWixnet)  2) in a block and 3) as blockchain as a whole. This means that when you restore your wallet, even your own wallet cannot retrieve the link between inputs and outputs in a block. Your wallet can only scan the blockchain to see which unspent transaction outputs (UTXO's) are yours since spend TXO's forgotten/not-stored. Since you only need to know about outputs that are unspent, your wallet balance is always correct, even without knowing the full transaction history.

Secondly, Grin does not store all transaction information on chain. Payment proofs are created interactively between you and the party you interact. Not storing Payment proofs on-chain keeps Grin lightweight and privacy preserving. The downside is that your wallet will not be able to restore payment proofs when recovering your wallet from the seed phrase. As merchant who would use Grin, it would be wise to make the occasional backup of your *wallet_data* folder.


> **SUMMARY:**
Your Grin wallet is just a normal BIP32 HD wallet. Your wallet seed is stored in a file called ***"wallet.seed"***. When you restore your wallet from the seed phrase, all funds will be recovered. However, you will not be able to see transactions of which the outputs were spend nor will you be able to restore payment proofs.

  
***

# How are transactions in Grin different from Bitcoin?
**Bitcoin transactions:**
In Bitcoin, the sender is in full control. The sender Signs the transaction data with a [digital signature](https://learnmeabitcoin.com/beginners/guide/digital-signatures/https://learnmeabitcoin.com/beginners/guide/digital-signatures/). Bitcoin addresses are either hashed public keys (legacy address) or a hash of a lock-script that includes your public key [[REF](https://learnmeabitcoin.com/technical/keys/address/https://learnmeabitcoin.com/technical/keys/address/)]. Whenever someone sends Bitcoin to your address, she transfers ownership of the Unspent Transaction Outputs (UTXO's) in that transaction by saying *"Anyone who can provide the lock-script together with a valid signature for this address, can spend all outputs associated to it"*. Since only you can provide a signature for your public key/address, only you can spend your coins. All miners can verify that your signature matches the lock script, after which they will include your transaction in the next block to be mined. Amounts and addresses are transparent, meaning anyone can exactly see how much Bitcoin is send and to who's address.

**Grin transactions:**
Grin transactions have two main differences from Bitcoin transactions. 
First, in Grin transactions both **users are in full control of their own inputs and outputs**. To receive or spend an output, you have to 'Sign' for that output using your wallet keys. This means that to send a transaction you have to interact with the receiver since both your and the receivers wallet need to provide information for your own inputs and outputs ([See this explanation](https://phyro.github.io/what-is-grin/interactive_txs.htmlhttps://phyro.github.io/what-is-grin/interactive_txs.html)) **Sender and receiver jointly create a Signature** for the transaction.   
Second, in grin transactions, output values are hidden and not bound to an address! Yep, that's right, Grin has no addresses like Bitcoin nor user accounts like Ethereum.
In the next section I will attempt to explain in more detail how Grin is different from Bitcoin and how Grin achieves a) hiding amounts, b) not needing addresses, c) making transactions easy to aggregate.

**Grin transactions involve a couple of cool tricks**:    
1) **Homomorphic Commitments**: In Grin, values are hidden by multiplying all values with a Generator point. Although the result of multiplying a value with a generator point results in a seemingly random number. Addition and subtraction still work. 
If a transaction would involve one input value and two output values `v1+v2-v3=0`, they can be multiplied with generator point `H` on an [Elliptic Curve](https://docs.grin.mw/wiki/introduction/mimblewimble/ecc/https://docs.grin.mw/wiki/introduction/mimblewimble/ecc/) and anyone can verify they still sum up to zero `v2*H + v3*H - v1*H = 0` without knowing what value `v1`, `v2` or `v3` is. 

2) **Pedersen Commitments**: As explained above, Grin transactions hide the amount in an output by multiplying them with a generator point. However, this would allow people to find `v1`, `v2` or `v3` by guessing them. To solve this problem, a second trick is applied by adding a private-key multiplied with another generator point `G` to the output commitment... if you were wondering, yes multiplying a key with a generator point a public key just like in Bitcoin! This effectively makes it impossible to know the value in an output:<br/>
`Output = K1*G+v*H`. To an outsider an output commitment looks like a random piece of data : `09551fd2ba097bbf53d027c9820c2f19a544a15f21cb46614ce5860077a3663181`.<br/><br/>  

3) **Range proofs:** There is one problem in Grin, that is: we have to make sure outputs do not have a negative value. Otherwise anyone could just create a large negative output and another positive output to create Grin out of thin air!  
To solve this problem Grin uses range-proofs, to be precise [Bulletproofs](https://tlu.tarilabs.com/cryptography/bulletproofs-and-mimblewimblehttps://tlu.tarilabs.com/cryptography/bulletproofs-and-mimblewimble). Range-proofs are the most 'bulky' part of a Grin transaction 674 bytes. Of these 674 bytes, 64 bytes are used to ***XOR*** the ***amount*** and ***derivation index*** into the range proof. Your wallets needs the range-proof associated to an output to spend it and to know the value inside. Output commitments themselves cannot be decomposed because of the Elliptic Curve Discrete Logarithm Problem (ECDLP). Therefore your wallet also 'forgets' spend outputs since the range-proof tells what the value inside is and what key to include when calculating the kernel excess (sum of the excess of all keys used to blind each transaction output).

4) **Joint Schnor signatures:** Hold on, if we add keys `K1*G` and `K2*G` to the output commitments, the outputs will not add up to zero right? Indeed, there is an excess that consists of the sum of all private-keys used to blind outputs, minus the sum of all private-keys used to blind the inputs. Note that therefore in Grin we often talk about blinding factors, which are in practice private-keys. By adding all private-keys times generator point G, we create an `excess` per output that is summed to get the **'kernel excess'**. By proving the sender and receiver jointly know the excess (with is just another point on the curve since its a multiple of `G`), we prove we know all private keys: 
`sum(outputs) - sum(inputs) = kernel_excess` 


**An example for aspiring wizards :mage:**<br/> 
Now lets say you make a transaction with one input, one output for the receiver and one change output (ignoring fees):<br/><br/>
`C1 = K1*G + v1*H`  
`C2 = K2*G + v2*H`
`C3 = K3*G + v2*G` <br/><br/>
How can you prove to an outsider you actually know that as sender you know private keys `K1` and `K2` and the receiving party knows `K3`? Remember that addition using EC generator points still works, meaning that we can combine these three commitments `C1`, `C2` and `C3` in a new commitment `Z`, by adding them. Also remember that all value add up to zero even when multiplied by `H`! 

The point `Z` (remember a commitment is simply a point on the curve) is the result of addition between points C1 and C2.<br/><br/>
`Z = C1 + C2 + C3`  
`Y - Xi = (K2+K3-K1)*G +(v2+v3-v1)*H = excess*G + 0*H`<br/> 
Where `Y - Xi` is a valid public key for generator point G; which is the case only if `Y - Xi = r*G + 0*H`. In other words, if the values don't sum to 0, the result is recognized as an invalid public key for G. This ensures that:

*  The transacting parties can collectively produce the excess value (it is the private key of their **joint signature**).
* **The sum of the outputs minus the inputs is 0**, because only a valid public key for G will check out against the joined signature generated by the public key access.
<br/><br/>

This is the foundation for the Elliptic-curve algebra used to **prove both ownership of outputs (coins) and non-inflation****.

The excess value of a transaction is the sum of all outputs blinding factors, minus the sum of all inputs blinding factors, `sum(outputs) - sum(inputs) = kernel_excess`.


> **SUMMARY:**    
Grin transactions are interactive, each party signs for their own outputs and jointly they generate a signature to prove that a) all values add up to zero, and b) that they know all private-keys by creating a signature for the private-key excess. Grin ***output commitment*** are both ***binding*** and ***hiding***. Any node can verify the outputs in a transaction, in a block or in the entire block-chain without knowing any of the output values.   
Range-proofs (bulletproofs) contain the information needed for your wallet to spend an output and to proof the value is non-negative. Therefore after you spend an output, all nodes will forget the output range-proof since it is no longer relevant. Nodes can even forget about spend outputs since they occur both as input and output they are not needed to validate the blockchain state is correct.
*** 


# How does my transaction end up on chain?
Grin transaction are broadcasted to the peers your node is connects to. Since the node does not use the Tor network, it means that your peers know that you send a transaction as well as your IP address. To protect you from this privacy leak, *grin-wallet* by default uses Dandelion to help protect your anonymity [REF] [https://docs.grin.mw/wiki/miscellaneous/dandelion/]. 
Remember that Grin transactions can be freely aggregated. Therefore transactions are merged when they are put in a block.
Since there are no addresses or otherwise identifying information in transactions, there is very little you can learn by analyzing the Grin blockchain. The more transactions are aggregated, the harder it will be to guess which inputs and outputs are linked. Simply put, unless an observer had access to transaction data before aggregating, he or she does not know which inputs or outputs are linked. That is pretty neat trick to improve the anonymity for all transactions.  

There is however another very important aspect of "aggregating" that we have to discuss. Any output that is spend occurs both as input as well as output with the same key, meaning its value is identical. This means you can simply forget the output ever existed!. **Grin only remembers unspent transaction outputs**, spend outputs are simply forgotten. This is great for saving blockchain space as well as for privacy. To put this to the test, make a backup of your **wallet_data** folder, delete the original and  try restoring your wallet from the seed. You  will find even your own wallet will have forgotten transactions for which the outputs were spend.

> **TAKE AWAY:**
Transactions are Homomorphic, transactions can be freely be aggregated at the level of a) a transactions, b) blocks, c) blockchain. A node treats the Grin blockchain as one large transaction! To verify this "large transaction" is true, you node verifies:   
`Σ utxo = Σ kernel + height * 60 * H`   
The above simple equation proves that no new coins are created apart from the block reward as well as the validity of all transactions. This trick of "forgetting" spend outputs is called *[cut-through](https://docs.grin.mw/wiki/introduction/(og)introduction-to-mimblewimble/#cut-throughhttps://docs.grin.mw/wiki/introduction/(og)introduction-to-mimblewimble/#cut-through)*.



# How does my wallet knows which outputs belong to it?  
Output commitments themselves cannot be decomposed by your wallet. Output commitments are binding and hiding and provide **proof** of **ownership** and **non-inflation**. The real information about outputs is retrieved from their associate**range-proof**. Range-proofs are XOR'ed with a *nonce* and *index* to indicate which wallet key were used. Since your wallet can scan for range-proof that are  XORed with your wallet keys, it scan for outputs and decode the *amount* and *key index* for from the range-proof. The wallet now knows the *amount* and *key index* it retrieved from the range-proof of outputs the wallet wants to use. The wallet can use these to calculate new outputs and the transaction kernel excess jointly with the receiver. The jointly calculated excess of private keys is called the **transaction kernel** and only is a valid valid public key for G if all values in the transactions add up to zero. Hence the kernel excess is both a signature to prove ownership of the keys as well as proving no new coins are create through the transaction (non-inflation)!.  [[REF](https://tlu.tarilabs.com/cryptography/bulletproofs-and-mimblewimblehttps://tlu.tarilabs.com/cryptography/bulletproofs-and-mimblewimble)]

# Mind boggling realizations :exploding_head:
* Mimblewimble is a new blockchain format
* Grin has no addresses or amounts on chain
* Grin / mimblewimble gets **free privacy** and **free scalability** through its **interactive** nature :magic_wand:.
* Interactivity is a **benefit**, not a cost :bulb:.
* Grin nodes treat all transactions as **a single transaction**:exploding_head:
* Grin transactions can be aggregated at any level!
    ```
    transaction: sum(outputs) - sum(inputs) = kernel_excess  
    block:       sum(outputs) - sum(inputs) = sum(kernel_excess)  
    blockchain:  sum(outputs) - sum(inputs) = sum(kernel_excess) + height*60*H  
    ```
* Proving a) **non-inflation** b) **ownership** is as simple as checking: 
  `Σ utxo = Σ kernel + height * 60 * H` 
* A Grin transaction consist of a) a **single transaction kernel** b) a *range proof* per output and c) a **public fee**.
* A spend Grin transaction only leaves the transaction kernel on chain. Range-proofs are 'forgotten'. This means that for a typical transaction of 1 input and two outputs that contain `~2 KB` of transaction data, only `~100 bytes` are left when you spend a transaction. 
Historic Grin transactions  `~100` bytes which is still 2/3 the size of a Bitcoin taproot transaction `~150`. 
* Grins code is minimal, around **~13% the size of Bitcoin**,`136216` versus `877341` lines of code.
* Grin is minimal, elegant digital cash and to be honest, even ore interesting than Bitcoin for me. Check it out yourself :pill: :rabbit: :hole:

![Grin historic transaction size](grin_historic_transaction_size.png)


***
# Acknowledgment
Big thank you to all the big minds that came up with mimblewimble and Grin and all those who wrote documentation or answered questions that helped me to understand Grin. To all readers, I wish you all well on your journey down the Grin rabbit hole, I am certain you will like what you find there.

# References
https://github.com/mimblewimble/docs
https://docs.grin.mw/wiki/introduction/mimblewimble/mimblewimble/
https://docs.grin.mw/wiki/table-of-contents/
https://phyro.github.io/what-is-grin/interactive_txs.html
https://phyro.github.io/what-is-grin/mimblewimble.html
https://medium.com/@brandonarvanaghi/grin-transactions-explained-step-by-step-fdceb905a853
https://tlu.tarilabs.com/protocols/mimblewimble-transactions-explained
https://tlu.tarilabs.com/cryptography/bulletproofs-and-mimblewimble
