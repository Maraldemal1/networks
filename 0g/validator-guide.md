# **Create the validator:**

Create a wallet for our validator:

```bash
0gchaind keys add $WALLET_NAME --eth
```

You may also import an existing wallet:

```bash
0gchaind keys add --recover $WALLET_NAME --eth
```

Paste the password (Do not enter it, but **copy** the password in advance and **paste** it into the **white square** + **Enter**)

We will be given our wallet with a seed — we save it in a safe place:

Request private key of the EVM address:

```bash
0gchaind keys unsafe-export-eth-key $WALLET_NAME
```

Copy address and import in Metamask.

Now, we go to the [faucet](https://faucet.0g.ai/) and request test tokens

Checking the balance in the terminal:

```bash
0gchaind q bank balances $(0gchaind keys show $WALLET_NAME -a)
```

### The faucet gives you 1000000000000000000aevmos. For a validator to join an active set, you need at least 10000000000000000000aevmos (10 times more)

- Create the validator (you may change identity, website and details):

```bash
0gchaind tx staking create-validator \
  --amount=1000000ua0gi \
  --pubkey=$(0gchaind tendermint show-validator) \
  --moniker="$MONIKER" \
  --chain-id=zgtendermint_16600-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --details="Your Details" \
  --min-self-delegation="1" \
  --from=$WALLET_NAME \
  --gas=auto \
  --gas-adjustment=1.4
```

- Copy 0gvaloper address

We delegate tokens to ourselves:

```bash
0gchaind q staking validator $(0gchaind keys show $WALLET_NAME --bech val -a)
```

Delegating to another validator:

```bash
0gchaind tx staking delegate <validator address> --from <wallet> <amount>ua0gi --gas=auto --gas-adjustment=1.4 -y
```

Check transactions in Explorer.
