---
date: Sep 01, 2023
title: How to use the Default Account State extension
description:
  "A token creator may want to restrict who can access and use their token.
  While several approaches exist, many involve an initial reliance on
  centralized services. Considering that anyone can create a token account for a
  given mint, the current solutions are not comprehensive."
keywords:
  - token 2022
  - token extensions
  - token program
difficulty: beginner
tags:
  - token 2022
  - token extensions
altRoutes:
  - /developers/guides/default-account-state
---

A token creator may want to restrict who can access and use their token. While several approaches exist, many involve an initial reliance on centralized services. Considering that anyone can create a token account for a given mint, the current solutions are not comprehensive.

The Default Account State extension allows all newly created token accounts to have a default state, such as being frozen. This way, users must eventually interact with some service to unfreeze their account in order to use the tokens.

This guide walks you through how to use the Default Account State extension to freeze all accounts upon creation.

Let’s get started!

## Install dependencies

```shell
npm i @solana/web3.js @solana/spl-token
```

Install the `@solana/web3.js` and `@solana/spl-token` packages.

## Setting up

Let’s start by setting up our script to create a new token mint.

First, we will need to:

- Establish a connection to the devnet cluster
- Generate a payer account and fund it
- Create a new token mint using the Token 2022 program

```javascript
import {
  Connection,
  Keypair,
  LAMPORTS_PER_SOL,
  SystemProgram,
  Transaction,
  clusterApiUrl,
  sendAndConfirmTransaction,
} from "@solana/web3.js";
import {
  AccountState,
  ExtensionType,
  TOKEN_2022_PROGRAM_ID,
  createAccount,
  createInitializeDefaultAccountStateInstruction,
  createInitializeMintInstruction,
  getMintLen,
  updateDefaultAccountState,
} from "@solana/spl-token";

// We establish a connection to the cluster
const connection = new Connection(clusterApiUrl("devnet"), "confirmed");

// Next, we create and fund the payer account
const payer = Keypair.generate();
const airdropSignature = await connection.requestAirdrop(
  payer.publicKey,
  2 * LAMPORTS_PER_SOL,
);
await connection.confirmTransaction({
  signature: airdropSignature,
  ...(await connection.getLatestBlockhash()),
});
```

## Mint setup

Next, let's configure the properties of our token mint and generate the necessary authorities.

```javascript
// authority that can mint new tokens
const mintAuthority = Keypair.generate();
const mintKeypair = Keypair.generate();
// the address for our mint
const mint = mintKeypair.publicKey;
// The amount of decimals for our mint
const decimals = 9;

const mintLen = getMintLen([ExtensionType.DefaultAccountState]);
const lamports = await connection.getMinimumBalanceForRentExemption(mintLen);
```

Next, we get the size of our new account and calculate the amount for rent exemption. We use the helper `getMinLen` helper function, which takes an array of extensions we want for this mint.

## The Instructions

Now, let's build the set of instructions to:

- Create a new mint account
- Initialize the default account state extension
- Initialize our new account as a token mint

```javascript
const createAccountInstruction = SystemProgram.createAccount({
  fromPubkey: payer.publicKey, // account that will transfer lamports to the created account
  newAccountPubkey: mint, // pulic key of the created account
  space: mintLen, // amount of bytes to allocate to the created account
  lamports, // amount of lamports to transfer to the created account
  programId: TOKEN_2022_PROGRAM_ID, // public key of the program to assign as the owner
});
```

We create our mint account and assign ownership to the token 2022 program.

```javascript
const defaultState = AccountState.Frozen;

const initializeDefaultAccountStateInstruction =
  createInitializeDefaultAccountStateInstruction(
    mint, // mint to initialize
    defaultState, // default account state to set on all new accounts
    TOKEN_2022_PROGRAM_ID, // SPL token program id
  );
```

Next, we initialize the Default Account State extension for our mint, setting all accounts to be frozen.

```javascript
const initializeMintInstruction = createInitializeMintInstruction(
  mint, // token mint account
  decimals, // number of decimals
  mintAuthority.publicKey, // minting authority
  mintAuthority.publicKey, // freeze authority
  TOKEN_2022_PROGRAM_ID, // SPL token program id
);
```

We then initialize our account as a mint account.

## Send and confirm

```javascript
const transaction = new Transaction().add(
  createAccountInstruction,
  initializeDefaultAccountStateInstruction,
  initializeMintInstruction,
);
await sendAndConfirmTransaction(
  connection,
  transaction,
  [payer, mintKeypair],
  undefined,
);
```

Finally, we add the instructions to our transaction and send it to the network. As a result, we've created a mint account with the default account state extension.

## Updating Default State

The token creator can choose to relax the initial restriction by using the `updateDefaultAccountState` function by passing in the new account state to set on created accounts.

```javascript
await updateDefaultAccountState(
  connection, // connectoin to use
  payer, // payer of the transaction fee
  mint, // mint to modify
  AccountState.Initialized, // new account state to set on created accounts
  freezeAuthority, // freeze authority
  [], // signing accounts
  undefined, // options for confirming the transaction
  TOKEN_2022_PROGRAM_ID, // SPL token program id
);
```

### Create a token mint with a frozen default state in Typescript

```typescript
import {
    clusterApiUrl,
    sendAndConfirmTransaction,
    Connection,
    Keypair,
    SystemProgram,
    Transaction,
    LAMPORTS_PER_SOL,
} from '@solana/web3.js';
import {
    AccountState,
    createInitializeMintInstruction,
    createInitializeDefaultAccountStateInstruction,
    getMintLen,
    updateDefaultAccountState,
    ExtensionType,
    TOKEN_2022_PROGRAM_ID,
} from '../src';

(async () => {
    const payer = Keypair.generate();

    const mintAuthority = Keypair.generate();
    const freezeAuthority = Keypair.generate();
    const mintKeypair = Keypair.generate();
    const mint = mintKeypair.publicKey;

    const extensions = [ExtensionType.DefaultAccountState];
    const mintLen = getMintLen(extensions);
    const decimals = 9;

    const connection = new Connection(clusterApiUrl('devnet'), 'confirmed');

    const airdropSignature = await connection.requestAirdrop(payer.publicKey, 2 * LAMPORTS_PER_SOL);
    await connection.confirmTransaction({ signature: airdropSignature, ...(await connection.getLatestBlockhash()) });

    const defaultState = AccountState.Frozen;

    const lamports = await connection.getMinimumBalanceForRentExemption(mintLen);
    const transaction = new Transaction().add(
        SystemProgram.createAccount({
            fromPubkey: payer.publicKey,
            newAccountPubkey: mint,
            space: mintLen,
            lamports,
            programId: TOKEN_2022_PROGRAM_ID,
        }),
        createInitializeDefaultAccountStateInstruction(mint, defaultState, TOKEN_2022_PROGRAM_ID),
        createInitializeMintInstruction(
            mint,
            decimals,
            mintAuthority.publicKey,
            freezeAuthority.publicKey,
            TOKEN_2022_PROGRAM_ID
        )
    );

    await sendAndConfirmTransaction(connection, transaction, [payer, mintKeypair], undefined);

    await updateDefaultAccountState(
        connection,
        payer,
        mint,
        AccountState.Initialized,
        freezeAuthority,
        [],
        undefined,
        TOKEN_2022_PROGRAM_ID
    );
})();
```
## Conclusion

The Default Account State extension introduces a valuable tool for token creators to have enhanced control of their token. This process provides a mechanism for controlled token distribution

# CLI Guide
You can also use the `spl-token` CLI to create a mint with the Default Account State extension.

Install the CLI with:
```shell
cargo install spl-token-cli
```

#### Example: Creating a mint with default frozen accounts

```console
$ spl-token --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb create-token --enable-freeze --default-account-state frozen
Creating token 8Sqz2zV8TFTnkLtnCdqRkjJsre3GKRwHcZd3juE5jJHf under program TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb

Address:  8Sqz2zV8TFTnkLtnCdqRkjJsre3GKRwHcZd3juE5jJHf
Decimals:  9

Signature: 5wfYvovguPEbyv2uSWxGt9JcpTWgyuP4hY3wutjS32Ahnoni4qd7gf6sLre855WvT6xLHwrvV7J8bVmXymNU2qUz

$ spl-token create-account 8Sqz2zV8TFTnkLtnCdqRkjJsre3GKRwHcZd3juE5jJHf
Creating account 6XpKagP1N3K1XnzStufpV5YZ6DksEkQWgLNG9kPpLyvv

Signature: 2awxWdQMgv89ew34sEyG361vshB2wPXHHfva5iJ43dWr18f2Pr6awoXfsqYPpyS2eSbH6jhfVY9EUck8iJ4wCSN6

$ spl-token display 6XpKagP1N3K1XnzStufpV5YZ6DksEkQWgLNG9kPpLyvv
SPL Token Account
  Address: 6XpKagP1N3K1XnzStufpV5YZ6DksEkQWgLNG9kPpLyvv
  Program: TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb
  Balance: 0
  Decimals: 9
  Mint: 8Sqz2zV8TFTnkLtnCdqRkjJsre3GKRwHcZd3juE5jJHf
  Owner: 4SnSuUtJGKvk2GYpBwmEsWG53zTurVM8yXGsoiZQyMJn
  State: Frozen
  Delegation: (not set)
  Close authority: (not set)
Extensions:
  Immutable owner
```

#### Example: Updating default state

Over time, if the mint creator decides to relax this restriction, the freeze authority may sign an `update_default_account_state` instruction to make all accounts unfrozen by default.

```console
$ spl-token update-default-account-state 8Sqz2zV8TFTnkLtnCdqRkjJsre3GKRwHcZd3juE5jJHf initialized

Signature: 3Mm2JCPrf6SrAe9awV3QzYvHiYmatiGWTmrQ7YnmzJSqyNCf75rLNMyH7jU26uZwX7q3MmBEBj1A36o5sGk9Vakb
```
