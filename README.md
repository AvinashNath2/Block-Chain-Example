# Block-Chain-Example
Simple example of blockchain using flask in python.

## Prerequisites
- Python 3.0+
- Flask and Requests 
```
# for install
pip install Flask==0.12.2 requests==2.18.4
```
### Major steps
1. Implementing a basic proof of work
2. Making our blockchain an API endpoint
3. Creating a miner for your blockchain
4. Interacting with the blockchain


### Step 1 - Implementing a basic proof of work
Understanding Proof of Work

A Proof of Work algorithm (PoW) is how new Blocks are created or mined on the blockchain.

The goal of PoW is to discover a number which solves a problem. The number must be difficult to find but easy to verify for anyone on the network. The number is made of cryptographic signatures, meaning that if the wrong number is sent somewhere it will be denied access. Thus the PoW algorithm means you can now send money without having to trust anybody or any organisation, as the blockchain only cares about the cryptographic signatures. This is the core idea behind Proof of Work.

<p align="center"> 
<img src="https://d2omlh28jsv4r7.cloudfront.net/wp-content/uploads/2017/03/infographics2017-01.png">
</p

**But bitcoin work on proof of work.**

```
from hashlib import sha256
x = 10
y = 0  # We don't know what y should be yet...
while sha256(f'{x*y}'.encode()).hexdigest()[-1] != "0":
    y += 1
print(f'The solution is y = {y}')

 @staticmethod
    def hash(block: Dict[str, Any]) -> str:
        """
        Creates a SHA-256 hash of a Block
        :param block: Block
        """

        # We must make sure that the Dictionary is Ordered, or we'll have inconsistent hashes
        block_string = json.dumps(block, sort_keys=True).encode()
        return hashlib.sha256(block_string).hexdigest()
        
            def proof_of_work(self, last_proof: int) -> int:
        """
        Simple Proof of Work Algorithm:
         - Find a number 0' such that hash(00') contains leading 4 zeroes, where 0 is the previous 0'
         - 0 is the previous proof, and 0' is the new proof
        """

        proof = 0
        while self.valid_proof(last_proof, proof) is False:
            proof += 1

        return proof

    @staticmethod
    def valid_proof(last_proof: int, proof: int) -> bool:
        """
        Validates the Proof
        :param last_proof: Previous Proof
        :param proof: Current Proof
        :return: True if correct, False if not.
        """

        guess = f'{last_proof}{proof}'.encode()
        guess_hash = hashlib.sha256(guess).hexdigest()
        return guess_hash[:4] == "0000"

```


### Step 2 - Making our blockchain an API endpoint
Add the below code to the bottom of your Python file. This code turns your file into an API end point. This will allow us to use postman to send/receive requests in our blockchain.

```
class Blockchain(object):

# Instantiate our Node
app = Flask(__name__)

# Generate a globally unique address for this node
node_identifier = str(uuid4()).replace('-', '')

# Instantiate the Blockchain
blockchain = Blockchain()


@app.route('/mine', methods=['GET'])
def mine():
    return "We'll mine a new Block"

@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    return "We'll add a new transaction"

@app.route('/chain', methods=['GET'])
def full_chain():
    response = {
        'chain': blockchain.chain,
        'length': len(blockchain.chain),
    }
    return jsonify(response), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)


@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    values = request.get_json()

    # Check that the required fields are in the POST'ed data
    required = ['sender', 'recipient', 'amount']
    if not all(k in values for k in required):
        return 'Missing values', 400

    # Create a new Transaction
    index = blockchain.new_transaction(values['sender'], values['recipient'], values['amount'])

    response = {'message': f'Transaction will be added to Block {index}'}
    return jsonify(response), 201
```

### Step 3 - Creating a miner for your blockchain
The below code will create a miner for your server, This will mine and create a new transaction to be placed on a block in your blockchain.

Add this code, below your new transaction code.

```
@app.route('/mine', methods=['GET'])
def mine():
    # We run the proof of work algorithm to get the next proof...
    last_block = blockchain.last_block
    last_proof = last_block['proof']
    proof = blockchain.proof_of_work(last_proof)

    # We must receive a reward for finding the proof.
    # The sender is "0" to signify that this node has mined a new coin.
    blockchain.new_transaction(
        sender="0",
        recipient=node_identifier,
        amount=1,
    )

    # Forge the new Block by adding it to the chain
    block = blockchain.new_block(proof=proof, previous_hash=0)

    response = {
        'message': "New Block Forged",
        'index': block['index'],
        'transactions': block['transactions'],
        'proof': block['proof'],
        'previous_hash': block['previous_hash'],
    }
    return jsonify(response), 200
```

### Step 4 - Interacting with the blockchain
Start up your server.

Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
Now open up postman, at the top of postman you will see a search bar. Make sure the button to the left of it is set to GET.

Then type into the bar http://localhost:5000/mine
