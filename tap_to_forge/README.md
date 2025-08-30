# Tap‑to‑Forge on Algorand

This repository contains the source code for **Tap‑to‑Forge**, a clicker
game built on the Algorand blockchain. Players unlock the game by paying a
small entry fee in Algos, then forge **WORK** tokens by tapping. Each tap
awards the player with WORK and simultaneously burns a portion of the
reward to a burn escrow. Gameplay is skill based – there is no chance
mechanic or profit guarantee – and the in‑game economy is handled entirely
on the blockchain.

## Features

* **Algorand Standard Asset (ASA)** – WORK is an ASA (ID `3189468904` on
  MainNet) with a fixed supply of 1 billion units. Players must opt in to
  this asset before they can receive rewards.
* **Smart Contract** – A PyTeal stateful application enforces the game
  rules, distributes rewards, burns tokens and maintains per‑user state.
* **Session & Cooldown** – Players may play for up to **one hour** per
  session. After an hour of play the account enters a **one hour
  cooldown** enforced by the contract. Tapping during cooldown is rejected.
  Once the cooldown expires a new session begins on the next tap.
* **Front‑End** – A minimalist, responsive web interface built with Vite,
  Algorand’s JavaScript SDK and Pera Wallet Connect. The UI works well
  on mobile devices and provides live feedback on unlock status, WORK
  balance and cooldown timers.
* **Deployment Script** – A Python script to compile and deploy the smart
  contracts, fund and opt‑in the burn escrow, opt‑in the application
  account to WORK and seed it with reward tokens.

## Getting Started

### Prerequisites

* Python 3.10+ with `pyteal` and `py‑algorand‑sdk`
* Node.js (18+) with `npm` for the front‑end
* An Algorand account with test Algos (for TestNet) or real Algos (for
  MainNet)
* A supply of WORK tokens in a treasury account

### Deployment

1. **Install dependencies**

   ```bash
   python -m venv venv && source venv/bin/activate
   pip install pyteal py-algorand-sdk
   cd tap_to_forge/frontend && npm install && cd ../../
   ```

2. **Set environment variables**

   Export the 25‑word mnemonic for your creator/admin account and,
   optionally, your treasury account. You can also override Algod
   endpoints and configure reward values here.

   ```bash
   export CREATOR_MNEMONIC="<creator mnemonic>"
   export TREASURY_MNEMONIC="<treasury mnemonic>"  # optional
   export ALGOD_URL="https://testnet-api.algonode.cloud"  # or MainNet
   export WORK_ASA_ID="3189468904"  # or your TestNet ASA ID
   ```

3. **Deploy**

   ```bash
   python tap_to_forge/scripts/deploy.py
   ```

   Note the printed `APP_ID`, `APP_ADDRESS` and `BURN_ESCROW_ADDRESS`.

4. **Configure the Front‑End**

   Edit `tap_to_forge/frontend/src/config.js` and set:

   * `APP_ID` to the value printed by the deploy script
   * `WORK_ASA_ID` to your asset ID (TestNet or MainNet)
   * `NETWORK` to `"TestNet"` or `"MainNet"` as appropriate

5. **Run the Front‑End**

   ```bash
   cd tap_to_forge/frontend
   npm run dev
   ```

   The site will open in your browser. Connect your Pera wallet (set to
   the correct network), opt in to WORK, unlock by paying the entry fee,
   then start forging!

## Mobile Experience

The front‑end is designed to be responsive. Buttons stack on narrow
screens and all interactive elements remain usable on mobile devices. If
you encounter any UI issues on your device, feel free to tweak the
CSS in `frontend/index.html`.

## Cooldown & Session Logic

The smart contract enforces a one hour play window followed by a one
hour cooldown. When a user taps for the first time (or after a
cooldown), the contract records the end of the play window (`ep`) and
the end of the cooldown (`cu`). Subsequent taps during the play window
are processed normally. Taps made after the play window but before the
cooldown ends are rejected. When the cooldown has expired the next tap
resets the session and updates the play/cooldown timestamps. This logic
is entirely on‑chain and cannot be bypassed.

## License

This project is provided as‑is with no warranty. You are free to
modify and use it for educational or experimental purposes.
