### CPI Guard
CPI Guard is an extension that prohibits certain actions inside cross-program invocations, to protect users from implicitly signing for actions they can't see, hidden in programs that aren't the System or Token programs.

Users may choose to enable or disable the CPI Guard extension on their token account at will. When enabled, it has the following effects during CPI:

* Transfer: the signing authority must be the account delegate
* Burn: the signing authority must be the account delegate
* Approve: prohibited
* Close Account: the lamport destination must be the account owner
* Set Close Authority: prohibited unless unsetting
* Set Owner: always prohibited, including outside CPI

#### Background

When interacting with a dapp, users sign transactions that are constructed by frontend code. Given a user's signature, there are three fundamental ways for a dapp to transfer funds from the user to the dapp (or, equivalently, burn them):

* Insert a transfer instruction in the transaction
* Insert an approve instruction in the transaction, and perform a CPI transfer under program authority
* Insert an opaque program instruction, and perform a CPI transfer with the user's authorization

The first two are safe, in that the user can see exactly what is being done, with zero ambiguity. The third is quite dangerous. A wallet signature allows the program to perform any action as the user, without any visibility into its actions. There have been some attempts at workarounds, for instance, simulating the transaction and warning about balance changes. But, fundamentally, this is intractable.

There are two ways to make this much safer:

* Wallets warn whenever a wallet signature is made available to an opaque (non-system, non-token) instruction. Users should be educated to treat the request for a signature on such an instruction as highly suspect
* The token program prohibits CPI calls with the user authority, forcing opaque programs to directly ask for the user's authority

The CPI Guard covers the second instance.

#### Example: Enable CPI Guard on a token account

<Tabs className="unique-tabs" groupId="language-selection">
  <TabItem value="cli" label="CLI" default>
```console
$ spl-token enable-cpi-guard 4YfkXX89TrsWqSSxb3av36Rk8EZBoDqxGzuaDNXr7UnL

Signature: 2fohon7oraTCgBZB3dfzhpGsBobYmYPgA8nvgCqKzjqpdX6EYZaBY3VwzjNuwDpsFYYNbpTVYBjxqiaMBrvXM8S2
```
  </TabItem>
  <TabItem value="jsx" label="JS">
```jsx
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
    createMint,
    createEnableCpiGuardInstruction,
    createInitializeAccountInstruction,
    disableCpiGuard,
    enableCpiGuard,
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

    const accountLen = getAccountLen([ExtensionType.CpiGuard]);
    const lamports = await connection.getMinimumBalanceForRentExemption(accountLen);

    const owner = Keypair.generate();
    const destinationKeypair = Keypair.generate();
    const destination = destinationKeypair.publicKey;
    const transaction = new Transaction().add(
        SystemProgram.createAccount({
            fromPubkey: payer.publicKey,
            newAccountPubkey: destination,
            space: accountLen,
            lamports,
            programId: TOKEN_2022_PROGRAM_ID,
        }),
        createInitializeAccountInstruction(destination, mint, owner.publicKey, TOKEN_2022_PROGRAM_ID),
        createEnableCpiGuardInstruction(destination, owner.publicKey, [], TOKEN_2022_PROGRAM_ID)
    );

    await sendAndConfirmTransaction(connection, transaction, [payer, owner, destinationKeypair], undefined);

    // OR
    await enableCpiGuard(connection, payer, destination, owner, [], undefined, TOKEN_2022_PROGRAM_ID);
})();
```
  </TabItem>
</Tabs>

#### Example: Disable CPI Guard on a token account

<Tabs className="unique-tabs" groupId="language-selection">
  <TabItem value="cli" label="CLI" default>
```console
$ spl-token disable-cpi-guard 4YfkXX89TrsWqSSxb3av36Rk8EZBoDqxGzuaDNXr7UnL

Signature: 4JJSBSc1UAtArbBqYRpTk9264WwJuZ8n6NqyXtCSmyVQpmHoetzyVDwHxtxrdK8wQawoocDxFD9rRPhpAMzJ6EdG
```
  </TabItem>
  <TabItem value="jsx" label="JS">
```jsx
    await disableCpiGuard(connection, payer, destination, owner, [], undefined, TOKEN_2022_PROGRAM_ID);
```
  </TabItem>
</Tabs>


### Complete code in Typescript
```ts
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
    createMint,
    createEnableCpiGuardInstruction,
    createInitializeAccountInstruction,
    disableCpiGuard,
    enableCpiGuard,
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

    const accountLen = getAccountLen([ExtensionType.CpiGuard]);
    const lamports = await connection.getMinimumBalanceForRentExemption(accountLen);

    const owner = Keypair.generate();
    const destinationKeypair = Keypair.generate();
    const destination = destinationKeypair.publicKey;
    const transaction = new Transaction().add(
        SystemProgram.createAccount({
            fromPubkey: payer.publicKey,
            newAccountPubkey: destination,
            space: accountLen,
            lamports,
            programId: TOKEN_2022_PROGRAM_ID,
        }),
        createInitializeAccountInstruction(destination, mint, owner.publicKey, TOKEN_2022_PROGRAM_ID),
        createEnableCpiGuardInstruction(destination, owner.publicKey, [], TOKEN_2022_PROGRAM_ID)
    );

    await sendAndConfirmTransaction(connection, transaction, [payer, owner, destinationKeypair], undefined);

    await disableCpiGuard(connection, payer, destination, owner, [], undefined, TOKEN_2022_PROGRAM_ID);

    await enableCpiGuard(connection, payer, destination, owner, [], undefined, TOKEN_2022_PROGRAM_ID);
})();
```