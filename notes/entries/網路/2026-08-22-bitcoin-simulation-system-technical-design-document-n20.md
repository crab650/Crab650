---
source: personal-learning-workspace
content_type: note
source_id: 20
published_date: 2026-08-22
---

# Bitcoin Simulation System — Technical Design Document

> 分類：網路  
> Tags：無  
> 建立日期：2026-08-22

# Bitcoin Simulation System — Technical Design Document

## 1. Project Overview

This project is a **Bitcoin Simulation System** designed for learning and demonstration purposes.

The goal is not to build a production cryptocurrency or connect directly to the real Bitcoin network. Instead, the system simulates the major technical concepts behind Bitcoin, including:

* Wallet creation
* Private and public keys
* Bitcoin addresses
* Mining
* Block rewards
* Transactions
* Digital signatures
* UTXO management
* Mempool processing
* Proof of Work
* Blockchain validation
* Bitcoin transfers
* Exchange simulation
* BTC/USD conversion
* Internal exchange ledger
* Deposit and withdrawal
* Transaction reconciliation

The project can be used to understand how blockchain systems differ from traditional banking and account-based financial systems.

---

# 2. High-Level Architecture

The system contains two major subsystems:

```text
Bitcoin Network Simulator
        +
Exchange Simulator
```

The Bitcoin Network Simulator represents the blockchain layer.

The Exchange Simulator represents a centralized cryptocurrency exchange.

The overall architecture is:

```text
Wallets
   ↓
Transactions
   ↓
Mempool
   ↓
Mining
   ↓
Blocks
   ↓
Blockchain
   ↓
UTXO Set
```

The exchange operates on top of the blockchain:

```text
Bitcoin Blockchain
        ↓
     Deposit
        ↓
Centralized Exchange
        ↓
 Internal Ledger
        ↓
     Trading
        ↓
   Withdrawal
        ↓
Bitcoin Blockchain
```

---

# 3. Wallet Module

## 3.1 Purpose

A wallet represents ownership and control of Bitcoin.

The wallet does not directly contain Bitcoin.

Instead, it contains cryptographic keys that allow the owner to spend specific UTXOs.

A wallet includes:

```text
Private Key
     ↓
Public Key
     ↓
Bitcoin Address
```

---

## 3.2 Private Key

The private key is a secret cryptographic value.

It is used to sign transactions.

Example:

```text
Private Key
    ↓
Digital Signature
    ↓
Authorization to spend BTC
```

The private key must never be exposed publicly.

In a production system, private keys must not be stored as plain text.

---

## 3.3 Public Key

The public key is mathematically derived from the private key.

It can be shared publicly.

The public key is used to verify digital signatures.

```text
Private Key
     ↓
Public Key
```

The reverse operation should be computationally infeasible.

---

## 3.4 Bitcoin Address

A Bitcoin address is derived from public-key-related cryptographic data.

For the simulator, an address can be represented using a simplified format such as:

```text
btc_alice_8f92ab34
```

A more realistic implementation can simulate Bitcoin-style address generation.

---

# 4. Cryptography

The simulator can use several cryptographic technologies.

## SHA-256

SHA-256 is a cryptographic hash function.

It converts arbitrary data into a fixed-length hash.

Example:

```text
Transaction Data

Alice → Bob → 10 BTC

        ↓

SHA-256

        ↓

74f8c91a...
```

SHA-256 can be used for:

* Transaction IDs
* Block hashes
* Proof of Work
* Data integrity
* Merkle trees

---

## ECDSA

Bitcoin uses the Elliptic Curve Digital Signature Algorithm.

ECDSA allows a user to prove ownership of a private key without exposing the private key.

The simplified process is:

```text
Transaction
     +
Private Key

     ↓

Signature
```

Verification:

```text
Transaction
     +
Signature
     +
Public Key

     ↓

Valid / Invalid
```

---

## secp256k1

Bitcoin uses the elliptic curve:

```text
secp256k1
```

The simulator can use secp256k1 when implementing realistic Bitcoin-compatible cryptographic behavior.

---

# 5. Transaction Model

A Bitcoin transaction does not simply subtract money from one account and add money to another account.

Bitcoin uses the **UTXO model**.

UTXO means:

```text
Unspent Transaction Output
```

---

# 6. UTXO Model

Suppose a miner receives:

```text
50 BTC
```

The blockchain may contain:

```text
Transaction: TX001

Output #0
Owner: Alice
Amount: 50 BTC
```

Because this transaction output has not been spent, it becomes a UTXO.

```text
UTXO

TX001:0
Owner = Alice
Amount = 50 BTC
```

Alice's wallet balance is therefore calculated as:

```text
Balance =
SUM(All spendable UTXOs owned by Alice)
```

The system should not simply maintain:

```text
wallet.balance = 50
```

as the authoritative source of truth.

The UTXO set should be the primary source.

---

# 7. Sending Bitcoin

Suppose Alice owns one UTXO:

```text
50 BTC
```

Alice wants to send:

```text
10 BTC
```

to Bob.

The transaction can look like this:

```text
Input

TX001:0
50 BTC
```

Outputs:

```text
Bob
10 BTC

Alice
39.9 BTC
```

Transaction fee:

```text
0.1 BTC
```

Therefore:

```text
Input
50 BTC

=

Output
10 BTC

+

Change
39.9 BTC

+

Fee
0.1 BTC
```

---

# 8. Change Output

Bitcoin normally consumes the entire selected UTXO.

For example:

```text
Alice UTXO = 50 BTC
```

Alice cannot simply remove 10 BTC from the UTXO.

The original 50 BTC output is completely spent.

The transaction generates new outputs:

```text
Bob → 10 BTC

Alice → 39.9 BTC
```

The second output is called the:

```text
Change Output
```

---

# 9. Coin Selection

If Alice owns multiple UTXOs:

```text
UTXO A = 2 BTC
UTXO B = 5 BTC
UTXO C = 10 BTC
```

and Alice wants to send:

```text
6 BTC
```

the wallet must select suitable UTXOs.

For example:

```text
UTXO B = 5 BTC
UTXO A = 2 BTC
```

Total input:

```text
7 BTC
```

Possible outputs:

```text
Bob = 6 BTC
Alice Change = 0.9 BTC
Fee = 0.1 BTC
```

This process is called:

```text
Coin Selection
```

---

# 10. Transaction Structure

A simplified transaction object can contain:

```text
Transaction
├── Transaction ID
├── Inputs
├── Outputs
├── Fee
├── Timestamp
└── Signature
```

Example:

```json
{
  "transaction_id": "TX1001",
  "inputs": [
    {
      "previous_transaction": "TX001",
      "output_index": 0
    }
  ],
  "outputs": [
    {
      "address": "bob_address",
      "amount": 10.0
    },
    {
      "address": "alice_address",
      "amount": 39.9
    }
  ],
  "fee": 0.1
}
```

---

# 11. Transaction Validation

Before accepting a transaction, the system should verify:

```text
1. Input UTXOs exist
2. Input UTXOs are unspent
3. Signature is valid
4. Sender has authorization
5. Input amount >= output amount
6. Transaction fee is valid
7. No double spending exists
```

If any validation fails:

```text
Transaction Rejected
```

Otherwise:

```text
Transaction Accepted
```

and it can enter the mempool.

---

# 12. Mempool

The mempool contains valid transactions that have not yet been included in a block.

Flow:

```text
Wallet
   ↓
Transaction
   ↓
Validation
   ↓
Mempool
   ↓
Mining
```

Example:

| Transaction | Sender | Receiver | Amount |      Fee | Status  |
| ----------- | ------ | -------- | -----: | -------: | ------- |
| TX1001      | Alice  | Bob      | 10 BTC |  0.1 BTC | Pending |
| TX1002      | Bob    | Charlie  |  2 BTC | 0.05 BTC | Pending |

The miner selects transactions from the mempool when creating a new block.

---

# 13. Double Spending Prevention

A UTXO can only be spent once.

Suppose:

```text
UTXO = TX001:0
```

Alice tries to create:

```text
Transaction A
TX001:0 → Bob
```

and:

```text
Transaction B
TX001:0 → Charlie
```

Only one valid spending transaction can ultimately be accepted.

The simulator should detect conflicting transactions.

---

# 14. Mining

Mining is responsible for adding new blocks to the blockchain.

The simplified process is:

```text
Mempool Transactions
        ↓
Candidate Block
        ↓
Proof of Work
        ↓
Valid Block
        ↓
Blockchain
```

---

# 15. Candidate Block

A candidate block can contain:

```text
Block
├── Block Height
├── Previous Block Hash
├── Timestamp
├── Merkle Root
├── Difficulty
├── Nonce
└── Transactions
```

Example:

```text
Block Height: 100
Previous Hash: 00003fa...
Merkle Root: 89ad21...
Nonce: 184392
```

---

# 16. Proof of Work

The simulator can implement simplified Proof of Work.

For example, the block hash must start with:

```text
0000
```

Mining repeatedly changes the nonce:

```text
Nonce = 1
Hash = a72391...

Nonce = 2
Hash = f82ab4...

Nonce = 3
Hash = 908abc...

...

Nonce = 184392
Hash = 00008df7a5d...
```

Once the hash satisfies the difficulty rule:

```text
Mining Successful
```

---

# 17. Difficulty

Mining difficulty controls how difficult it is to discover a valid block.

For educational purposes:

```text
Difficulty 1
0xxxxxxxx
```

```text
Difficulty 2
00xxxxxxx
```

```text
Difficulty 3
000xxxxxx
```

Higher difficulty requires more computational attempts.

---

# 18. Block Reward

When a miner successfully mines a block, the miner receives a block reward.

For the simulator, the initial reward can be:

```text
50 BTC
```

The reward is created using a special transaction called:

```text
Coinbase Transaction
```

Example:

```text
Coinbase Transaction

Input:
NONE

Output:
Miner → 50 BTC
```

This is how new simulated Bitcoin enters circulation.

---

# 19. Coinbase Transaction

Unlike ordinary transactions, a Coinbase transaction does not spend an existing UTXO.

Instead:

```text
New Block
     ↓
Coinbase Transaction
     ↓
Mining Reward
     ↓
Miner UTXO
```

Example:

```text
TX000001

Input:
NONE

Output #0:
Miner A
50 BTC
```

This creates:

```text
UTXO
TX000001:0
50 BTC
```

---

# 20. Blockchain

Blocks are connected using cryptographic hashes.

Example:

```text
Genesis Block
      ↓
Block 1
      ↓
Block 2
      ↓
Block 3
```

Each block stores the hash of the previous block.

```text
Block 1
Hash = AAA

Block 2
Previous Hash = AAA
Hash = BBB

Block 3
Previous Hash = BBB
Hash = CCC
```

This creates a cryptographic chain.

---

# 21. Blockchain Integrity

Suppose Block 1 originally contains:

```text
Alice → Bob
10 BTC
```

Someone changes it to:

```text
Alice → Bob
100 BTC
```

The Block 1 hash changes.

Then:

```text
Block 2.previous_hash
```

no longer matches:

```text
Block 1.hash
```

The simulator should report:

```text
CHAIN VALIDATION FAILED
```

This demonstrates blockchain immutability.

---

# 22. Merkle Tree

Transactions inside a block can be summarized using a Merkle Tree.

Example:

```text
TX1      TX2      TX3      TX4
 |        |        |        |
 H1       H2       H3       H4
   \     /           \     /
    H12               H34
       \             /
          Merkle Root
```

The Merkle Root is stored inside the block header.

It provides an efficient way to verify transaction membership.

---

# 23. Blockchain Explorer

The simulator should include an Explorer UI.

Possible pages:

```text
Block Explorer
Transaction Explorer
Address Explorer
UTXO Explorer
Mempool Explorer
```

---

# 24. Block Explorer

Example:

```text
Block #100

Hash:
00008df7a5d2...

Previous Hash:
000032ab84...

Transactions:
20

Mining Reward:
50 BTC

Miner:
Miner A

Nonce:
184392
```

---

# 25. Transaction Explorer

Example:

```text
Transaction TX1001

Status:
Confirmed

Block:
#100

Input:
TX001:0

Outputs:

Bob
10 BTC

Alice
39.9 BTC

Fee:
0.1 BTC
```

---

# 26. Address Explorer

Example:

```text
Address:
btc_alice_123

Balance:
39.9 BTC

UTXOs:
TX1001:1

Transactions:
12
```

---

# 27. Exchange Simulator

The second major subsystem simulates a centralized cryptocurrency exchange.

Example:

```text
BTC/USD Exchange
```

Users can:

* Deposit BTC
* Deposit simulated USD
* Buy BTC
* Sell BTC
* Withdraw BTC
* Check balances
* Review trades
* Review transactions

---

# 28. Blockchain Balance vs Exchange Balance

This distinction is extremely important.

Suppose Alice deposits:

```text
1 BTC
```

into an exchange.

Blockchain:

```text
Alice Wallet
     ↓
Exchange Wallet
1 BTC
```

The exchange detects the deposit.

It then updates its internal database:

```text
Alice Exchange Balance
BTC = 1
```

From this point onward, internal trades do not necessarily create blockchain transactions.

---

# 29. Internal Exchange Ledger

Suppose Alice sells:

```text
0.1 BTC
```

for:

```text
5,000 USD
```

The exchange may simply update its internal ledger:

```text
Alice BTC
-0.1 BTC

Alice USD
+5,000 USD
```

No Bitcoin blockchain transaction is required.

This is because the exchange controls the underlying wallets.

---

# 30. Deposit

A BTC deposit works like this:

```text
External Wallet
      ↓
Blockchain Transaction
      ↓
Exchange Deposit Address
      ↓
Blockchain Confirmation
      ↓
Exchange Detects Deposit
      ↓
Internal Ledger Credit
```

Example:

```text
Deposit

User: Alice
Amount: 1 BTC
Transaction: TX9001
Confirmations: 3
Status: Completed
```

---

# 31. Withdrawal

A withdrawal works in the opposite direction.

```text
User Withdrawal Request
       ↓
Exchange Internal Ledger
       ↓
Validation
       ↓
Exchange Wallet
       ↓
Blockchain Transaction
       ↓
User External Address
```

The exchange reduces the user's internal balance and creates an on-chain Bitcoin transaction.

---

# 32. BTC/USD Conversion

A simple initial exchange implementation can use a fixed simulated price.

Example:

```text
1 BTC = 50,000 USD
```

Buying:

```text
Alice spends:
5,000 USD

Alice receives:
0.1 BTC
```

Selling:

```text
Alice sells:
0.1 BTC

Alice receives:
5,000 USD
```

A later version can introduce a dynamic market price.

---

# 33. Order System

A more advanced version can support:

```text
Market Order
Limit Order
Buy Order
Sell Order
```

An order can contain:

```text
Order ID
User
Side
Price
Quantity
Filled Quantity
Status
Created Time
```

Possible statuses:

```text
OPEN
PARTIALLY_FILLED
FILLED
CANCELLED
```

---

# 34. Order Book

The exchange can maintain:

```text
SELL ORDERS

Price        BTC
51,000       0.5
50,500       1.0
50,100       0.2

---------------------

BUY ORDERS

Price        BTC
50,000       0.4
49,800       0.8
49,500       1.2
```

The matching engine matches compatible buy and sell orders.

---

# 35. Exchange Ledger

The exchange should maintain an auditable internal ledger.

Example entries:

```text
Alice
BTC  -0.1

Alice
USD  +5,000
```

The system should preferably use an append-only ledger instead of directly modifying balances without history.

---

# 36. Double-Entry Accounting

The exchange module can eventually implement double-entry accounting.

Example:

Alice deposits:

```text
1 BTC
```

Possible accounting concept:

```text
Exchange BTC Asset
+1 BTC

Customer BTC Liability
+1 BTC
```

The exchange now controls one BTC but also owes one BTC to the customer.

This distinction is essential for understanding cryptocurrency exchanges.

---

# 37. Reconciliation

The exchange must verify:

```text
Blockchain Assets
        VS
Customer Liabilities
```

Example:

```text
Exchange Wallet BTC:
100 BTC

Customer Internal BTC Balances:
98 BTC

Operational Reserve:
2 BTC
```

The system should periodically perform reconciliation.

Conceptually:

```text
Blockchain
     ↓
Wallet Balances
     ↓
Exchange Ledger
     ↓
Customer Balances
     ↓
Reconciliation
```

---

# 38. Suggested Database Design

The first version can use SQLite.

Possible tables include:

```text
wallets
addresses
transactions
transaction_inputs
transaction_outputs
utxos
mempool_transactions
blocks
block_transactions
miners
exchange_users
exchange_accounts
exchange_ledger
deposits
withdrawals
orders
trades
```

---

# 39. Wallet Table

```text
wallets
-------------------------
id
name
public_key
address
created_at
```

Private-key storage should be treated separately from ordinary wallet data.

---

# 40. Transaction Table

```text
transactions
-------------------------
id
transaction_hash
transaction_type
fee
status
created_at
confirmed_block_id
```

Transaction types may include:

```text
COINBASE
TRANSFER
```

---

# 41. Transaction Input Table

```text
transaction_inputs
-------------------------
id
transaction_id
previous_transaction_id
previous_output_index
signature
public_key
```

---

# 42. Transaction Output Table

```text
transaction_outputs
-------------------------
id
transaction_id
output_index
address
amount
spent
```

---

# 43. UTXO Table

A separate cached UTXO table can also be maintained:

```text
utxos
-------------------------
transaction_id
output_index
address
amount
created_block
spent_by_transaction
```

However, the system should understand that a UTXO logically originates from transaction outputs.

---

# 44. Block Table

```text
blocks
-------------------------
id
height
previous_hash
block_hash
merkle_root
nonce
difficulty
timestamp
miner_address
```

---

# 45. Mempool Table

```text
mempool_transactions
-------------------------
transaction_id
received_at
fee
status
```

---

# 46. Exchange Account Table

```text
exchange_accounts
-------------------------
id
user_id
asset
available_balance
locked_balance
```

Example assets:

```text
BTC
USD
```

---

# 47. Exchange Ledger Table

```text
exchange_ledger
-------------------------
id
user_id
asset
transaction_type
amount
reference_type
reference_id
created_at
```

Possible transaction types:

```text
DEPOSIT
WITHDRAWAL
TRADE_BUY
TRADE_SELL
FEE
ADJUSTMENT
```

---

# 48. Recommended Technology Stack

For the first implementation:

```text
Backend:
Python

Web Framework:
Flask

Database:
SQLite

Frontend:
HTML
CSS
JavaScript

Cryptography:
Python cryptography libraries

Hashing:
SHA-256

Digital Signature:
ECDSA / secp256k1
```

A later version can use:

```text
Backend:
FastAPI

Database:
PostgreSQL

Cache:
Redis

Frontend:
Vue.js or React

Background Processing:
Celery / Redis Queue

Container:
Docker
```

---

# 49. Version 1 Scope

The first version should remain intentionally small.

Create three wallets:

```text
Alice
Bob
Miner
```

Implement:

```text
1. Create Wallet
2. Mine Block
3. Generate Mining Reward
4. Create UTXO
5. Send BTC
6. Validate Signature
7. Add Transaction to Mempool
8. Mine Pending Transactions
9. Create Block
10. Update UTXO Set
11. Calculate Wallet Balance
12. Display Blockchain Explorer
```

---

# 50. Version 1 Demonstration Scenario

## Step 1 — Create Wallets

```text
Alice
Bob
Miner
```

Balances:

```text
Alice = 0 BTC
Bob = 0 BTC
Miner = 0 BTC
```

---

## Step 2 — Mine First Block

Miner mines a block.

Reward:

```text
50 BTC
```

Result:

```text
Miner = 50 BTC
```

---

## Step 3 — Miner Sends BTC to Alice

```text
Miner
↓
20 BTC
↓
Alice
```

Result:

```text
Alice = 20 BTC
Miner ≈ 30 BTC
```

minus applicable transaction fees.

---

## Step 4 — Alice Sends BTC to Bob

Alice sends:

```text
10 BTC
```

to Bob.

The transaction first enters:

```text
Mempool
```

Status:

```text
Pending
```

---

## Step 5 — Mine Another Block

The miner selects Alice's transaction.

```text
Mempool
   ↓
Block
   ↓
Blockchain
```

The transaction becomes:

```text
Confirmed
```

---

## Step 6 — Inspect UTXOs

The system displays:

```text
Alice UTXOs
Bob UTXOs
Miner UTXOs
```

and calculates balances from these outputs.

---

# 51. Version 2 Scope

Version 2 introduces the exchange.

Implement:

```text
Exchange Account
BTC Deposit
BTC Withdrawal
USD Balance
BTC/USD Price
Buy BTC
Sell BTC
Internal Ledger
Transaction History
```

---

# 52. Version 3 Scope

Version 3 can introduce advanced exchange functionality.

```text
Limit Orders
Market Orders
Order Book
Matching Engine
Trading Fees
Locked Balance
Partial Fill
Trade History
```

---

# 53. Version 4 Scope

A later blockchain version can introduce:

```text
Multiple Nodes
Peer-to-Peer Network
Blockchain Synchronization
Forks
Longest Chain Rule
Block Propagation
Transaction Propagation
Mining Competition
Difficulty Adjustment
```

This turns the project from a blockchain data model into a distributed systems simulator.

---

# 54. Key Learning Objectives

By completing this project, the developer should understand:

```text
Bitcoin
Blockchain
Wallet
Private Key
Public Key
Digital Signature
SHA-256
ECDSA
secp256k1
Transaction
UTXO
Coin Selection
Transaction Fee
Mempool
Mining
Proof of Work
Nonce
Difficulty
Coinbase Transaction
Block Reward
Block
Blockchain
Merkle Tree
Exchange
Custody
Deposit
Withdrawal
Internal Ledger
Double-Entry Accounting
Reconciliation
Order Book
Matching Engine
```

---

# 55. Core Conceptual Difference

The project should emphasize the difference between traditional banking and Bitcoin.

Traditional account system:

```text
Alice Balance = 100
Bob Balance = 20

Transfer 10

Alice Balance = 90
Bob Balance = 30
```

Bitcoin UTXO model:

```text
Previous Outputs
      ↓
Transaction Inputs
      ↓
Transaction
      ↓
New Outputs
      ↓
New UTXOs
```

A centralized exchange introduces a third model:

```text
Blockchain UTXO Ledger
        +
Exchange Internal Account Ledger
```

Understanding the relationship between these two ledgers is one of the primary technical objectives of this project.

---

# 56. Final System Architecture

```text
                        BITCOIN SIMULATOR

┌───────────────────────────────────────────────────────┐
│                       Wallets                         │
│                                                       │
│          Alice        Bob        Miner               │
└──────────────────────────┬────────────────────────────┘
                           │
                           │ Signed Transaction
                           ▼
┌───────────────────────────────────────────────────────┐
│                       Mempool                         │
│                                                       │
│ Signature Validation                                  │
│ UTXO Validation                                       │
│ Double-Spend Detection                                │
│ Fee Validation                                        │
└──────────────────────────┬────────────────────────────┘
                           │
                           │ Mining
                           ▼
┌───────────────────────────────────────────────────────┐
│                        Block                          │
│                                                       │
│ Previous Hash                                         │
│ Merkle Root                                           │
│ Nonce                                                 │
│ Transactions                                          │
│ Coinbase Transaction                                  │
└──────────────────────────┬────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────┐
│                     Blockchain                        │
│                                                       │
│ Genesis → Block 1 → Block 2 → Block 3 → ...          │
└──────────────────────────┬────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────┐
│                       UTXO Set                        │
│                                                       │
│ TX001:0 → Alice → 10 BTC                              │
│ TX002:1 → Bob   → 5 BTC                               │
└───────────────────────────────────────────────────────┘


                    CENTRALIZED EXCHANGE

┌───────────────────────────────────────────────────────┐
│                 Blockchain Wallets                    │
└──────────────────────────┬────────────────────────────┘
                           │
                Deposit / Withdrawal
                           │
                           ▼
┌───────────────────────────────────────────────────────┐
│                  Exchange Ledger                      │
│                                                       │
│ BTC Accounts                                          │
│ USD Accounts                                          │
│ Deposits                                              │
│ Withdrawals                                           │
│ Fees                                                  │
└──────────────────────────┬────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────┐
│                   Trading Engine                      │
│                                                       │
│ Buy / Sell                                            │
│ Order Book                                            │
│ Matching Engine                                       │
│ Trade History                                         │
└───────────────────────────────────────────────────────┘
```

## Conclusion

The recommended development strategy is to build the system progressively rather than attempting to reproduce Bitcoin Core immediately.

The first milestone should focus on:

```text
Wallet
→ Mining Reward
→ UTXO
→ Transaction
→ Signature
→ Mempool
→ Mining
→ Block
→ Blockchain Explorer
```

Once this flow works correctly, the project can expand into:

```text
Exchange
→ Deposit
→ Internal Ledger
→ BTC/USD Trading
→ Withdrawal
→ Double-Entry Accounting
→ Reconciliation
```

This architecture creates a complete educational environment for studying both **decentralized cryptocurrency infrastructure** and **centralized financial ledger systems**.
