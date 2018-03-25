# ChainGun - A conceptual blockchain network for gun ownership

There's been a lot of talk about gun violence and calls for action against it in social media and through protests such as __March For Our Lives__. Hearing about it on a regular basis, I couldn't help but wonder if there was something that could be done to support this cause. 

This project here is a manifestation of that desire and my personal interest in blockchain tech. Here, I conceptualize a theoretical blockchain, implemented in Python, that could be used to track gun ownership and transfer of possession. This follows an excellent tutorial listed on hackernoon by [Daniel van Flymen](https://hackernoon.com/learn-blockchains-by-building-one-117428612f46). 

## Summary

Most licensed firearms have a serial id. It is this ID that police and ATF officials use, to try and trace the manufacturer and purchase history of the gun. This entire project is based on the concept that each transaction can be encoded into a blockchain and hence, a distribute ledger, and that the transaction can be recorded by all nodes in the system.
Let me first define a few variables in this theoretical transaction system.
* Owner - Person/Organization with a firearm for sale: e.g. Acme Firearms Inc.
* Receiver - Person buying a firearm: e.g __John Doe__
* Amount - Monetary value paid by __John Doe__ for acquiring firearm from Owner
* Gun_ID - Serial ID of firearm being sold.

Names can be substituted with some form of ID number for anonymity e.g. Firearms License Card ID Number

Its quite self-explanatory. When John walks up to to Acme Inc. to buy himself a new pistol, he initiates a transaction. The transaction records all these details as one dictionary variable. Now, when John decides to sell his firearm, he becomes the owner and Jane Doe who's buying his pistol becomes the receiver. Another, transaction is added to the chain. 

In these two theoretical transactions, we observe that we can preserve information regarding the gun serial ID while still tracking the movement of the gun from one hand to another.

The ledger nature of a blockchain also allows for John, Jane and Acme to be simultaneously updated about the movement of the gun. Jane can trace the origins of the gun to Acme if necessary. 

## Packages Required

This project was done in Python3

* Requests
* Flask

I would also recommend using Postman over cURL for the HTTP client requests.

## Prerequisites

A simple of understanding of blockchain basics and python programming is enough to make to implement this project. I would recommend the following guides for more knowledge on blockchain tech and the underlying priciples like Proof of Work, Hashing Algorithms and Network Consensus.

* https://bitsonblocks.net/2015/09/09/a-gentle-introduction-to-blockchain-technology/
* https://mlsdev.com/blog/156-how-to-build-your-own-blockchain-architecture
* https://learncryptography.com/hash-functions/what-are-hash-functions
* https://www.youtube.com/watch?v=9mNgeTA13Gc

## ChainGun 

I will be explaining the crucial functions that make this program up

### Chain Validity

This functions seeks to test the validity of the chain

```python
class Chaingun:

...

def valid_chain(self, chain):
        last_block = chain[0]
        current_index = 1

        while current_index < len(chain):
            block = chain[current_index]
            print(f'{last_block}')
            print(f'{block}')
            print("\n-----------\n")
            # Check that the hash of the block is correct
            if block['previous_hash'] != self.hash(last_block):
                return False

            # Check that the Proof of Work is correct
            if not self.valid_proof(last_block['proof'], block['proof'], last_block['previous_hash']):
                return False

            last_block = block
            current_index += 1

        return True

...
```

Here __chain__ is the current blockchain and the __return__ variable is boolean true or false denoting if the chain passed the validity tests.

### Network Consensus

```python
def resolve_conflicts(self):
        neighbours = self.nodes
        new_chain = None

        # Look only for chains longer than current
        max_length = len(self.chain)

        # Find and verify the chains from all the nodes in network
        for node in neighbours:
            response = requests.get(f'http://{node}/chain')

            if response.status_code == 200:
                length = response.json()['length']
                chain = response.json()['chain']

                # Check to see if length is longer than current and the chain is valid
                if length > max_length and self.valid_chain(chain):
                    max_length = length
                    new_chain = chain

        # Replace our chain if we discovered a new, valid chain longer than ours
        if new_chain:
            self.chain = new_chain
            return True

        return False
```
This is the network consensus algorithm, it resolves conflicts by replacing our chain with the longest one in the network by checking all the other registered nodes.
Our logic here is that the longest chain in the network is the most valid chain.
We __return__ True if our chain was replaced, and False if not.

### Transaction and Hashing Function

The transaction function is next. This contains all the information for our transaction.

```python
def new_transaction(self, owner, receiver, amount, gun_id):
        self.current_transactions.append({
            'owner': owner,
            'receiver': receiver,
            'amount': amount,
            'gun_id':gun_id
            
        })

        return self.last_block['index'] + 1
```
Creates a new transaction thats queued for the next mined Block
* owner: ID of the current gun owner
* receiver: ID of the buyer 
* amount: Amount paid in the transaction
* gun_id: Unique gun id for the gun that's being transacted
* return: The index of the Block that will hold this transaction

The hashing function takes all of this information, orders and encodes it performing a standard SHA-256 hash on it.

```python
def hash(block):
        block_string = json.dumps(block, sort_keys=True).encode()
        return hashlib.sha256(block_string).hexdigest()
```

### Proof of Work and Validation of PofW
Every blockchain network needs to have a proper proof of work system for the functioning of the system. Every time a new node is mined, this proof of work algo is what gives legitimacy to it. This PofW algo could be any mathematical equation or problem.
Simple Proof of Work Algorithm:
- Find a number p' such that hash(pp') contains leading 4 zeroes
- Where p is the previous proof, and p' is the new proof
         

```python
def proof_of_work(self, last_block):
        last_proof = last_block['proof']
        last_hash = self.hash(last_block)
        proof = 0
        while self.valid_proof(last_proof, proof, last_hash) is False:
            proof += 1
        return proof

   @staticmethod
   def valid_proof(last_proof, proof, last_hash):
        guess = f'{last_proof}{proof}{last_hash}'.encode()
        guess_hash = hashlib.sha256(guess).hexdigest()
        return guess_hash[:4] == "0000"
```

## Demonstration

Let's try out how it works! Fire up Postman and do the following.

Initializing: Send a GET request to ```/mine```

<img src="https://user-images.githubusercontent.com/10093954/37879823-861c293c-304c-11e8-946b-7221d57973f8.png" alt="Drawing" style="width:100%;"/>


Make a new transaction using POST and filling out the transaction details in JSON. Keep owner as the hash id of receiver in previous block ```/transactions/new```. Mine again. You can add transactions and play around with this.

<img src="https://user-images.githubusercontent.com/10093954/37879822-8611d3d8-304c-11e8-8593-dadbcd8899b0.png" alt="Drawing" style="width:100%;"/>


Check out the addition in the chain using ```/chain```

<img src="https://user-images.githubusercontent.com/10093954/37879819-85f04916-304c-11e8-8681-af039f719d66.png" alt="Drawing" style="width:100%;"/>

Change port number from 5000 to 5001 and run another instance of ```chaingun.py```. Then in a POST request on your old node, register your new node. 

<img src="https://user-images.githubusercontent.com/10093954/37879818-85e62990-304c-11e8-9a14-96be1b35dcac.png" alt="Drawing" style="width:100%;"/>


Add Some transactions to your new node

<img src="https://user-images.githubusercontent.com/10093954/37879871-71dbc918-304d-11e8-8113-b0dd450c8e32.png" alt="Drawing" style="width:100%;"/>

Try merging using ```nodes/resolve``` and see what you get!

<img src="https://user-images.githubusercontent.com/10093954/37879817-85dc3dae-304c-11e8-96f9-e2bb891ae001.png" alt="Drawing" style="width:100%;"/>




## Conclusion

Obviously, this is not even close to the actual complexity of a real block-chain network like Ethereum. But it was fun learning in depth into blockchain systems and to come up with a simple network that demostrated it. Play around with it on your own systems.

## Common Issues

I found that sometimes there were other processes that were still running and occupying ports in my system.
Run this on your CLI ```ps -fA | grep python```
This will list the processes running on the port.

You can kill them by entering the process number ```kill <processnumber>```. Run chaingun.py after that.

## Acknowledgments

* https://hackernoon.com/learn-blockchains-by-building-one-117428612f46

## License

This project is licensed under the MIT License. See [LICENSE.md](LICENSE.md) for more information.



