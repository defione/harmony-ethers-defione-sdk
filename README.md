# Harmony Ethers.js Wrapper

## How to use provider to query Harmony chain

```ts
import {
  Block,
  BlockWithTransactions,
  CXTransactionReceipt,
  StakingTransactionResponse,
  TransactionReceipt,
  TransactionResponse,
} from "../src/types";
import { ApiHarmonyProvider } from "../src/provider";

// provider auto detects shard
const provider = new ApiHarmonyProvider("https://api.s0.b.hmny.io");
// to get block with harmony properties
const block: Block = await provider.getBlock(blockNumber || blockHash);
// to get block with transactions including staking transactions
const blockWithTransaction: BlockWithTransactions = await provider.getBlockWithTransactions(blockNumber || blockHash);

// get transaction / cx transaction
const tx: TransactionResponse = provider.getTransaction(txHash);
// get staking transaction
const sTx: StakingTransactionResponse = provider.getStakingTransaction(txHash);

// to get staking transaction recipt
const txReceipt: TransactionReceipt = provider.getTransactionReceipt(txHash);

// to get Cross Shard Receipt
const cTxReceipt: CXTransactionReceipt = provider.getCXTransactionReceipt(txHash);
```

## How to use wallet to sign and send transaction and cross shard transactions

```ts
import { ApiHarmonyProvider } from "../src/provider";
import { ExternallyOwnedAccount } from "@ethersproject/abstract-signer";
import { parseEther } from "@ethersproject/units";
import Wallet from "../src/wallet";

const account: ExternallyOwnedAccount = {
  address: "0x",
  privateKey: "0x",
};

const provider = new ApiHarmonyProvider("https://api.s0.b.hmny.io");
const wallet = new Wallet(account, provider);
await provider.ready;

// transaction
const tx = await wallet.sendTransaction({
  to: "one..." || "0x...",
  value: parseEther("10"),
});

// cross shard transaction
const tx = await wallet.sendTransaction({
  to: "one..." || "0x...",
  value: parseEther("10"),
  toShardID: 1,
});
```

## How to use wallet to sign and send staking transaction

```ts
import { ExternallyOwnedAccount } from "@ethersproject/abstract-signer";
import { parseEther } from "@ethersproject/units";
import { ApiHarmonyProvider } from "../src/provider";
import { Directive, StakingTransactionRequest } from "../src/types";
import Wallet from "../src/wallet";

const account: ExternallyOwnedAccount = {
  address: "0x",
  privateKey: "0x",
};

const provider = new ApiHarmonyProvider("https://api.s0.b.hmny.io");
const wallet = new Wallet(account, provider);
await provider.ready;

// Create Validator
const createValidator: StakingTransactionRequest = {
  type: Directive.CreateValidator,
  msg: {
    validatorAddress: wallet.address,
    amount: parseEther("10000"),
    commissionRates: {
      rate: "0.1",
      maxRate: "0.1",
      maxChangeRate: "0.01",
    },
    maxTotalDelegation: parseEther("1000000"),
    minSelfDelegation: parseEther("10000"),
    slotPubKeys: [],
    slotKeySigs: [],
    description: {
      name: "Test",
      identity: "Test",
      details: "Testing",
      securityContact: "test@test.com",
      website: "test.com",
    },
  },
};

const createValidatorRes = await wallet.sendStakingTransaction(createValidator);

// Edit Validator
const editValidator: StakingTransactionRequest = {
  type: Directive.EditValidator,
  msg: {
    validatorAddress: wallet.address,
    commissionRate: "0.09",
    description: {
      name: "Test",
      identity: "test",
      details: "",
      securityContact: "test@test.com",
      website: "",
    },
    active: false,
  },
};

const editValidatorRes = await wallet.sendStakingTransaction(editValidator);

// Delegate
const delegate: StakingTransactionRequest = {
  type: Directive.Delegate,
  msg: {
    delegatorAddress: wallet.address,
    validatorAddress: "one1xjanr7lgulc0fqyc8dmfp6jfwuje2d94xfnzyd",
    amount: parseEther("1000"),
  },
};

const delegateRes = await wallet.sendStakingTransaction(delegate);

// Undelegate
const undelegate: StakingTransactionRequest = {
  type: Directive.Undelegate,
  msg: {
    delegatorAddress: wallet.address,
    validatorAddress: "one1xjanr7lgulc0fqyc8dmfp6jfwuje2d94xfnzyd",
    amount: parseEther("1000"),
  },
};

const undelegateRes = await wallet.sendStakingTransaction(undelegate);

// Collect Rewards
const collectRewards: StakingTransactionRequest = {
  type: Directive.CollectRewards,
  msg: {
    delegatorAddress: wallet.address,
  },
};

const collectRewardsRes = await wallet.sendStakingTransaction(collectRewards);
```

## HarmonyAddress

same api as harmony js sdk

```ts
const address = new HarmonyAddress("one..." || "0x...");
```

## HarmonyNetwork

```ts
interface ShardStructure {
  current: boolean;
  http: string;
  shardID: number;
  ws: string;
}

interface HarmonyNetwork extends Network {
  shardID: number;
  shardingStructure?: ShardStructure[];
}
```

## HarmonyProvider

```ts
export interface HarmonyProvider extends BaseProvider {
  network: HarmonyNetwork;

  // Execution
  sendTransaction(signedTransaction: string | Promise<string>): Promise<TransactionResponse>;
  sendStakingTransaction(signedTransaction: string | Promise<string>): Promise<StakingTransactionResponse>;

  call(transaction: Deferrable<TransactionRequest>, blockTag?: BlockTag | Promise<BlockTag>): Promise<string>;
  estimateGas(transaction: Deferrable<TransactionRequest>): Promise<BigNumber>;

  // Queries
  getBlock(blockHashOrBlockTag: BlockTag | string | Promise<BlockTag | string>): Promise<Block>;
  getBlockWithTransactions(blockHashOrBlockTag: BlockTag | string | Promise<BlockTag | string>): Promise<BlockWithTransactions>;

  getTransaction(transactionHash: string): Promise<TransactionResponse>;
  getTransactionReceipt(transactionHash: string): Promise<TransactionReceipt>;
  getCXTransactionReceipt(transactionHash: string): Promise<CXTransactionReceipt>;

  getStakingTransaction(transactionHash: string): Promise<StakingTransactionResponse>;

  getCirculatingSupply(): Promise<BigNumber>;
  getTotalSupply(): Promise<BigNumber>;

  getEpoch(): Promise<number>;
  getEpochLastBlock(epoch: number): Promise<number>;

  getLeader(): Promise<string>;

  getValidatorsAddresses(): Promise<Array<string>>;
  getActiveValidatorsAddresses(): Promise<Array<string>>;

  getDelegationsByValidator(validatorAddress: string): Promise<Array<Delegation>>;
  getDelegationsByDelegator(delegatorAddress: string): Promise<Array<Delegation>>;
}
```
