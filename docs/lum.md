## Lum Network

create validator

```bash
lumd tx staking create-validator \
    --amount=1000000ulum \
    --pubkey=$(lumd tendermint show-validator) \
    --moniker="DeCrypto" \
    --commission-max-change-rate "0.01" \
    --commission-max-rate "0.10" \
    --commission-rate "0.05" \
    --min-self-delegation="1" \
    --website "https://decrypto.online" \
    --identity "5E54385AA88D04BB" \
    --security-contact "https://t.me/rapha_decrypto" \
    --details "DeCrypto is a validator and contributor for Cosmos based blockchains." \
    --chain-id=lum-network-1 \
    --from decrypto \
    --fees 200ulum
```

Delegate

```bash
lumd tx staking edit-validator \
   --details="Something something" \
   --chain-id=lum-network-1 \
   --from decrypto \
   --fees 200ulum
```
