# Hackmos

- Documentation for [spawn](https://github.com/rollchains/spawn)



### How to setup spawn

#### Prerequisites Setup

If you do not have [`go 1.22+`](https://go.dev/doc/install), [`Docker`](https://docs.docker.com/get-docker/), or [`git`](https://git-scm.com/) installed, follow the instructions below.

* [MacOS](https://github.com/rollchains/spawn/blob/release/v0.50/docs/versioned_docs/version-v0.50.x/01-setup/01-system-setup.md#macos)
* [Windows](https://github.com/rollchains/spawn/blob/release/v0.50/docs/versioned_docs/version-v0.50.x/01-setup/01-system-setup.md#windows)
* [Ubuntu](https://github.com/rollchains/spawn/blob/release/v0.50/docs/versioned_docs/version-v0.50.x/01-setup/01-system-setup.md#linux-ubuntu)


### Install Spawn

```bash
# Download the the Spawn repository
git clone https://github.com/rollchains/spawn.git --depth=1 --branch v0.50.8
cd spawn

# Install Spawn
make install

# Install Local-Interchain (testnet runner)
make get-localic

# Attempt to run a command
spawn help

# If you get "command 'spawn' not found", add to path.
# Run the following in your terminal to test
# Then add to ~/.bashrc (linux / windows) or ~/.zshrc (mac)
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc
source ~/.bashrc
```

## Spawn in Action

In this 4 minute demo we:
- Create a new chain, customizing the modules and genesis
- Create a new `ems` module
  - Add the new message structure for transactions and queries
  - Store the new data types
  - Add the application logic
  - Connect it to the command line
- Build and launch a chain locally
- Interact with the chain's nameservice logic, settings a name, and retrieving it

Follow [these](./Instructions.md) docs to create EMS module


#### References

- https://github.com/cosmos/cosmos-sdk
- https://docs.osmosis.zone/osmosis-core/modules/tokenfactory/
- https://docs.cosmos.network/main/build/architecture/adr-043-nft-module