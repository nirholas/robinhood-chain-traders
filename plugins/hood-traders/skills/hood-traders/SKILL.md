---
name: hood-traders
description: Run the hood-traders fleet of autonomous Robinhood Chain agents in paper mode against live market data, read its dashboard API and decision journal, and halt it with the kill switch. Use to test or study memecoin and Stock Token strategies without real funds.
---

# hood-traders

`hood-traders` runs three strategy agents on Robinhood Chain (mainnet 4663, testnet 46630): `launch-sniper` (new NOXA and The Odyssey launches), `momentum` (graduated tokens), and `premium-watch` (Stock Token premium or discount against Chainlink). Each tick an agent observes, decides, simulates with a real `eth_call`, checks risk caps, executes and journals the result. Paper mode is the default: fills are simulated against live prices and no funds move.

## Run it (paper mode)

The package is on npm and its bin is `hood-traders`. Node 20+.

```bash
npx -y hood-traders
```

Or from a clone: `git clone https://github.com/nirholas/robinhood-chain-traders`, then `npm install && npm run build && npm run fleet`. The dashboard and API listen on `http://localhost:4670` (`DASHBOARD_PORT`). State is kept in SQLite at `./data/hood-traders.db` (`HOOD_TRADERS_DB`).

## Read the fleet

```bash
curl http://localhost:4670/api/summary
curl http://localhost:4670/api/agents
curl http://localhost:4670/api/agents/sniper-1/journal
curl http://localhost:4670/api/agents/sniper-1/trades
curl http://localhost:4670/api/agents/sniper-1/equity
curl http://localhost:4670/api/trades
curl http://localhost:4670/api/health
```

Agent ids are `sniper-1`, `momentum-1` and `premium-1`. The journal records why each trade fired or did not, which is the thing to read when evaluating a strategy.

## Stop it

Any one of these halts new orders across the fleet: Ctrl-C or SIGTERM, creating the file named by `KILL_FILE` (default `./KILL`), or `curl -X POST http://localhost:4670/api/kill`. The kill switch never force-sells; it only stops taking on new risk.

## Configuration

All optional in paper mode: `HOOD_NETWORK` (`mainnet` or `testnet`), `HOOD_RPC_URL`, `FLEET_MAX_DAILY_SPEND_USDG` (250), `AGENT_MAX_POSITION_USDG` (50), `AGENT_MAX_DAILY_SPEND_USDG` (100), `AGENT_MAX_SLIPPAGE_BPS` (100), `AGENT_COOLDOWN_SECONDS` (60). Risk caps are enforced before execution in both modes.

## Live mode

Live mode signs and broadcasts real transactions. It requires both `HOOD_TRADERS_LIVE=1` and `ROBINHOOD_CHAIN_PRIVATE_KEY`; with either missing the fleet stays in paper mode. Stock Token acquisition additionally needs `HOOD_STOCK_TOKEN_ELIGIBLE=true`, otherwise `premium-watch` stays alerts-only.

Do not switch to live mode on your own. Show the user the network, wallet, risk caps and fleet daily cap, and wait for an explicit yes. Provide the key only through the environment, never in a file, a prompt or a command line.

## Worked example

```bash
HOOD_NETWORK=testnet npx -y hood-traders &
sleep 30
curl -s http://localhost:4670/api/summary
curl -s http://localhost:4670/api/agents/premium-1/journal
curl -X POST http://localhost:4670/api/kill
```
