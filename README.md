# PoS Wrapper Contracts

These contracts enable anyone to withdraw using the PoS bridge to specific addresses on the root chain. We leverage the
PoS bridge and special `MessageSent` events to communicate between two contracts with the same address.

These contracts are mostly intended for multisigs (like Gnosis safes) who cannot use the PoS bridge as it withdraws to
the same address, this is resolved using another event in conjunction to communicate the destination address.

## How does it work?
1. Approve our contract to move any arbitrary ERC-20 token. Note that this token needs to be mapped on the PoS bridge
   for us to exit with.
2. Call `withdrawTo()` on the child chain contract with any amount of your child token and the address of the receiver
   on the root chain.
3. The transaction emits a `Transfer` event to `0x0` and a `MessageSent` event for our root contract to decode the
   details we want to send across.
4. Wait till the transaction is checkpointed on the root chain.
5. Generate proof for the `Transfer` event and determine the offset of the `MessageSent` event (typically `1`).
6. Call `exit()` with the burn proof and the offset. The contract will execute an exit from the PoS bridge in its own 
   context and allow you to exit with your tokens by constructing the proof for the `MessageSent` event using your
   provided offset and the burn proof.
7. Your tokens are unlocked! Ta-da 🎉

## Deploying contracts

### Set environment variables

Create a `.env` with your private key, similar to `.env.example`:

```
ETHERSCAN_API_KEY=ABC123ABC123ABC123ABC123ABC123ABC1
ROPSTEN_URL=https://eth-ropsten.alchemyapi.io/v2/<YOUR ALCHEMY KEY>
PRIVATE_KEY=0xabc123abc123abc123abc123abc123abc123abc123abc123abc123abc123abc1
```

### Deployments

Run `deploy.sh` to deploy contracts on both chains, alternatively, you can use the specific deployment scripts. Keep
in mind that the contracts must be deployed to the same address on both chains to work properly, you can leverage the
`CREATE` opcode with the same deployer address and nonce.

For example,

```bash
./scripts/deploy.sh
```

or

```bash
npx run --network goerli scripts/deployroot.js

npx run --network mumbai scripts/deploychild.js
```
