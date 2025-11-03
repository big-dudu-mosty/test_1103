# CCTP 包装合约部署指南

## 📋 前置准备

### 1. 安装 Aptos CLI

```bash
# 使用官方脚本安装
curl -fsSL "https://aptos.dev/scripts/install_cli.py" | python3

# 验证安装
aptos --version
# 预期输出：aptos 3.x.x
```

### 2. 初始化 Aptos 账户

```bash
cd /home/su/dome/move/all/Saffron-contract/cctp-wrapper

# 初始化配置
aptos init --network testnet

# 按提示操作：
# 1. 选择网络: testnet
# 2. 选择创建新密钥: Y
# 3. 记录下显示的账户地址（这将是你的合约地址）
```

输出示例：
```
Configuring for profile default
Choose network from [devnet, testnet, mainnet, local, custom | defaults to devnet]
testnet
Account 0xabc123def456... doesn't exist, creating it and funding it with 100000000 Octas
Account 0xabc123def456... funded successfully

---
Aptos CLI is now set up for account 0xabc123def456... as profile default!
```

**🎯 重要：记录你的账户地址，这就是合约部署地址！**

### 3. 领取测试 APT

```bash
# 方式 1: 使用 CLI
aptos account fund-with-faucet --account default

# 方式 2: 使用网页水龙头
# 访问: https://aptoslabs.com/testnet-faucet
# 输入你的地址并点击领取
```

验证余额：
```bash
aptos account list --account default
# 应该看到 APT 余额 >= 1.0
```

---

## 🚀 部署步骤

### 步骤 1: 验证文件结构

```bash
cd /home/su/dome/move/all/Saffron-contract/cctp-wrapper

# 检查文件是否都存在
ls -la
# 应该看到:
# - Move.toml
# - sources/cctp_wrapper.move

tree
```

预期结构：
```
cctp-wrapper/
├── Move.toml
└── sources/
    └── cctp_wrapper.move
```

### 步骤 2: 编译合约

```bash
cd /home/su/dome/move/all/Saffron-contract/cctp-wrapper

# 编译合约
aptos move compile
```

预期输出：
```
Compiling, may take a little while to download git dependencies...
UPDATING GIT DEPENDENCY https://github.com/aptos-labs/aptos-core.git
INCLUDING DEPENDENCY AptosFramework
INCLUDING DEPENDENCY AptosStdlib
INCLUDING DEPENDENCY MoveStdlib
BUILDING cctp_wrapper
{
  "Result": [
    "0xYOUR_ADDRESS::wrapper"
  ]
}
```

✅ 如果看到 `"Result"` 且没有错误，说明编译成功！

❌ 如果有编译错误，请检查：
- Move.toml 配置是否正确
- cctp_wrapper.move 语法是否正确
- 网络连接是否正常（需要下载依赖）

### 步骤 3: 测试编译（可选但推荐）

```bash
# 运行 Move 编译器测试
aptos move test
```

### 步骤 4: 部署合约到测试网

```bash
cd /home/su/dome/move/all/Saffron-contract/cctp-wrapper

# 部署合约（--assume-yes 跳过确认）
aptos move publish --assume-yes
```

预期输出：
```
Compiling, may take a little while to download git dependencies...
BUILDING cctp_wrapper
package size 2345 bytes
Do you want to submit a transaction for a range of [123400 - 185100] Octas at a gas unit price of 100 Octas? [yes/no] >
yes
Transaction submitted: 0x9876543210abcdef...
{
  "Result": {
    "transaction_hash": "0x9876543210abcdef...",
    "gas_used": 1234,
    "gas_unit_price": "100",
    "sender": "0xabc123def456...",
    "sequence_number": "0",
    "success": true,
    "timestamp_us": "1699876543210000",
    "version": "123456789",
    "vm_status": "Executed successfully"
  }
}
```

**🎉 看到 `"success": true` 就说明部署成功了！**

### 步骤 5: 记录合约地址

```bash
# 从配置文件中提取合约地址
cat .aptos/config.yaml | grep account

# 输出示例：
# account: 0xabc123def456789abc123def456789abc123def456789abc123def456789abcd
```

**⚠️ 复制这个地址，下一步需要配置到前端！**

---

## 🔍 验证部署

### 验证 1: 调用 view 函数

```bash
# 获取你的合约地址
CONTRACT_ADDRESS=$(cat .aptos/config.yaml | grep account | cut -d: -f2 | tr -d ' ')

# 调用 get_version 函数验证
aptos move view \
  --function-id ${CONTRACT_ADDRESS}::wrapper::get_version
```

预期输出：
```json
{
  "Result": [
    "1"
  ]
}
```

✅ 看到 `"1"` 说明合约部署成功且可以调用！

### 验证 2: 在浏览器查看

```bash
# 生成浏览器 URL
CONTRACT_ADDRESS=$(cat .aptos/config.yaml | grep account | cut -d: -f2 | tr -d ' ')
echo "https://explorer.aptoslabs.com/account/${CONTRACT_ADDRESS}?network=testnet"
```

在浏览器打开这个 URL，你应该看到：
- ✅ Account 页面显示你的地址
- ✅ Modules 标签显示 `wrapper` 模块
- ✅ Functions 下有 `receive_usdc` 和 `get_version` 函数

### 验证 3: 检查模块

```bash
aptos account list --account default
```

在输出中查找 `move_modules`，应该看到你的 `wrapper` 模块。

---

## ⚙️ 前端配置

### 步骤 6: 更新前端配置文件

```bash
# 1. 获取合约地址
CONTRACT_ADDRESS=$(cat .aptos/config.yaml | grep account | cut -d: -f2 | tr -d ' ')
echo "你的合约地址: ${CONTRACT_ADDRESS}"

# 2. 打开前端配置文件
nano /home/su/dome/move/all/Saffron/constants/contracts.ts
# 或
code /home/su/dome/move/all/Saffron/constants/contracts.ts
```

在 `contracts.ts` 中，将 `cctpWrapper` 的值替换为你的合约地址：

```typescript
export const APTOS_TESTNET_CONTRACTS = {
  messageTransmitter: '0x081e86cebf457a0c6004f35bd648a2794698f52e0dde09a48619dcd3d4cc23d9',
  tokenMessenger: '0x5f9b937419dda90aa06c1836b7847f65bbbe3f1217567758dc2488be31a477b9',
  usdc: '0x69091fbab5f7d635ee7ac5098cf0c1efbe31d68fec0f2cd565e8d168daf52832',
  
  // 🎯 替换为你的合约地址
  cctpWrapper: '0xabc123def456789abc123def456789abc123def456789abc123def456789abcd',
};
```

保存文件后，前端配置就完成了！

---

## 🧪 测试部署

### 快速测试脚本

创建测试文件 `test-wrapper.sh`：

```bash
#!/bin/bash

echo "🧪 测试 CCTP 包装合约部署"
echo "================================"

# 获取合约地址
CONTRACT_ADDRESS=$(cat .aptos/config.yaml | grep account | cut -d: -f2 | tr -d ' ')

if [ -z "$CONTRACT_ADDRESS" ]; then
  echo "❌ 错误：无法获取合约地址"
  exit 1
fi

echo "✅ 合约地址: $CONTRACT_ADDRESS"
echo ""

# 测试 1: 调用 view 函数
echo "📋 测试 1: 调用 get_version"
aptos move view --function-id ${CONTRACT_ADDRESS}::wrapper::get_version

if [ $? -eq 0 ]; then
  echo "✅ 测试 1 通过"
else
  echo "❌ 测试 1 失败"
  exit 1
fi

echo ""

# 测试 2: 生成浏览器链接
echo "📋 测试 2: 浏览器验证链接"
echo "https://explorer.aptoslabs.com/account/${CONTRACT_ADDRESS}?network=testnet"
echo "✅ 请在浏览器中打开上面的链接验证"

echo ""
echo "🎉 所有测试完成！"
echo ""
echo "📝 下一步:"
echo "1. 复制合约地址: $CONTRACT_ADDRESS"
echo "2. 更新前端配置: /home/su/dome/move/all/Saffron/constants/contracts.ts"
echo "3. 在浏览器中验证部署"
```

运行测试：

```bash
chmod +x test-wrapper.sh
./test-wrapper.sh
```

---

## 🔧 故障排除

### 问题 1: 编译失败 - 依赖下载失败

```
Error: Failed to download git dependency
```

**解决方案：**
```bash
# 检查网络连接
ping github.com

# 手动清理并重试
rm -rf build/
aptos move clean
aptos move compile
```

### 问题 2: 余额不足

```
Error: Insufficient balance to pay gas
```

**解决方案：**
```bash
# 再次领取测试币
aptos account fund-with-faucet --account default

# 检查余额
aptos account list --account default
```

### 问题 3: 合约已存在

```
Error: LINKER_ERROR: Duplicate module
```

**解决方案：**
如果你之前部署过，需要修改 `Move.toml` 中的包名，或使用新账户。

```bash
# 创建新账户
aptos init --profile wrapper2 --network testnet
aptos move publish --profile wrapper2 --assume-yes
```

### 问题 4: Circle CCTP 合约地址错误

```
Error: MODULE_NOT_FOUND
```

**解决方案：**
检查 `Move.toml` 中的 Circle CCTP 合约地址是否正确：

```toml
message_transmitter = "0x081e86cebf457a0c6004f35bd648a2794698f52e0dde09a48619dcd3d4cc23d9"
token_messenger = "0x5f9b937419dda90aa06c1836b7847f65bbbe3f1217567758dc2488be31a477b9"
```

---

## 📚 参考资料

- Aptos CLI 文档: https://aptos.dev/tools/aptos-cli/
- Aptos Move 文档: https://aptos.dev/move/move-on-aptos/
- Circle CCTP 文档: https://developers.circle.com/stablecoins/docs/cctp-getting-started
- Aptos 测试网浏览器: https://explorer.aptoslabs.com/?network=testnet
- Aptos 测试网水龙头: https://aptoslabs.com/testnet-faucet

---

## ✅ 检查清单

部署完成后，请确认以下事项：

```
□ Aptos CLI 已安装
□ 测试账户已创建且有 APT 余额
□ 合约编译成功
□ 合约部署成功
□ get_version 调用成功返回 1
□ 浏览器中可以看到合约模块
□ 合约地址已记录
□ 前端配置文件已更新
□ 前端应用可以正常启动
```

全部勾选后，你就可以开始测试跨链功能了！🎉

---

## 🎯 下一步

合约部署完成后：

1. **更新前端配置** - 将合约地址填入 `contracts.ts`
2. **启动前端应用** - `npm run dev` 或 `expo start`
3. **准备测试环境**:
   - MetaMask 连接 Base Sepolia，领取测试 USDC
   - Petra 连接 Aptos Testnet，领取测试 APT
4. **执行小额跨链测试** - 转账 1 USDC 测试整个流程
5. **验证结果** - 检查 Aptos 账户是否收到 USDC

祝部署顺利！🚀

