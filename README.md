# ECST
ECST is a blockchain storage testing framework based on transaction fuzzing and exception injection.

```
ECST/
├── account/            # Account management
├── blob/               # Blob transaction component
├── cmd/                # Command-line tools directory
│   ├── livefuzzer/     # Transaction fuzzer tool
│   └── manual/         # Manual testing tool
├── config/             # Configuration modules
├── devp2p/             # P2P network protocol modules
├── ethclient/          # Unified client management
├── fuzzer/             # Fuzzing core modules
├── logs/               # Log files directory
├── mutation/           # Mutation strategies
├── output/             # Reports storage directory
├── poc/                # Proof of Concept implementations
├── rpc/                # Local rpc component
├── scripts/            # Helper scripts
├── stress_test/        # Stress testing directory
├── templates/          # Template config files
├── testing/            # Test runner framework
├── transaction/        # Transaction building
├── utils/              # Utility functions
├── config.yaml         # Main configuration file
├── constants.go        # Global constants
├── main.go             # Program entry point
└── README.md           # Usage of the repository
```

---

## Prerequisites

* Go 1.21+
* Docker & Docker Compose (for local multi-client testnet)
* Git

## Spin up a Local Ethereum Testnet (multi-client)

We use [`ethpandaops/ethereum-package`](https://github.com/ethpandaops/ethereum-package).

```bash
./scripts/run_ethereum_network.sh -c <your_ethereumpackage_config.yaml>
# Produces scripts/output.txt with enodes and RPC URLs
```

Grab enodes/RPCs from `scripts/output.txt` and fill your YAML (see `templates/`).

---

#### Quick Start

**1. Build the tool:**
```bash
cd cmd/manual
go build -o manual
```

**2. Configure (edit `cmd/manual/config.yaml`):**
```yaml
test:
  mode: "single"              # Test mode
  single_node_index: 4        # Which node to test (0-4)
  single_node_batch_size: 1   # Number of transactions
```

**3. Run tests:**
```bash
# Use default config
./manual

# Specify custom config
./manual -config /path/to/config.yaml

# Override test mode from command line
./manual -mode multi

# List all available test modes
./manual --list

# Show version
./manual --version
```

#### Available Test Modes

| Mode | Description |
|------|-------------|
| `single` | Test a single specific node |
| `multi` | Test all configured nodes |
| `test-soft-limit` | Test soft limit for all clients |
| `test-soft-limit-single` | Test soft limit for one client |
| `test-soft-limit-report` | Generate soft limit test report |
| `GetPooledTxs` | Test GetPooledTransactions protocol |
| `oneTransaction` | Send a single transaction |
| `largeTransactions` | Send large batch of transactions |

#### Configuration

The manual tool uses its own `config.yaml` in `cmd/manual/` directory, independent from the root project configuration:

```yaml
# P2P Configuration
p2p:
  jwt_secret: "..."
  node_names:
    - "geth-lighthouse"
    - "netherhmind-teku"
    - "besu-prysm"
    - "besu-lodestar"
    - "geth-nimbus"
  bootstrap_nodes: [...]

# Test Configuration
test:
  mode: "single"
  single_node_index: 4
  single_node_nonce: 1
  single_node_batch_size: 1
  multi_node_batch_size: 20
  multi_node_nonces: [0, 0, 0, 0, 0]
  soft_limit_scenarios: [4096, 5000, 8192]
```

#### Example Usage

**Single Node Test:**
```bash
cd cmd/manual
./manual -mode single
# Tests node 4 (configured in config.yaml) with 1 transaction
```

**Multi-Node Test:**
```bash
./manual -mode multi
# Tests all 5 configured nodes with 20 transactions each
```

**Soft Limit Testing:**
```bash
./manual -mode test-soft-limit-report
# Generates comprehensive soft limit test report for all clients
```

**Custom Configuration:**
```bash
./manual -config my-test-config.yaml -mode single
# Use custom config file
```

### 2) Stress test

```bash
cd stress_test
./run_stress_test.sh
```

### POC (Proof of Concept) Testing

Specialized testing tools for specific scenarios.

#### Maximum Nonce Testing

Test extreme nonce values (`math.MaxUint64`) to verify how Ethereum clients handle boundary conditions.

**Quick Start:**
```bash
cd poc/maxNonce
# Edit maxNonce.go to configure your node parameters
go run maxNonce.go
```

**Expected Result:** Transaction should be `QUEUED` (waiting for conditions) due to extreme nonce value.

```
--- Stats (Runtime: 30s) ---
Total Sent: 150 | Mined: 145 | Failed: 3 | Pending: 2
Mutation Used: 45 | Random Used: 105
Success Rate: 96.7% | Mutation Rate: 30.0%
```

---

## Troubleshooting

* **`connection refused`**

  * Confirm RPC is reachable; node is up; firewall rules allow access.
* **`insufficient funds for gas`**

  * Fund the sender; lower `max_gas_price`/`max_gas_limit`.
* **Low observed TPS**

  * Check network latency; reduce `max_retries`/`retry_delay`; add RPCs and increase concurrency.
* **High memory**

  * Shorten `fuzz_duration_sec`; disable tracking; lower TPS.

---

## License

MIT. See [LICENSE](LICENSE).
---

