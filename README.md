# Cosmos-based Node Ansible Setup

This repo is to set up the Cosmos-based node. It currently support:

- Akash
- BitCanna
- Chihuahua
- Comdex
- Juno
- Kava
- KiChain
- Osmosis
- Sifchain
- Terra

## Summary

You run one playbook and set up a node. For example:

```bash
ansible-playbook -i inventory juno.yml -e "target=juno_mainnet_main"
```

But before you rush with this easy setup, you probably want to read on so you understand the structure of this Ansible program and all the features it offers.

## Some Preparations

First, make sure that you have a production inventory file with your confidential server info. You will start by copying the sample inventory file (included in the repo). The sample file gives you a good idea on how to define the inventory.

```bash
cp inventory.sample inventory
```

Needless to say, you need to update the dummy values in the inventory file. For each node, you need to update the server IP, validator_name and log_name (log_name is optional if you do not install Promtail). While you are free to keep the default, you might also want to update:

1. ansible_user: The sample file assumes `ubuntu`, but feel free to use other user name. This user need sudo privilege.
2. ansible_port: The sample file assumes `22`. But if you are like me, you will have a different ssh port other than `22` to avoid port sniffing.
3. ansible_ssh_private_key_file: The sample file assumes `~/.ssh/id_rsa`, but you might have a different key location.
4. path: This is to make sure that the ansible_user can access the `go` executable. If your ansible_user is not `ubuntu`, you need to update it.
5. node_exporter: Default is `true`. Change it to `false` if you do not want to install node_exporter
6. promtail: Default is `true`. Change it to `false` if you do not want to install promtail
7. log_monitor: Enter your monitor server IP. It is most likely a private IP address if you use a firewall around your private virtual cloud (VPC). You do not need this if you do not install promtail
8. cosmos_exporter: Default is `false`. Change it to `true` if you want to install cosmos_exporter

It is beyond the scope of this guide to help you create a sudo user, alternate ssh port, create a private key, install Ansible on your machine, etc. You can do a quick online search and find the answers. In my experience, Digital Ocean have some quality guides on these topics. Stack Overflow can help you trouble-shoot if you are stuck.

## Basic Cluster Structure

The basic cluster structure is:

1. Name each juno node as `juno_mainnet_main`, `juno_mainnet_backup`, etc. Group all Juno nodes into `juno_mainnet` group.
2. Name each juno node as `comdex_mainnet_main`, `Comdex_mainnet_test`, etc. Group all comdex nodes into `comdex_mainnet` group.
3. ...

The structure allows you to target `vars` to each node, or a group cluster, or the whole cluster. Make sure that you are familiar with the files in the `group_vars` folder. They follow this clustered structure closely.

## Playbooks

The key Ansible playbook is `<chain>.yml` files. It will set up a fresh node from scratch. For example:

```bash
ansible-playbook -i inventory akash.yml -e "target=akash_mainnet_main"
ansible-playbook -i inventory bitcanna.yml -e "target=bitcanna_mainnet_main"
ansible-playbook -i inventory chihuahua.yml -e "target=chihuahua_mainnet_main"
ansible-playbook -i inventory comdex.yml -e "target=comdex_mainnet_main"
ansible-playbook -i inventory juno.yml -e "target=juno_mainnet_main"
ansible-playbook -i inventory kava.yml -e "target=kava_mainnet_main"
ansible-playbook -i inventory kichain.yml -e "target=kichain_mainnet_main"
ansible-playbook -i inventory osmosis.yml -e "target=osmosis_mainnet_main"
ansible-playbook -i inventory sifchain.yml -e "target=sifchain_betanet_main"
ansible-playbook -i inventory terra.yml -e "target=terra_mainnet_main"
```

If you prefer to install the node manually, you can run a 'prepare' playbook to set up a server for a cosmos-based chain without installing the node itself.

Playbooks are:

| Playbook             | Description                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------------- |
| `prepare.yml `       | Prepare the server with node_exporter, promtail, go, cosmovisor, and firewall rules         |
| `auto_compound.yml ` | Copy an auto-compound shell script (more details below)                                     |
| `bitcanna.yml`       | Set up Bitcanna node. It includes the general `prepare` task and `bitcanna` specific task   |
| `chihuahua.yml`      | Set up Chihuahua node. It includes the general `prepare` task and `chihuahua` specific task |
| `comdex.yml`         | Set up Comdex node. It includes the general `prepare` task and `comdex` specific task       |
| `juno.yml`           | Set up Juno node. It includes the general `prepare` task and `juno` specific task           |
| `kava.yml`           | Set up Kava node. It includes the general `prepare` task and `kava` specific task           |
| `kichain.yml`        | Set up KiChain node. It includes the general `prepare` task and `kichain` specific task     |
| `osmosis.yml`        | Set up Osmosis node. It includes the general `prepare` task and `osmosis` specific task     |
| `sifchain.yml`       | Set up Sifchain node. It includes the general `prepare` task and `sifchain` specific task   |
| `terra.yml`          | Set up Sifchain node. It includes the general `prepare` task and `terra` specific task      |

## Auto Compound

The playbook will copy an auto-compound script to the user home directory.

| Playbook                            | Description                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| `auto_compound.yml`                 | Use this playbook when auto-compounding with own validator   |
| `auto_compound_for_delegation.yml ` | Use this playbook when auto-compounding with other validator |

```bash
ansible-playbook -i inventory auto_compound.yml -e "target=chihuahua_main"
ansible-playbook -i inventory auto_compound_for_delegation.yml -e "target=chihuahua_main"
```

You can run the script on the node with the following:

```bash
./auto_compound.sh <KEY> <PASSWORD>
./auto_compound_for_delegation.sh <KEY> <PASSWORD>
```

Alternatively, you can add a cronjob. For example, this following cronjob will run the auto-compound script daily at midnight. In this case, make sure that your server is super secure, as you will expose your key password in the crontab. Adopt this strategy at your own risk.

```bash
0 0 * * * /bin/bash /home/<USER>/auto_compound.sh <KEY> <PASSWORD>
0 0 * * * /bin/bash /home/<USER>/auto_compound_for_delegation.sh <KEY> <PASSWORD>
```

## Node Snapshot

We also offer node snapshot service for the validator community. We run snapshot script on any networks where we run a backup node. You need to have awscli installed and configured.

| Playbook       | Description                                  |
| -------------- | -------------------------------------------- |
| `snapshot.yml` | Copy the snapshot shell script to the server |

```bash
ansible-playbook -i inventory snapshot.yml -e "target=chihuahua_main_backup"
```

You can run the script on the node with the following:

```bash
./snapshot.sh
```

Alternatively, you can add a cronjob. For example, this following cronjob will run the snapshot script at the midnight

```bash
0 0 * * * /bin/bash /home/<USER>/auto_compound.sh
```

## Additional Info

When you install a node that has upgrades in the past, you can either sync from Block 1, or use a snapshot to time-travel to the present quickly. We strongly recommend using snapshots. It will save you time of syncing and debugging. Here are a list of trusted snapshot providers. Do your own research and make sure that you can trust the snapshot providers:

Akash: Snapshot here [here](https://docs.akash.network/operations/node#step9-blockchain-snapshot-use)

Kava: Snapshot is [here](https://www.chainlayer.io/quicksync/)

KiChain: Follow instruction [here](https://mzonder.notion.site/KiChain-2-Mainnet-Clean-Install-b20ce6400131499f854abc7567ce3b3f). In fact, we cannot make it work with the peers listed when we tried to sync from Block 1. The only way we made it work is by syncing with the snapshot from the included link.

Osmosis: Snapshot is [here](https://www.chainlayer.io/quicksync/)

Sifchain: Follow instruction [here](https://github.com/Sifchain/sifchain-validators/blob/master/docs/setup/standalone/manual.md)

## Please stake with our validators

| Network      | Validator                                                 | Useful Commands                | Notes to self         |
| ------------ | --------------------------------------------------------- | ------------------------------ | --------------------- |
| Akash ☁️     | `akashvaloper1gp957czryfgyvxwn3tfnyy2f0t9g2p4pz5w0ry`     | [Akash](docs/akash.md)         | Yes                   |
| Bitcanna 🌿  | `bcnavaloper1gp957czryfgyvxwn3tfnyy2f0t9g2p4pxqv0cj`      | [BitCanna](docs/bitcanna.md)   |                       |
| Chihuahua 🐶 | `chihuahuavaloper1gp957czryfgyvxwn3tfnyy2f0t9g2p4p40qac2` | [Chihuahua](docs/chihuahua.md) |                       |
| Comdex 🛢️    | `comdexvaloper1gp957czryfgyvxwn3tfnyy2f0t9g2p4p3447dz`    | [Comdex](docs/comdex.md)       | Currently Inactive    |
| Juno ♃       | `junovaloper1gp957czryfgyvxwn3tfnyy2f0t9g2p4pvzc6k3`      | [Juno](docs/juno.md)           | Yes                   |
| Kava 🌋      | `kavavaloper125s8t5c6ypwee7ytun90lnhgpls2zl3vta43aj`      | [Kava](docs/kava.md)           |                       |
| KiChain 🆗   | `kivaloper1gp957czryfgyvxwn3tfnyy2f0t9g2p4pq8jud7`        | [KiChain](docs/kichain.md)     |                       |
| Osmosis 🧪   | `osmovaloper1gp957czryfgyvxwn3tfnyy2f0t9g2p4phpkatp`      | [Osmosis](docs/osmosis.md)     | Yes (commission)      |
| Sifchain ⛓️  | `sifvaloper1gp957czryfgyvxwn3tfnyy2f0t9g2p4pfj2j90`       | [Sifchain](docs/sifchain.md)   | File ready; need work |
| Stargaze ⛓️  | `starsvaloper1gp957czryfgyvxwn3tfnyy2f0t9g2p4p60w86a`     | [Stargaze](docs/stargaze.md)   | Currently inactive    |

## P.S.

[General](docs/general.md)

[Terra](docs/terra.md), [Injective](docs/injective.md), [Celestia](docs/celestia.md)

[Validator Server Migration Best Practice](docs/validator_server_migration_best_practice.md)

[Polkachu's Testnet Setup](docs/testnets.md)

[Polkachu's Mainnet Setup](docs/mainnet.md)
