### Metadata Pointer

With the potential proliferation of multiple metadata programs, a mint can have multiple different accounts all claiming to describe the mint.

To make it easy for clients to distinguish, the metadata pointer extension allows a token creator to designate an address that describes the canonical metadata. As you'll see in the "Metadata" section, this address can be the mint itself!

To avoid phony mints claiming to be stablecoins, however, a client must check that the mint and the metadata both point to each other.

#### Example: Create a mint with a metadata pointer to an external account

<Tabs className="unique-tabs" groupId="language-selection">
  <TabItem value="cli" label="CLI" default>

```console
$ spl-token --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb create-token --metadata-address 7N4HggYEJAtCLJdnHGCtFqfxcB5rhQCsQTze3ftYstVj
Creating token HFg1FFaj4PqFHmkYrqbZsarNJEZT436aXAXgQFMJihwc under program TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb

Address:  HFg1FFaj4PqFHmkYrqbZsarNJEZT436aXAXgQFMJihwc
Decimals:  9

Signature: 3ug4Ejs16jJgEm1WyBwDDxzh9xqPzQ3a2cmy1hSYiPFcLQi9U12HYF1Dbhzb2bx75SSydfU6W4e11dGUXaPbJqVc
```

  </TabItem>
  <TabItem value="jsx" label="JS">

Coming soon!

  </TabItem>
</Tabs>

### Metadata

To facilitate token-metadata usage, Token-2022 allows a mint creator to include their token's metadata directly in the mint account.

Token-2022 implements all of the instructions from the [spl-token-metadata-interface](https://github.com/solana-labs/solana-program-library/tree/master/token-metadata/interface).

The metadata extension should work directly with the metadata-pointer extension. During mint creation, you should also add the metadata-pointer extension, pointed at the mint itself.

The tools do this for you automatically.

#### Example: Create a mint with metadata

<Tabs className="unique-tabs" groupId="language-selection">
  <TabItem value="cli" label="CLI" default>

```console
$ spl-token --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb create-token --enable-metadata
Creating token 5K8RVdjpY3CHujyKjQ7RkyiCJqTG8Kba9krNfpZnmvpS under program TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb
To initialize metadata inside the mint, please run `spl-token initialize-metadata 5K8RVdjpY3CHujyKjQ7RkyiCJqTG8Kba9krNfpZnmvpS <YOUR_TOKEN_NAME> <YOUR_TOKEN_SYMBOL> <YOUR_TOKEN_URI>`, and sign with the mint authority

Address:  5K8RVdjpY3CHujyKjQ7RkyiCJqTG8Kba9krNfpZnmvpS
Decimals:  9

Signature: 2BZH8KE7zVcBj7Mmnu6uCM9NT4ey7qHasZmEk6Bt3tyx1wKCXS3JtcgEvrXXEMFB5numQgA9wvR67o2Z4YQdEw7m

$ spl-token initialize-metadata 5K8RVdjpY3CHujyKjQ7RkyiCJqTG8Kba9krNfpZnmvpS MyTokenName TOKEN http://my.token --update-authority 3pGiHDDek35npQuyWQ7FGcWxqJdHvVPDHDDmBFs2YxQj
Signature: 2H16XtBqdwSbvvq8g5o2jhy4TknP6zgt71KHawEdyPvNuvusQrV4dPccUrMqjFeNTbk75AtzmzUVueH3yWiTjBCG
```

  </TabItem>
  <TabItem value="jsx" label="JS">

Coming soon!

  </TabItem>
</Tabs>

#### Example: Update a field

<Tabs className="unique-tabs" groupId="language-selection">
  <TabItem value="cli" label="CLI" default>

```console
$ spl-token update-metadata 5K8RVdjpY3CHujyKjQ7RkyiCJqTG8Kba9krNfpZnmvpS name YourToken
Signature: 2H16XtBqdwSbvvq8g5o2jhy4TknP6zgt71KHawEdyPvNuvusQrV4dPccUrMqjFeNTbk75AtzmzUVueH3yWiTjBCG
```
  </TabItem>
  <TabItem value="jsx" label="JS">

Coming soon!

  </TabItem>
</Tabs>

#### Example: Add a custom field

<Tabs className="unique-tabs" groupId="language-selection">
  <TabItem value="cli" label="CLI" default>

```console
$ spl-token update-metadata 5K8RVdjpY3CHujyKjQ7RkyiCJqTG8Kba9krNfpZnmvpS new-field new-value
Signature: 31uerYNa6yhb21k5CCX69k7RLUKEhJEV99UadEpPnZtWWpykwr7vkTFkuFeJ7AaEyQPrepe8m8xr4N23JEAeuTRY
```
  </TabItem>
  <TabItem value="jsx" label="JS">

Coming soon!

  </TabItem>
</Tabs>

#### Example: Remove a custom field

<Tabs className="unique-tabs" groupId="language-selection">
  <TabItem value="cli" label="CLI" default>

```console
$ spl-token update-metadata 5K8RVdjpY3CHujyKjQ7RkyiCJqTG8Kba9krNfpZnmvpS new-field --remove
Signature: 52s1mxRqnr2jcZNvcmcgsQuXfVyT2w1TuRsEE3J6YwEZBu74BbFcHh2DvwnJG7qC7Cy6C5ZrTfnoPREFjFS7kXjF
```
  </TabItem>
  <TabItem value="jsx" label="JS">

Coming soon!

  </TabItem>
</Tabs>
