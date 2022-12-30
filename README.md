# wYerbas Decentralized Custodian

The wYerbas Decentralized Custodian (wDDC) is a system to wrap coins from the Yerbas Mainnet to the BSC Smart Chain, and vice versa.  It consists of a _BSC smart contract_ of the wrapped coin, and multiple _authority nodes_ who must jointly verify and sign every single conversion between the two networks. This decentralized system ensures the integrity and security of the wrapped coin.

The files in this repository are used to run the authority node. Setup instructions for the authority node can be found in [INSTRUCTIONS.md](INSTRUCTIONS.md).

Users access the system via the [web application](https://wrap.yerbas.org/). The original repository's Github Pages [web application](https://wyerbas.github.io/wyerbas-frontend/) also redirects here. The source code can be found [here](https://github.com/The-Yerbas-Endeavor/wyerbas-frontend). The web application interacts directly with the authority nodes.  Alternatively you use the CLI (`node cli.js`; `help`) to interact with the authority nodes.

### Converting from Yerbas Mainnet -> BSC Smart Chain

The user first signs up for a unique Yerbas Mainnet deposit address, linked to his BSC wallet. Each authority node provides a Yerbas Mainnet deposit address, which are collated to build a multisig deposit address, linked to the user's BSC wallet. 

The user proceeds with depositing an amount of Yerbas to this unique multisig deposit address. Finally, the user requests to mint an equal amount (after taxes) of wYerbas to his BSC wallet, and this request must be verified by enough (> 1/2) authority nodes. The signed request is submitted to the BSC smart contract, which verifies on-contract all signatures before proceeding to mint.

### Converting from BSC Smart Chain -> Yerbas Mainnet

The user burns some amount of coins from his BSC wallet, specifying the withdrawal Yerbas Mainnet address in the burn request. This publicly registers the burn details (amount + withdrawal address) on the smart contract.

The user then registers the proof of burn to the authority nodes, who records the burn amount and withdrawal address. This information is used to approve withdrawal payouts.

### Payouts

Withdrawal payouts and tax payouts are not sent immediately, since the Yerbas Mainnet has to wait for the required amount of confirmations between transactions (if not the UTXOs are likely to desynchronize).

Instead, a _payout coordinator_ coordinates the payouts at regular intervals. The payout coordinator builds a raw Yerbas Mainnet transaction based on his registered withdrawal payouts and taxes collected. This transaction contains all pending withdrawal payouts and tax payouts. He sends this transaction around to each authority node, who must all verify against their own registered state before signing the raw transaction. When an authority node signs the raw transaction, he updates his database to disable signing future requests for the same payouts again. Finally, the payout coordinator submits the signed raw transaction to the Yerbas Mainnet.

Note that there is some centralization here, since only the payout coordinator can coordinate the payouts. This is necessary to prevent other nodes from requesting signatures, since every request received by a particular node will lock the payouts from future requests. In the case of renegade nodes, the other authority nodes can decide who should be the new payout coordinator, blocking off requests from the old payout coordinator.
