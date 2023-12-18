### Transfer Hook

#### Motivation

Token creators may need more control over how their token is transferred. The most prominent use case revolves around NFT royalties. Whenever a token is moved, the creator should be entitled to royalties, but due to the design of the current token program, it's impossible to stop a transfer at the protocol level.

Current solutions typically resort to perpetually freezing tokens, which requires a whole proxy layer to interact with the token. Wallets and marketplaces need to be aware of the proxy layer in order to properly use the token.

Worse still, different royalty systems have different proxy layers for using their token. All in all, these systems harm composability and make development harder.

#### Solution

To improve the situation, Token-2022 introduces the concept of the transfer-hook interface and extension. A token creator must develop and deploy a program that implements the interface and then configure their token mint to use their program.

During transfer, Token-2022 calls into the program with the accounts specified at a well-defined program-derived address for that mint and program id. This call happens after all other transfer logic, so the accounts reflect the *end* state of the transfer.

When interacting with a transfer-hook program, it's possible to send an instruction - such as `Execute` (transfer) - to the program with only the accounts required for the `Transfer` instruction, and any extra accounts that the program may require are automatically resolved on-chain! This process is explained in detail in many of the linked `README` files below under **Resources**.

#### Resources

The interface description and structs exist at [spl-transfer-hook-interface](https://github.com/solana-labs/solana-program-library/tree/master/token/transfer-hook/interface), along with a sample minimal program implementation. You can find detailed instructions on how to implement this interface for an on-chain program or interact with a program that implements transfer-hook in the repository's [README](https://github.com/solana-labs/solana-program-library/tree/master/token/transfer-hook/interface/README.md).

The `spl-transfer-hook-interface` library provides offchain and onchain helpers for resolving the additional accounts required. See [invoke.rs](https://github.com/solana-labs/solana-program-library/tree/master/token/transfer-hook/interface/src/invoke.rs) for usage on-chain, and [offchain.rs](https://github.com/solana-labs/solana-program-library/tree/master/token/transfer-hook/interface/src/offchain.rs) for fetching the additional required account metas with any async off-chain client like `BanksClient` or `RpcClient`.

A usable example program exists at [spl-transfer-hook-example](https://github.com/solana-labs/solana-program-library/tree/master/token/transfer-hook/example). Token-2022 uses this example program in tests to ensure that it properly uses the transfer hook interface.

The example program and the interface are powered by the [spl-tlv-account-resolution](https://github.com/solana-labs/solana-program-library/tree/master/libraries/tlv-account-resolution) library, which is explained in detail in the repository's [README](https://github.com/solana-labs/solana-program-library/tree/master/libraries/tlv-account-resolution/README.md)

#### Example: Create a mint with a transfer hook

<Tabs className="unique-tabs" groupId="language-selection">
  <TabItem value="cli" label="CLI" default>

```console
$ spl-token --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb create-token --transfer-hook 7N4HggYEJAtCLJdnHGCtFqfxcB5rhQCsQTze3ftYstVj
Creating token HFg1FFaj4PqFHmkYrqbZsarNJEZT436aXAXgQFMJihwc under program TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb

Address:  HFg1FFaj4PqFHmkYrqbZsarNJEZT436aXAXgQFMJihwc
Decimals:  9

Signature: 3ug4Ejs16jJgEm1WyBwDDxzh9xqPzQ3a2cmy1hSYiPFcLQi9U12HYF1Dbhzb2bx75SSydfU6W4e11dGUXaPbJqVc
```

  </TabItem>
  <TabItem value="jsx" label="JS">

```jsx
import {
    clusterApiUrl,
    sendAndConfirmTransaction,
    Connection,
    Keypair,
    PublicKey,
    SystemProgram,
    Transaction,
    LAMPORTS_PER_SOL,
} from '@solana/web3.js';

import {
    ExtensionType,
    createInitializeMintInstruction,
    createInitializeTransferHookInstruction,
    mintTo,
    createAccount,
    getMintLen,
    TOKEN_2022_PROGRAM_ID,
} from '../src';

(async () => {
    const payer = Keypair.generate();

    const mintAuthority = Keypair.generate();
    const mintKeypair = Keypair.generate();
    const mint = mintKeypair.publicKey;

    const extensions = [ExtensionType.TransferHook];
    const mintLen = getMintLen(extensions);
    const decimals = 9;
    const transferHookProgramId = new PublicKey('7N4HggYEJAtCLJdnHGCtFqfxcB5rhQCsQTze3ftYstVj')

    const connection = new Connection(clusterApiUrl('devnet'), 'confirmed');

    const airdropSignature = await connection.requestAirdrop(payer.publicKey, 2 * LAMPORTS_PER_SOL);
    await connection.confirmTransaction({ signature: airdropSignature, ...(await connection.getLatestBlockhash()) });

    const mintLamports = await connection.getMinimumBalanceForRentExemption(mintLen);
    const mintTransaction = new Transaction().add(
        SystemProgram.createAccount({
            fromPubkey: payer.publicKey,
            newAccountPubkey: mint,
            space: mintLen,
            lamports: mintLamports,
            programId: TOKEN_2022_PROGRAM_ID,
        }),
        createInitializeTransferHookInstruction(mint, payer.publicKey, transferHookProgramId, TOKEN_2022_PROGRAM_ID),
        createInitializeMintInstruction(mint, decimals, mintAuthority.publicKey, null, TOKEN_2022_PROGRAM_ID)
    );
    await sendAndConfirmTransaction(connection, mintTransaction, [payer, mintKeypair], undefined);
})();
```

  </TabItem>
</Tabs>

#### Example: Update transfer-hook program in mint

<Tabs className="unique-tabs" groupId="language-selection">
  <TabItem value="cli" label="CLI" default>

```console
$ spl-token set-transfer-hook HFg1FFaj4PqFHmkYrqbZsarNJEZT436aXAXgQFMJihwc EbPBt3XkCb9trcV4c8fidhrvoeURbDbW87Acustzyi8N

Signature: 3Ffw6yjseDsL3Az5n2LjdwXXwVPYxDF3JUU1JC1KGAEb1LE68S9VN4ebtAyvKeYMHvhjdz1LJVyugGNdWHyotzay
```

  </TabItem>
  <TabItem value="jsx" label="JS">

```js
await updateTransferHook(
  connection,
  payer, mint,
  newTransferHookProgramId,
  payer.publicKey,
  [],
  undefined,
  TOKEN_2022_PROGRAM_ID
);
```

  </TabItem>
</Tabs>

#### Example: Manage a transfer-hook programa

A sample CLI for managing a transfer-hook program exists at [spl-transfer-hook-cli](https://github.com/solana-labs/solana-program-library/tree/master/token/transfer-hook/cli). A mint manager can fork the tool for their own program.

It only contains a command to create the required transfer-hook account for the mint.

First, you must build the transfer-hook program and deploy it:

```console
$ cargo build-sbf
$ solana program deploy target/deploy/spl-transfer-hook-example.so
```

After that, you can initialize the transfer-hook account:

```console
$ spl-transfer-hook create-extra-metas <PROGRAM_ID> <MINT_ID> [<ACCOUNT_PUBKEY>:<ROLE> ...]
```
