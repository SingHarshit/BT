# BankAccount Assignment Run Guide

This guide explains how to run the BankAccount Solidity assignment from a fresh project folder on Windows.

## 1. What this project contains

The project is a Hardhat 3 Ethereum project:

```text
BankAccount/
├── contracts/
│   └── BankAccount.sol
├── scripts/
│   ├── deploy.js
│   └── interaction.js
├── hardhat.config.js
├── package.json
└── package-lock.json
```

The smart contract supports:

- `deposit()`: deposits ETH into the account.
- `withdraw(uint256 amount)`: withdraws ETH to the caller.
- `showBalance()`: reads the contract balance tracked by the contract.

The interaction script deposits `1 ETH`, then withdraws `0.5 ETH`, and prints the resulting balance.

## 2. Prerequisites

Install the following software:

1. **Node.js LTS**, which includes npm.
2. **Visual Studio Code** or another code editor.
3. **Git**, only if you are cloning the project from a repository.

Verify Node.js and npm in PowerShell:

```powershell
node --version
npm --version
```

Node.js 18 or newer is recommended.

## 3. Open the project

Open PowerShell and change to the project root:

```powershell
cd C:\Users\ASUS\OneDrive\Desktop\BankAccount
```

All npm and Hardhat commands in this guide must be run from this folder, not from the `contracts` or `scripts` folder.

Confirm that the project files are present:

```powershell
Get-ChildItem
Get-ChildItem .\contracts
Get-ChildItem .\scripts
```

## 4. Install dependencies

For a fresh setup, install the dependencies from `package-lock.json`:

```powershell
npm install
```

This installs:

- Hardhat
- The Hardhat ethers plugin
- The ethers library used by the deployment and interaction scripts

## 5. Compile the contract

Run:

```powershell
npm run compile
```

A successful first compilation reports that one Solidity file was compiled. On later runs, this is also successful:

```text
No contracts to compile
```

That message means Hardhat detected no source changes and is using the existing compilation output. It is not an error.

The generated build files are placed in the `artifacts` and `cache` folders.

## 6. Run the tests

Run:

```powershell
npm test
```

The current assignment does not contain test files, so Hardhat may report:

```text
Running Solidity tests
```

The command is successful when PowerShell returns to the prompt without an error.

## 7. Start a local blockchain

Use two PowerShell terminals.

In **Terminal 1**, stay in the project root and run:

```powershell
cd C:\Users\ASUS\OneDrive\Desktop\BankAccount
npx hardhat node
```

Keep this terminal open. It runs a local blockchain at:

```text
http://127.0.0.1:8545
```

Hardhat prints test accounts and private keys. These accounts are for local development only and must never be used with real funds.

## 8. Deploy the contract manually

In **Terminal 2**, also change to the project root:

```powershell
cd C:\Users\ASUS\OneDrive\Desktop\BankAccount
npx hardhat run scripts/deploy.js --network localhost
```

Expected output looks like:

```text
BankAccount deployed to: 0x...
```

Copy the printed address if you want to interact with this exact deployment.

## 9. Run the complete interaction

The interaction script automatically deploys a fresh `BankAccount` contract when no address is supplied. Run:

```powershell
npx hardhat run scripts/interaction.js --network localhost
```

Expected output:

```text
BankAccount deployed to: 0x...
Account: 0x...
Balance after deposit: 1.0 ETH
Balance after withdrawal: 0.5 ETH
```

This is the easiest complete demonstration because it does not depend on an old contract address.

## 10. Interact with a specific deployment

To use the address printed by `deploy.js`, set the address in the current PowerShell session:

```powershell
$env:BANK_ACCOUNT_ADDRESS = "0xYourDeployedContractAddress"
npx hardhat run scripts/interaction.js --network localhost
```

Replace `0xYourDeployedContractAddress` with the actual address.

The address must belong to a `BankAccount` contract deployed on the currently running local node. Restarting `npx hardhat node` creates a fresh blockchain, so old addresses will no longer work.

To remove the environment variable:

```powershell
Remove-Item Env:BANK_ACCOUNT_ADDRESS
```

## 11. One-command workflow after setup

After dependencies are installed, use this workflow:

**Terminal 1**

```powershell
cd C:\Users\ASUS\OneDrive\Desktop\BankAccount
npx hardhat node
```

**Terminal 2**

```powershell
cd C:\Users\ASUS\OneDrive\Desktop\BankAccount
npm run compile
npx hardhat run scripts/interaction.js --network localhost
```

## 12. Stop the local blockchain

Return to the terminal running `npx hardhat node` and press:

```text
Ctrl+C
```

If a later attempt reports that port `8545` is already in use, find the process:

```powershell
Get-NetTCPConnection -LocalPort 8545 -State Listen
```

Stop it using the PID shown in the `OwningProcess` column:

```powershell
Stop-Process -Id <PID> -Force
```

Replace `<PID>` with the numeric process ID. Do not stop a process by name.

## 13. Common errors and fixes

### `No contracts to compile`

This is normally successful. It means the contract is already compiled and no source file changed.

### `Cannot find module`

Run this from the project root:

```powershell
npm install
```

Then retry the command.

### `EADDRINUSE: address already in use 127.0.0.1:8545`

Another local blockchain is already running. Reuse that node, or stop the process using the port-cleanup commands in section 12.

### `could not decode result data` or `BAD_DATA`

The script is connected to an address that does not contain a `BankAccount` contract on the current network. Fix it by removing the environment variable and allowing the script to deploy a fresh contract:

```powershell
Remove-Item Env:BANK_ACCOUNT_ADDRESS -ErrorAction SilentlyContinue
npx hardhat run scripts/interaction.js --network localhost
```

Alternatively, deploy again and use the new address.

### `Cannot connect to the network`

Make sure Terminal 1 is still running:

```powershell
npx hardhat node
```

Then run the deployment or interaction command again in Terminal 2.

### Running from the wrong folder

Always return to:

```powershell
cd C:\Users\ASUS\OneDrive\Desktop\BankAccount
```

Then run the command.

## 14. Contract behavior

The contract starts with a zero balance.

When `deposit()` receives `1 ETH`:

```text
balance = 1.0 ETH
```

When `withdraw(0.5 ETH)` is called:

```text
balance = 0.5 ETH
```

The contract rejects:

- Deposits of zero ETH.
- Withdrawals of zero ETH.
- Withdrawals greater than the tracked balance.

## 15. Important development notes

- The local Hardhat blockchain is temporary and resets when it is restarted.
- Contract addresses from a previous local-node session should not be reused.
- Never send real ETH to the local Hardhat test accounts.
- Never commit private keys, wallet seed phrases, or production RPC credentials.
- The current scripts use Hardhat's local test accounts and are intended for development only.
