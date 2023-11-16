# Execute Contracts

## Command Line Interface

When you execute a message, a user can also pass through a flag which sends funds from their account to the contract to do logic. You can check if a user sends any funds in your contract's execute endpoint with the `info.funds` array of Coins sent by the user. These funds then get added to the contracts balance just like any other account. So it is up to you as the developer to ensure to save how many funds each user has sent via a BTreeMap or other object storage in state (if they can redeem funds back at a later time).

To send funds to a contract with some arbitrary endpoint, you use the `--amount` flag.

```sh
mintstationd tx wasm execute CONTRACT '{"some_endpoint":{}}' --amount 1000000umint
```

If the "some_endpoint" execute errors on the contract, the funds will remain in the users account.

## Typescript

```typescript
import type { Coin } from '@cosmjs/stargate';
import { SigningStargateClient, StargateClient, type StdFee } from '@cosmjs/stargate';

import type { OfflineAminoSigner } from '@cosmjs/amino';
import type { OfflineDirectSigner } from '@cosmjs/proto-signing';

import { SigningCosmWasmClient } from '@cosmjs/cosmwasm-stargate';

let RPC = '<RPC_ENDPOINT>';

const get_wallet_for_chain = async (
  chain_id: string,
): Promise<OfflineAminoSigner | OfflineDirectSigner> => {
  // open keplr
  const cosmostation = window as CosmostationWindow;
  if (keplr === undefined) {
    throw new Error('Cosmostation not found');
  }
  let signer = cosmostation.getOfflineSignerAuto;
  if (signer === undefined) {
    throw new Error('Cosmostation not found');
  }
  return signer(chain_id);
};

let wallet = await get_wallet_for_chain('mintstation-1');
let address = (await wallet.getAccounts())[0].address;

let from_client = await SigningCosmWasmClient.connectWithSigner(RPC, wallet, {
  prefix: 'mint',
});

const msg = { some_endpoint: {} };

let fee: StdFee = {
  amount: [{ amount: '5000', denom: 'umint' }],
  gas: '500000',
};

let send_amount: Coin = {
  amount: '100000',
  denom: 'umint',
};

await from_client
  .execute(address, REVIEWS_CONTRACT_ADDRESS, msg, fee, 'memo', send_amount)
  .then((res) => {
    console.log(`Success @ height ${res.height}\n\nTxHash: ${res.transactionHash}`);
  });
```
