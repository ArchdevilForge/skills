# Phase 2 公共数据源端点速查（Blockscout/BSCScan/RPC/Solana）

（从主文件移出，按需加载）


# 批量（逗号分隔）
curl -s "https://coins.llama.fi/prices/current/solana:So11111111111111111111111111111111111111112,bsc:0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c"
```

1) **DexScreener**
```bash
curl -s "https://api.dexscreener.com/latest/dex/tokens/{tokenAddress}"
curl -s "https://api.dexscreener.com/latest/dex/pairs/{chainId}/{pairAddress}"
curl -s "https://api.dexscreener.com/latest/dex/search?q={query}"
```

2) **GeckoTerminal**
```bash
curl -s "https://api.geckoterminal.com/api/v2/networks/{network}/tokens/{tokenAddress}"
curl -s "https://api.geckoterminal.com/api/v2/networks/{network}/tokens/{tokenAddress}/info"
curl -s "https://api.geckoterminal.com/api/v2/networks/{network}/tokens/{tokenAddress}/pools"
```

3) **Jupiter Lite API（Solana，无 key）**
```bash
curl -s "https://lite-api.jup.ag/price/v3?ids={mintAddress}"
curl -s "https://lite-api.jup.ag/tokens/v2/search?query={mintAddress}"
```

> 说明：`api.jup.ag` 常见 401；公共无 key 场景优先 `lite-api.jup.ag`。
>
> 限制：Jupiter Token API 适合做**可交易性发现**与基础信息补全，不等于官方认证来源；不得仅凭 Jupiter 收录就判定“官方 token”。

### 2.2 EVM 链上数据源

#### 2.2.0 Blockscout API（Base/ETH 优先，完全免费）

**发现：** Base 和 ETH 有独立的 Blockscout 实例，提供完整的 REST API，无需 API key。

**Base Blockscout:** `https://base.blockscout.com/api/v2`
**ETH Blockscout:** `https://eth.blockscout.com/api/v2`

**使用方式：**
```python
from scripts.evm_explorer_client import EVMExplorerClient

# Base 链（优先 Blockscout）
client = EVMExplorerClient(chain="base")

# 代币信息（含价格、持有者数、市值）
token = client.token_info("0x4200000000000000000000000000000000000006")

# 代币持有者列表
holders = client.token_holders("0x4200000000000000000000000000000000000006")

# 代币转账记录
transfers = client.token_transfers("0x4200000000000000000000000000000000000006")

# 地址信息
addr = client.address_info("0x...")

# 地址交易
txs = client.address_transactions("0x...")

# 链统计
stats = client.stats()
```

**CLI 测试：**
```bash
cd ~/.claude/skills/chain-trace

# Base 代币信息
uv run python scripts/evm_explorer_client.py --chain base --token 0x4200000000000000000000000000000000000006 --method info

# Base 代币持有者
uv run python scripts/evm_explorer_client.py --chain base --token 0x4200000000000000000000000000000000000006 --method holders

# BSC 搜索（用 searchHandler）
uv run python scripts/evm_explorer_client.py --chain bsc --address 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c --method search
```

**Blockscout API 端点：**
- `/tokens/{address}` - 代币信息
- `/tokens/{address}/holders` - 持有者列表
- `/tokens/{address}/transfers` - 转账记录
- `/addresses/{address}` - 地址信息
- `/addresses/{address}/transactions` - 地址交易
- `/transactions/{hash}` - 交易详情
- `/blocks/{number}` - 区块信息
- `/stats` - 链统计

---

#### 2.2.1 Etherscan searchHandler（BSC/Base，已失效 403）

**状态：** `bscscan.com` 和 `basescan.org` 的 `/searchHandler` 端点已被 Cloudflare 拦截，返回 403。
BSC 链 fallback 到 `eth_call` RPC 查询代币元数据（symbol/name/decimals）。

---

#### 2.2.2 Ethereum (ETH) 公共 RPC

候选池（按稳定性优先，需先探测可用性）：

**Tier A（优先）**
1. `https://cloudflare-eth.com`
2. `https://ethereum-rpc.publicnode.com`
3. `https://eth.llamarpc.com`

**Tier B（公开聚合/社区节点）**
4. `https://eth.drpc.org`
5. `https://1rpc.io/eth`
6. `https://rpc.ankr.com/eth`（部分环境可能限流/未授权）

#### 2.2.2 Base 公共 RPC

候选池（按稳定性优先，需先探测可用性）：

**Tier A（优先）**
1. `https://mainnet.base.org`
2. `https://base-rpc.publicnode.com`
3. `https://base.llamarpc.com`

**Tier B（公开聚合/社区节点）**
4. `https://base.drpc.org`
5. `https://1rpc.io/base`
6. `https://rpc.ankr.com/base`（部分环境可能限流/未授权）

#### 2.2.3 BSC 公共 RPC

候选池（按稳定性优先，需先探测可用性）：

**Tier A（官方/高可用，优先）**
1. `https://bsc-dataseed.binance.org`
2. `https://bsc-dataseed1.binance.org`
3. `https://bsc-dataseed2.binance.org`
4. `https://bsc-dataseed3.binance.org`
5. `https://bsc-dataseed4.binance.org`
6. `https://bsc-dataseed.bnbchain.org`
7. `https://bsc-dataseed1.bnbchain.org`
8. `https://bsc-dataseed2.bnbchain.org`
9. `https://bsc-dataseed3.bnbchain.org`
10. `https://bsc-dataseed4.bnbchain.org`
11. `https://bsc-dataseed-public.bnbchain.org`

**Tier B（公开镜像/社区节点，作为轮换补充）**
12. `https://bsc-dataseed.defibit.io`
13. `https://bsc-dataseed1.defibit.io`
14. `https://bsc-dataseed2.defibit.io`
15. `https://bsc-dataseed3.defibit.io`
16. `https://bsc-dataseed4.defibit.io`
17. `https://bsc-dataseed.ninicoin.io`
18. `https://bsc-dataseed1.ninicoin.io`
19. `https://bsc-dataseed2.ninicoin.io`
20. `https://bsc-dataseed3.ninicoin.io`
21. `https://bsc-dataseed4.ninicoin.io`
22. `https://bsc-dataseed.nariox.org`
23. `https://bsc.nodereal.io`
24. `https://1rpc.io/bnb`

**Tier C（部分 IP/机房可能被风控，探测通过再启用）**
25. `https://bsc-rpc.publicnode.com`
26. `https://bsc.publicnode.com`
27. `https://binance.llamarpc.com`
28. `https://bsc.drpc.org`
29. `https://rpc.ankr.com/bsc`（常见未授权/需 key 场景）

> 说明：EVM 公共端点普遍存在限流；其中 BSC 官方文档明确 mainnet 公共端点存在限流，且部分端点禁用 `eth_getLogs`。高频日志拉取必须走轮换 + 降级。

常用 RPC：
```bash
# 链ID
curl -s -X POST "{evmRpc}" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'

# 钱包原生币余额（ETH/Base/BSC 分别返回对应链原生币）
curl -s -X POST "{evmRpc}" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getBalance","params":["{address}","latest"],"id":1}'

# 合约字节码（判定是否合约地址）
curl -s -X POST "{evmRpc}" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getCode","params":["{address}","latest"],"id":1}'

# ERC20 totalSupply / decimals / symbol / name
# totalSupply: 0x18160ddd
# decimals:    0x313ce567
# symbol:      0x95d89b41
# name:        0x06fdde03

# ERC20 Transfer 日志（代币转账轨迹）
curl -s -X POST "{evmRpc}" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getLogs","params":[{"fromBlock":"0x{start}","toBlock":"latest","address":"{tokenContract}","topics":["0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55aeb"]}],"id":1}'

# 常见 evmRpc 示例：
# ETH  -> https://ethereum-rpc.publicnode.com
# Base -> https://mainnet.base.org
# BSC  -> https://bsc-dataseed.binance.org
```

### 2.3 Solana 链上数据源

#### 2.3.0 Solana 公共 RPC（Solscan 逆向已不可用）

**状态：** Solscan 内部 API（`public-api.solscan.io` / `api-v2.solscan.io`）已被 Cloudflare 彻底拦截（401/403）。`free-solscan-api` 包已移除。

当前 Solana 客户端仅使用公共 RPC：

```python
from scripts.solscan_client import SolscanClient

client = SolscanClient()

# 交易详情
tx = client.transaction("57YB5kSKyBqFqLtmnzJKn3ZJuGsaMKDuJaKoZKHZJqU3...")

# 地址交易列表
txs = client.transactions("地址", page=1, page_size=40)

# 代币供应量
token = client.token_data("mint地址")

# Top 20 持有者（仅 getTokenLargestAccounts）
holders = client.token_holders("mint地址")

# 批量账户信息（getMultipleAccounts）
accounts = client.accounts_info(["addr1", "addr2", ...])
```

**CLI 测试：**
```bash
uv run python scripts/solscan_client.py --mint So11111111111111111111111111111111111111112 --method token_data
uv run python scripts/solscan_client.py --mint <代币地址> --method token_holders
uv run python scripts/solscan_client.py --addresses addr1 addr2 --method accounts_info
```

**优化：**
- `accounts_info()` 使用 `getMultipleAccounts` 批量查询，替代单次 `getAccountInfo`
- 指数退避 + jitter + RPC 轮换应对 429 限流
- 热门代币的 `getTokenLargestAccounts` 可能被公开 RPC 限流，此时 holders 返回空（graceful degradation）

---

#### 2.3.1 公共 RPC（降级备选）

候选池（按稳定性优先，需先探测可用性）：

**Tier A（优先）**
1. `https://api.mainnet-beta.solana.com`
2. `https://api.mainnet.solana.com`

**Tier B（公开聚合/社区节点）**
3. `https://solana-rpc.publicnode.com`
4. `https://solana.drpc.org`
5. `https://solana.api.onfinality.io/public`
6. `https://endpoints.omniatech.io/v1/sol/mainnet/public`

**Tier C（条件启用）**
7. `https://rpc.ankr.com/solana`（常见未授权/策略限制）

> 说明：Solana 官方公共端点有明确速率上限，不适合单点生产流量。必须多端点轮换 + 冷却窗口。

常用 RPC：
```bash
# 余额
curl -s -X POST "https://api.mainnet-beta.solana.com" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getBalance","params":["{address}"]}'

# 钱包历史签名（资金轨迹入口）
curl -s -X POST "https://api.mainnet-beta.solana.com" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getSignaturesForAddress","params":["{address}",{"limit":50}]}'

# 交易详情
curl -s -X POST "https://api.mainnet-beta.solana.com" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getTransaction","params":["{signature}",{"encoding":"jsonParsed","maxSupportedTransactionVersion":0}]}'

# 代币供应量
curl -s -X POST "https://api.mainnet-beta.solana.com" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getTokenSupply","params":["{mintAddress}"]}'
```

#### 2.3.1 多链公共 RPC 抗限流模板（必须）

```bash
# --- Endpoint pools ---
ETH_RPCS=(
  "https://cloudflare-eth.com"
  "https://ethereum-rpc.publicnode.com"
  "https://eth.llamarpc.com"
  "https://eth.drpc.org"
  "https://1rpc.io/eth"
)

BASE_RPCS=(
  "https://mainnet.base.org"
  "https://base-rpc.publicnode.com"
  "https://base.llamarpc.com"
  "https://base.drpc.org"
  "https://1rpc.io/base"
)

BSC_RPCS=(
  "https://bsc-dataseed.binance.org"
  "https://bsc-dataseed1.binance.org"
  "https://bsc-dataseed2.binance.org"
  "https://bsc-dataseed3.binance.org"
  "https://bsc-dataseed4.binance.org"
  "https://bsc-dataseed.bnbchain.org"
  "https://bsc-dataseed1.bnbchain.org"
  "https://bsc-dataseed2.bnbchain.org"
  "https://bsc-dataseed3.bnbchain.org"
  "https://bsc-dataseed4.bnbchain.org"
  "https://bsc-dataseed-public.bnbchain.org"
  "https://bsc-dataseed.defibit.io"
  "https://bsc-dataseed1.defibit.io"
  "https://bsc-dataseed2.defibit.io"
  "https://bsc-dataseed.ninicoin.io"
  "https://bsc-dataseed1.ninicoin.io"
  "https://bsc-dataseed2.ninicoin.io"
  "https://bsc-dataseed.nariox.org"
  "https://bsc.nodereal.io"
  "https://1rpc.io/bnb"
)

SOLANA_RPCS=(
  "https://api.mainnet-beta.solana.com"
  "https://api.mainnet.solana.com"
  "https://solana-rpc.publicnode.com"
)

# --- Runtime state ---
declare -A RPC_FAILS          # key: endpoint, value: consecutive failures
declare -A RPC_COOLDOWN_UNTIL # key: endpoint, value: unix ts

now_ts() { date +%s; }

retry_sleep() {
  local attempt="$1"
  local base=$((1 << attempt))
  local jitter=$((RANDOM % 2))
  local total=$((base + jitter))
  [ "$total" -gt 16 ] && total=16
  sleep "$total"
}

rpc_probe_pool() {
  # 用轻量方法探测当前 IP 下可用端点，避免一上来就被 403/1010 卡死
  local chain="$1"
  local -n pool_ref="$2"
  local method payload out

  if [ "$chain" = "solana" ]; then
    payload='{"jsonrpc":"2.0","id":1,"method":"getSlot","params":[]}'
  else
    payload='{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
  fi

  local active=()
  for rpc in "${pool_ref[@]}"; do
    out=$(curl -sS --max-time 8 -X POST "$rpc" -H "Content-Type: application/json" -d "$payload" 2>/dev/null || true)
    if echo "$out" | grep -q '"result"'; then
      active+=("$rpc")
      RPC_FAILS["$rpc"]=0
      RPC_COOLDOWN_UNTIL["$rpc"]=0
    fi
  done
  pool_ref=("${active[@]}")
}

rpc_call() {
  # rpc_call <chain> <payload_json>
  local chain="$1"
  local payload="$2"
  local -n pool_ref

  case "$chain" in
    solana) pool_ref=SOLANA_RPCS ;;
    eth)    pool_ref=ETH_RPCS ;;
    base)   pool_ref=BASE_RPCS ;;
    bsc)    pool_ref=BSC_RPCS ;;
    *)      return 1 ;;
  esac

  local now cooldown_until fails out
  for rpc in "${pool_ref[@]}"; do
    now=$(now_ts)
    cooldown_until=${RPC_COOLDOWN_UNTIL["$rpc"]:-0}
    [ "$now" -lt "$cooldown_until" ] && continue

    for attempt in 0 1 2 3; do
      [ "$attempt" -gt 0 ] && retry_sleep "$attempt"
      out=$(curl -sS --max-time 12 -X POST "$rpc" -H "Content-Type: application/json" -d "$payload" 2>/dev/null || true)

      if echo "$out" | grep -q '"result"'; then
        RPC_FAILS["$rpc"]=0
        echo "$out"
        return 0
      fi

      # 命中 429/403/风控页/未授权：快速熔断当前端点，切到下一个
      if echo "$out" | grep -Eqi '429|403|Too Many Requests|rate limit|error code: 1010|Unauthorized'; then
        fails=$(( ${RPC_FAILS["$rpc"]:-0} + 1 ))
        RPC_FAILS["$rpc"]=$fails
        RPC_COOLDOWN_UNTIL["$rpc"]=$(( now + (fails * 30) ))
        break
      fi
    done
  done

  return 1
}
```

> 说明：`getTokenLargestAccounts` 在公开 RPC 上可能 429/403。失败时按以下顺序降级：
> 0) 先执行 `rpc_probe_pool`，剔除当前 IP 下直接 403/1010 的端点；
> 1) 退避重试 + RPC 轮换；
> 2) 先用 `getTokenAccountsByOwner(owner+mint)` 验证关键官方地址是否持有目标 mint；
> 3) 再用第三方 holders 近似（Jupiter/GeckoTerminal/GoPlus）补充，且必须下调置信度。
>
> 禁止：把第三方 holders 近似数据当“链上精确 Top 持仓”。

> 实战备注（重要）：同一 RPC 对不同机房/IP 可表现不同（例如 Cloudflare 1010、区域性 403）。
> 端点是否“可用”必须以**当前执行环境探测结果**为准，而不是网络清单页面。

#### 2.3.2 Cloudflare 拦截降级（cloudscraper + uv，条件启用）

当 `curl`/普通 `requests` 频繁出现 `403`、`error code: 1010`、`1020` 时，可使用 `cloudscraper` 做**探测层降级**（仅用于公开 RPC 可用性探测）。

**依赖管理必须使用 uv（禁止 pip）**：

```bash
cd ~/.claude/skills/chain-trace

# 若项目尚未初始化（已有 pyproject.toml 可跳过）
uv init

# 添加依赖（写入 pyproject.toml + uv.lock）
uv add cloudscraper

# 锁定并同步
uv lock
uv sync --locked

# 探测 Solana / ETH / Base / BSC 公开 RPC 池
uv run python scripts/rpc_probe_cloudscraper.py --chain solana --tries 2
uv run python scripts/rpc_probe_cloudscraper.py --chain eth --tries 2
uv run python scripts/rpc_probe_cloudscraper.py --chain base --tries 2
uv run python scripts/rpc_probe_cloudscraper.py --chain bsc --tries 2
```

判定规则：
- `status=active`：可进入轮询池；
- `status=blocked`：进入冷却或直接剔除；
- `status=network_error/unknown`：不直接判死，降低优先级并等待下一轮探测。

边界与限制（必须声明）：
- `cloudscraper` 是 best-effort，不保证绕过所有 Cloudflare/WAF 规则；
- 仅用于合法公开接口取证，必须遵守目标站 ToS 与当地法律；
- 不得用于登录态、付费墙、隐私数据、验证码破解等越权场景。

### 2.4 安全与合约风险（无 key）

1) **GoPlus EVM（ETH / Base / BSC）**
```bash
curl -s "https://api.gopluslabs.io/api/v1/token_security/1?contract_addresses={ethTokenAddress}"
curl -s "https://api.gopluslabs.io/api/v1/token_security/8453?contract_addresses={baseTokenAddress}"
curl -s "https://api.gopluslabs.io/api/v1/token_security/56?contract_addresses={bscTokenAddress}"
```

2) **GoPlus Solana**
```bash
curl -s "https://api.gopluslabs.io/api/v1/solana/token_security?contract_addresses={solMint}"
```

3) **Honeypot.is（EVM）**
```bash
curl -s "https://api.honeypot.is/v2/IsHoneypot?address={tokenAddress}&chainID=1"
curl -s "https://api.honeypot.is/v2/IsHoneypot?address={tokenAddress}&chainID=8453"
curl -s "https://api.honeypot.is/v2/IsHoneypot?address={tokenAddress}&chainID=56"
```

---
