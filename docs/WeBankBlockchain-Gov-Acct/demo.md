# 组件使用demo
## 合约集成demo

### 存证demo

存证合约的demo
```
pragma solidity ^0.4.25;

import "./AccountManager.sol";

contract EvidenceDemo {
    struct EvidenceData {
        string hash;
        address owner;
        uint256 timestamp;
    }
    address public _owner;
    mapping(string => EvidenceData) private _evidences;
    // import AccountManager
    AccountManager _accountManager;

    constructor(address accountManager) public {
        // import accountManager
        _accountManager = AccountManager(accountManager);
        // set user account instead of external account
        _owner = _accountManager.getAccount(msg.sender);
        require(_owner != 0x0, "Invalid account!");
    }

    modifier onlyOwner() {
        // get user account by external account
        address userAccountAddress = _accountManager.getAccount(msg.sender);
        require(userAccountAddress == _owner, "Not admin");
        _;
    }

    function setData(
        string hash,
        address owner,
        uint256 timestamp
    ) public onlyOwner {
        _evidences[hash].hash = hash;
        _evidences[hash].owner = owner;
        _evidences[hash].timestamp = timestamp;
    }

    function getData(string hash)
        public
        view
        returns (
            string,
            address,
            uint256
        )
    {
        EvidenceData storage evidence = _evidences[hash];
        return (evidence.hash, evidence.owner, evidence.timestamp);
    }
}

```


存证合约的demo在控制台的部署指令

```
=============================================================================================
Welcome to FISCO BCOS console(1.0.9)!
Type 'help' or 'h' for help. Type 'quit' or 'q' to quit console.
 ________ ______  ______   ______   ______       _______   ______   ______   ______
|        |      \/      \ /      \ /      \     |       \ /      \ /      \ /      \
| $$$$$$$$\$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\    | $$$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\
| $$__     | $$ | $$___\$| $$   \$| $$  | $$    | $$__/ $| $$   \$| $$  | $| $$___\$$
| $$  \    | $$  \$$    \| $$     | $$  | $$    | $$    $| $$     | $$  | $$\$$    \
| $$$$$    | $$  _\$$$$$$| $$   __| $$  | $$    | $$$$$$$| $$   __| $$  | $$_\$$$$$$\
| $$      _| $$_|  \__| $| $$__/  | $$__/ $$    | $$__/ $| $$__/  | $$__/ $|  \__| $$
| $$     |   $$ \\$$    $$\$$    $$\$$    $$    | $$    $$\$$    $$\$$    $$\$$    $$
 \$$      \$$$$$$ \$$$$$$  \$$$$$$  \$$$$$$      \$$$$$$$  \$$$$$$  \$$$$$$  \$$$$$$

=============================================================================================
[group:1]> deploy EvidenceDemo "0x08ba44f04df9671a2fc298756bc06f1dbca56e11"
revert instruction

[group:1]> deploy AdminGovernBuilder
contract address: 0xc00504c8dad8f75d08ebceda4f7c7b1d13b327d5

[group:1]> call AdminGovernBuilder 0xc00504c8dad8f75d08ebceda4f7c7b1d13b327d5 _governance
0x24c327f0bf851a936c5049b019142709067b711d

[group:1]> call WEGovernance 0x24c327f0bf851a936c5049b019142709067b711d getAccountManager
0x6e869333a1e3dc83501723b7dcb624b09c1757e3

[group:1]>  deploy EvidenceDemo "0x6e869333a1e3dc83501723b7dcb624b09c1757e3"
contract address: 0x99276281a782a199d1998ce7a56927d99dcd6c6a

[group:1]> call EvidenceDemo 0x99276281a782a199d1998ce7a56927d99dcd6c6a setData "hash" "0x6e869333a1e3dc83501723b7dcb624b09c1757e3" 1
transaction hash: 0xc7f2531ca9e968a97a6035a4fbfa79f2a0cda7adfb4b9f878e36e4a879fa5249

[group:1]> call EvidenceDemo 0x99276281a782a199d1998ce7a56927d99dcd6c6a getData "hash"
[hash, 0x6e869333a1e3dc83501723b7dcb624b09c1757e3, 1]

```

### 转账demo

转账合约的demo

```
pragma solidity ^0.4.25;


library LibSafeMath {   
    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b <= a, "SafeMath: subtraction overflow");
        uint256 c = a - b;

        return c;
    }
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");

        return c;
    }
}


pragma solidity ^0.4.25;

import "./LibSafeMath.sol";
import "./AccountManager.sol";


contract TransferDemo {
    using LibSafeMath for uint256;
    mapping(address => uint256) private _balances;
    // import AccountManager
    AccountManager _accountManager;

    constructor(address accountManager, uint256 initBalance) public {
        // import accountManager
        _accountManager = AccountManager(accountManager);
        address owner = _accountManager.getAccount(msg.sender);
        _balances[owner] = initBalance;
    }

    modifier validateAccount(address addr) {
        require(
            // predicate account status
            _accountManager.isExternalAccountNormal(addr),
            "Account is abnormal!"
        );
        _;
    }

    modifier checkTargetAccount(address sender) {
        require(
            msg.sender != sender && sender != address(0),
            "Can't transfer to illegal address!"
        );
        _;
    }

    function balance(address owner) public view returns (uint256) {
        // 1.get account by external account, 2.get balace by account.
        return _balances[_accountManager.getAccount(owner)];
    }

    function transfer(address toAddress, uint256 value)
        public
        // validate source & target account
        validateAccount(msg.sender)
        validateAccount(toAddress)
        checkTargetAccount(toAddress)
        returns (bool)
    {
        // 1. get source account
        address fromAccount = _accountManager.getAccount(msg.sender);
        // 2. sub the balance of source account
        uint256 balanceOfFrom = _balances[fromAccount].sub(value);
        // 3. modify the balance of source account
        _balances[fromAccount] = balanceOfFrom;
        // 4. get target account
        address toAccount = _accountManager.getAccount(toAddress);
        // 5. add balance of target account
        uint256 balanceOfTo = _balances[toAccount].add(value);
        // set the new balance of target account
        _balances[toAccount] = balanceOfTo;
        return true;
    }
}

```

转账合约的demo在控制台的部署指令

```
=============================================================================================
Welcome to FISCO BCOS console(1.0.9)!
Type 'help' or 'h' for help. Type 'quit' or 'q' to quit console.
 ________ ______  ______   ______   ______       _______   ______   ______   ______
|        |      \/      \ /      \ /      \     |       \ /      \ /      \ /      \
| $$$$$$$$\$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\    | $$$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\
| $$__     | $$ | $$___\$| $$   \$| $$  | $$    | $$__/ $| $$   \$| $$  | $| $$___\$$
| $$  \    | $$  \$$    \| $$     | $$  | $$    | $$    $| $$     | $$  | $$\$$    \
| $$$$$    | $$  _\$$$$$$| $$   __| $$  | $$    | $$$$$$$| $$   __| $$  | $$_\$$$$$$\
| $$      _| $$_|  \__| $| $$__/  | $$__/ $$    | $$__/ $| $$__/  | $$__/ $|  \__| $$
| $$     |   $$ \\$$    $$\$$    $$\$$    $$    | $$    $$\$$    $$\$$    $$\$$    $$
 \$$      \$$$$$$ \$$$$$$  \$$$$$$  \$$$$$$      \$$$$$$$  \$$$$$$  \$$$$$$  \$$$$$$

=============================================================================================
[group:1]> deploy AdminGovernBuilder
contract address: 0xd17a5cdcb5e4b1cc3ce67c71c8e0e1dd07f33914

[group:1]> call AdminGovernBuilder 0xd17a5cdcb5e4b1cc3ce67c71c8e0e1dd07f33914 _governance
0xda80ff2d1cd498c86439cf52c1b1a8bb01da6fbc

[group:1]> call WEGovernance 0xda80ff2d1cd498c86439cf52c1b1a8bb01da6fbc getAccountManager
0x199a2b9f43415f1f5ca9da6dc7c3dc124c531fd5

[group:1]> deploy TransferDemo "0x199a2b9f43415f1f5ca9da6dc7c3dc124c531fd5" 1000
contract address: 0x7873756b7a93afed89482040257d332e3fc72336

[group:1]> call TransferDemo 0x7873756b7a93afed89482040257d332e3fc72336 transfer "0x1" 1
The execution of the contract rolled back.

[group:1]> call AccountManager 0x199a2b9f43415f1f5ca9da6dc7c3dc124c531fd5 newAccount "0x1"
transaction hash: 0xf5a806396256e01295d014edf5688afdc0e7c1f4363c47f5bd083a14f1398192
---------------------------------------------------------------------------------------------
Output
function: newAccount(address)
return type: (bool, address)
return value: (true, 0x941d587493454784874e7d463dc76368f20bd3ff)
---------------------------------------------------------------------------------------------
Event logs
event signature: LogSetOwner(address,address) index: 0
event value: (0x0000000000000000000000000000000000000001, 0x941d587493454784874e7d463dc76368f20bd3ff)
event signature: LogManageNewAccount(address,address,address) index: 0
event value: (0x0000000000000000000000000000000000000001, 0x941d587493454784874e7d463dc76368f20bd3ff, 0x199a2b9f43415f1f5ca9da6dc7c3dc124c531fd5)
---------------------------------------------------------------------------------------------

[group:1]> call TransferDemo 0x7873756b7a93afed89482040257d332e3fc72336 transfer "0x1" 1
transaction hash: 0xdbe329df31c18fd54b87139045db1fe2d0358c54139f1b2b649fb730c9a33420
---------------------------------------------------------------------------------------------
Output
function: transfer(address,uint256)
return type: (bool)
return value: (true)
---------------------------------------------------------------------------------------------

[group:1]> call TransferDemo 0x7873756b7a93afed89482040257d332e3fc72336 balance "0x1"
1

[group:1]>

```
## SDK集成Demo

提供了SDK集成和使用的demo，展示了如何使用Java SDK。 详情可参考 [Gov-Acct-Demo](https://github.com/WeBankBlockchain/Gov-Acct-Demo/)


## 测试代码说明

### 链配置

打开src/main/application.properties，修改链配置信息。

```
## 机构ID
system.orgId=org1
## 链的ip端口，多个节点使用;分隔
system.nodeStr=[ip]:[channel_port]
## 群组ID
system.groupId=1
```

### 自动运行

```
./gradlew test
```
<br />可查看自动化测试的运行结果报告。<br />

