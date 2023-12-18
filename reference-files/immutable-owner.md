---
date: Sep 01, 2023
difficulty: beginner
title: "How to use the immutable owner extension"
description:
  "With the Token program, the `SetAuthority` instruction can be used for various use 
  cases. Among them, an Account's owner may transfer ownership of an account to another.
  "
keywords:
  - token 2022
  - token extensions
  - token program
tags:
  - token 2022
  - token extensions
altRoutes:
  - /developers/guides/immutable-owner
---

With the Token program, the `SetAuthority` instruction can be used for various use cases. Among them, an Account's owner may transfer ownership of an account to another.

The immutable owner extension ensures that ownership of a token account cannot be reassigned.

## Understanding the implications

So, why is this important? The addresses for Associated Token Accounts are derived based on the owner and the mint. This makes it easy to find the related token account for a specific owner. If the account owner has reassigned ownership of this account, then applications may derive the address for that account and use it, not knowing that it no longer belongs to the owner.

This guide walks you through how to use the Immutable Owner extension to prevent the transfer of ownership of a token account.

Let's get started!

## Install dependencies

```shell
npm i @solana/web3.js @solana/spl-token
```

Install the `@solana/web3.js` and `@solana/spl-token` packages.

## Setting up

Let's start by setting up our script to create a new token mint.

First, we will need to:

- Establish a connection to the devnet cluster
- Generate a payer account and fund it
- Create a new token mint using the Token 2022 program

```javascript
import {
  clusterApiUrl,
  sendAndConfirmTransaction,
  Connection,
  Keypair,
  SystemProgram,
  Transaction,
  LAMPORTS_PER_SOL,
} from "@solana/web3.js";
import {
  createAccount,
  createMint,
  createInitializeImmutableOwnerInstruction,
  createInitializeAccountInstruction,
  getAccountLen,
  ExtensionType,
  TOKEN_2022_PROGRAM_ID,
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
const decimals = 9;

// Next, we create a new token mint
const mint = await createMint(
  connection, // Connection to use
  payer, // Payer of the transaction and initialization fees
  mintAuthority.publicKey, //Account or multisig that will control minting
  mintAuthority.publicKey, // Optional Account or multisig that can freeze token accounts
  decimals, // Location of the decimal place
  undefined, // Optional keypair, defaulting to a new random one
  undefined, // Options for confirming the transaction
  TOKEN_2022_PROGRAM_ID, // Token Program ID
);
```

As a result, we create a new token mint using the `createMint` helper function.

Lets explore two options of using the extension:

- [CLI guide](#cli-guide)
      - [Example: Explicitly creating an account with immutable ownership](#example-explicitly-creating-an-account-with-immutable-ownership)
      - [Example: Creating an associated token account with immutable ownership](#example-creating-an-associated-token-account-with-immutable-ownership)

## Creating an account with immutable ownership

### Account setup

```javascript
// owner of the token account
const ownerKeypair = Keypair.generate();
const accountKeypair = Keypair.generate();
// address of our token account
const account = accountKeypair.publicKey;

const accountLen = getAccountLen([ExtensionType.ImmutableOwner]);
const lamports = await connection.getMinimumBalanceForRentExemption(accountLen);
```

Next, we get the size of our new account and calculate the amount for rent exemption. We use the helper `getAccountLen` helper function, which takes an array of extensions we want for this account.

### The Instructions

Now, let's build the set of instructions to:

- Create a new account
- Initialize the immutable owner extension
- Initialize our new account as a token account

```javascript
const createAccountInstruction = SystemProgram.createAccount({
  fromPubkey: payer.publicKey,
  newAccountPubkey: account,
  space: accountLen,
  lamports,
  programId: TOKEN_2022_PROGRAM_ID,
});
```

We create a new account and assign ownership to the token 2022 program.

```javascript
const initializeImmutableOwnerInstruction =
  createInitializeImmutableOwnerInstruction(account, TOKEN_2022_PROGRAM_ID);
```

We then initialize the Immutable Owner extension for the given token account. It's important to note that this can only be done for accounts that have not been initialized yet.

```javascript
const initializeAccountInstruction = createInitializeAccountInstruction(
  account,
  mint,
  ownerKeypair.publicKey,
  TOKEN_2022_PROGRAM_ID,
);
```

Next, we initialize our newly created account to hold tokens.

### Send and confirm

```javascript
const transaction = new Transaction().add(
  createAccountInstruction,
  initializeImmutableOwnerInstruction,
  initializeAccountInstruction,
);
await sendAndConfirmTransaction(
  connection,
  transaction,
  [payer, accountKeypair],
  undefined,
);
```

Finally, we add the instructions to our transaction and send it to the network. As a result, we've created a token account for our new mint with the immutable owner extension applied.

If we attempt to change the owner of this account, we get an error:

```shell
"Program log: Instruction: SetAuthority",
"Program log: The owner authority cannot be changed"
```

## Creating an associated token account with immutable ownership

By default, all associated token accounts have the immutable owner extension applied.

### Create Account

```javascript
const associatedTokenAccount = await createAccount(
  connection, // connection
  payer, // fee payer
  mint, // token mint
  ownerKeypair.publicKey, // owner
  undefined, // keypair | defaults to ATA
  undefined, // confirm options
  TOKEN_2022_PROGRAM_ID, // program id
);
```

The newly created `associatedTokenAccount` has the immutable owner extension applied as the Associated Token Account program always uses the extension when creating accounts.

### Create a mint with immutable ownership in Typescript

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
    createAccount,
    createMint,
    createInitializeImmutableOwnerInstruction,
    createInitializeAccountInstruction,
    getAccountLen,
    ExtensionType,
    TOKEN_2022_PROGRAM_ID,
} from '../src';

(async () => {
    const connection = new Connection(clusterApiUrl('devnet'), 'confirmed');

    const payer = Keypair.generate();
    const airdropSignature = await connection.requestAirdrop(payer.publicKey, 2 * LAMPORTS_PER_SOL);
    await connection.confirmTransaction({ signature: airdropSignature, ...(await connection.getLatestBlockhash()) });

    const mintAuthority = Keypair.generate();
    const decimals = 9;
    const mint = await createMint(
        connection,
        payer,
        mintAuthority.publicKey,
        mintAuthority.publicKey,
        decimals,
        undefined,
        undefined,
        TOKEN_2022_PROGRAM_ID
    );

    const accountLen = getAccountLen([ExtensionType.ImmutableOwner]);
    const lamports = await connection.getMinimumBalanceForRentExemption(accountLen);

    const owner = Keypair.generate();
    const accountKeypair = Keypair.generate();
    const account = accountKeypair.publicKey;
    const transaction = new Transaction().add(
        SystemProgram.createAccount({
            fromPubkey: payer.publicKey,
            newAccountPubkey: account,
            space: accountLen,
            lamports,
            programId: TOKEN_2022_PROGRAM_ID,
        }),
        createInitializeImmutableOwnerInstruction(account, TOKEN_2022_PROGRAM_ID),
        createInitializeAccountInstruction(account, mint, owner.publicKey, TOKEN_2022_PROGRAM_ID)
    );
    await sendAndConfirmTransaction(connection, transaction, [payer, accountKeypair], undefined);

    // create associated token account
    await createAccount(connection, payer, mint, owner.publicKey, undefined, undefined, TOKEN_2022_PROGRAM_ID);
})();
```
## Conclusion

With the Immutable Owner extension, Token 2022 removes a potential foot gun. By ensuring that a token account's derived address genuinely reflects its owner.

# CLI guide
You can also use the `spl-token` CLI to create a mint with the Immutable Owner extension.

Install the CLI with:
```shell
cargo install spl-token-cli
```

#### Example: Explicitly creating an account with immutable ownership

```console
$ spl-token --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb create-token
Creating token CZxztd7SEZWxg6B9PH5xa7QwKpMCpWBJiTLftw1o3qyV under program TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb

Address:  CZxztd7SEZWxg6B9PH5xa7QwKpMCpWBJiTLftw1o3qyV
Decimals:  9

Signature: 4fT19YaE3zAscj71n213K22M3wDSXgwSn39RBCVtiCTxMX7pZhAoHywP2QMKqWpZMB5vT7diQ8QaFp3abHztpyPC
$ solana-keygen new -o account.json
$ spl-token create-account CZxztd7SEZWxg6B9PH5xa7QwKpMCpWBJiTLftw1o3qyV account.json --immutable
Creating account EV2xsZto1TRqehewwWHUUQm68X6C6MepBSkbfZcVdShy

Signature: 5NqXiE3LPFnufnZhcwKPoZt7DaPR7qwfhmRr9W9ykhNM7rnu6MDdx7n5eTpEisiaSET2R4fZW7a91Ai6pCuskXF8
```

#### Example: Creating an associated token account with immutable ownership

All associated token accounts have the immutable owner extension included, so it's extremely easy to use the extension.

```console
$ spl-token create-account CZxztd7SEZWxg6B9PH5xa7QwKpMCpWBJiTLftw1o3qyV
Creating account 4nvfLgYMERdNbbf1pADUSp44XukAyjeWWXCMkM1gMqC4

Signature: w4TRYDdCpTfmQh96E4UNgFFeiAHphWNaeYrJTu6bGyuPMokJrKFR33Ntj3iNQ5QQuFqom2CaYkhXiX9sBpWEW23
```

The CLI will tell us that it's unnecessary to specify the `--immutable` argument if it's provided:

```console
$ spl-token create-account CZxztd7SEZWxg6B9PH5xa7QwKpMCpWBJiTLftw1o3qyV --immutable
Creating account 4nvfLgYMERdNbbf1pADUSp44XukAyjeWWXCMkM1gMqC4
Note: --immutable specified, but Token-2022 ATAs are always immutable, ignoring

Signature: w4TRYDdCpTfmQh96E4UNgFFeiAHphWNaeYrJTu6bGyuPMokJrKFR33Ntj3iNQ5QQuFqom2CaYkhXiX9sBpWEW23
```
