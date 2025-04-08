# Pump.fun bot development roadmap

| Stage | Feature | Comments | Implementation status
|-------|---------|----------|---------------------|
| **Stage 1: General updates & QoL** | Lib updates | Updating to the latest libraries | ✅ |
| | Error handling | Improving error handling | WIP | 
| | Configurable RPS | Ability to set RPS in the config to match your provider's and plan RPS | WIP |
| | Dynamic priority fees | Ability to set dynamic priority fees | ✅ |
| | Review & optimize `json`, `jsonParsed`, `base64` | Improve speed and traffic for calls, not just `getBlock`. | ✅ | 
| **Stage 2: Bonding curve and migration management** | `logsSubscribe` integration | Integrate `logsSubscribe` instead of `blockSubscribe` for sniping minted tokens into the main bot | ✅ |
| | Dual subscription methods | Keep both `logsSubscribe` & `blockSubscribe` in the main bot for flexibility and adapting to Solana node architecture changes | ✅ |
| | Transaction retries | Do retries instead of cooldown and/or keep the cooldown | WIP |
| | Bonding curve status tracking | Checking a bonding curve status progress. Predict how soon a token will start the migration process | WIP | 
| | Account closure script | Script to close the associated bonding curve account if the rest of the flow txs fails | ✅ |
| | PumpSwap migration listening | pump_fun migrated to their own DEX — [PumpSwap](https://x.com/pumpdotfun/status/1902762309950292010), so we need to FAFO with that instead of Raydium (and attempt `logSubscribe` implementation) | ✅ |
| **Stage 3: Trading experience** | Take profit/stop loss | Implement take profit, stop loss exit strategies | FAFO |
| | Market cap-based selling | Sell when a specific market cap has been reached | Not started |
| | Copy trading | Enable copy trading functionality | Not started |
| | Token analysis script | Script for basic token analysis (market cap, creator investment, liquidity, token age) | Not started |
| | Archive node integration | Use Solana archive nodes for historical analysis (accounts that consistently print tokens, average mint to raydium time) | Not started |
| | Geyser implementation | Leverage Solana Geyser for real-time data stream processing | Not started |
| **Stage 4: Minting experience** | Token minting | Ability to mint tokens (based on user request - someone minted 18k tokens) | FAFO |


## 🚀 Getting started

### Prerequisites
- Install [uv](https://github.com/astral-sh/uv), a fast Python package manager.

> If Python is already installed, `uv` will detect and use it automatically.

### Installation

#### 1️⃣ Install Python (if needed)
```bash
uv python install
```
> **Why?** `uv` will fetch and install the required Python version for your system.

#### 2️⃣ Clone the repository
```bash
git clone https://github.com/chainstacklabs/pump-fun-bot.git
cd pump-fun-bot
```

#### 3️⃣ Set up a virtual environment
```bash
# Create virtual environment
uv venv

# Activate (Unix/macOS)
source .venv/bin/activate  

# Activate (Windows)
.venv\Scripts\activate
```
> Virtual environments help keep dependencies isolated and prevent conflicts.

#### 4️⃣ Install dependencies
```bash
uv pip install -e .
```
> **Why `-e` (editable mode)?** Lets you modify the code without reinstalling the package—useful for development!

#### 5️⃣ Configure the bot
```bash
# Copy example config
cp .env.example .env  # Unix/macOS

# Windows
copy .env.example .env
```
Edit the `.env` file and add your **Solana RPC endpoints** and **private key**.

### Running the bot

```bash
# Option 1: run as installed package
pump_bot --help

# Option 2: run directly
python -m src.cli --help
```

> **You're all set! 🎉**  Now you can start using the bot. Check `--help` for available commands. 🚀

## Bonding curve state check

`check_boding_curve_status.py` — checks the state of the bonding curve associated with a token. When the bonding curve state is completed, the token is migrated to Raydium.

To run:

`python check_boding_curve_status.py TOKEN_ADDRESS`

## Listening to the Raydium migration

When the bonding curve state completes, the liquidity and the token graduate to Raydium.

`listen_to_raydium_migration.py` — listens to the migration events of the tokens from pump_fun to Raydium and prints the signature of the migration, the token address, and the liquidity pool address on Raydium.

To run:

`python listen_to_raydium_migration.py`

You can take the compute the associatedBondingCurve address following the [Solana docs PDA](https://solana.com/docs/core/pda) description logic. Take the following as input *as seed* (order seems to matter):

- bondingCurve address
- the Solana system token program address: `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA`
- the token mint address

And compute against the Solana system associated token account program address: `ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL`.

The implications of this are kinda huge:
* you can now use `logsSubscribe` to snipe the tokens and you are not limited to the `blockSubscribe` method
* see which one is faster
* not every provider supports `blockSubscribe` on lower tier plans or at all, but everyone supports `logsSubscribe`

The following script showcase the implementation.

## Compute associated bonding curve

`compute_associated_bonding_curve.py` — computes the associated bonding curve for a given token.    

To run:

`python compute_associated_bonding_curve.py` and then enter the token mint address.

## Listen to new direct full details

`listen_new_direct_full_details.py` — listens to the new direct full details events and prints the signature, the token address, the user, the bonding curve address, and the associated bonding curve address using just the `logsSubscribe` method. Basically everything you need for sniping using just `logsSubscribe` and no extra calls like doing `getTransaction` to get the missing data. It's just computed on the fly now.

To run:

`python listen_new_direct_full_details.py`

So now you can run `listen_create_from_blocksubscribe.py` and `listen_new_direct_full_details.py` at the same time and see which one is faster.

## Reference Link
[ChainStack Doc](https://docs.chainstack.com/docs/solana-creating-a-pumpfun-bot)