WETH 是包装 ETH 主币，作为 ERC20 的合约。
标准的 ERC20 合约包括如下几个
- 3 个查询
  - balanceOf: 查询指定地址的 Token 数量
  - allowance: 查询指定地址对另外一个地址的剩余授权额度
  - totalSupply: 查询当前合约的 Token 总量
- 2 个交易
  - transfer: 从当前调用者地址发送指定数量的 Token 到指定地址。
    - 这是一个写入方法，所以还会抛出一个 Transfer 事件。
  - transferFrom: 当向另外一个合约地址存款时，对方合约必须调用 transferFrom 才可以把 Token 拿到它自己的合约中。
- 2 个事件
  - Transfer
  - Approval
- 1 个授权
  - approve: 授权指定地址可以操作调用者的最大 Token 数量。
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;
contract WETH {
    string public name = "Wrapped Ether";
    string public symbol = "WETH";
    uint8 public decimals = 18;
    event Approval(address indexed src, address indexed delegateAds, uint256 amount);
    event Transfer(address indexed src, address indexed toAds, uint256 amount);
    event Deposit(address indexed toAds, uint256 amount);
    event Withdraw(address indexed src, uint256 amount);
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    function deposit() public payable {
        balanceOf[msg.sender] += msg.value;
        emit Deposit(msg.sender, msg.value);
    }
    function withdraw(uint256 amount) public {
        require(balanceOf[msg.sender] >= amount);
        balanceOf[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
        emit Withdraw(msg.sender, amount);
    }
    function totalSupply() public view returns (uint256) {
        return address(this).balance;
    }
    function approve(address delegateAds, uint256 amount) public returns (bool) {
        allowance[msg.sender][delegateAds] = amount;
        emit Approval(msg.sender, delegateAds, amount);
        return true;
    }
    function transfer(address toAds, uint256 amount) public returns (bool) {
        return transferFrom(msg.sender, toAds, amount);
    }
    function transferFrom(
        address src,
        address toAds,
        uint256 amount
    ) public returns (bool) {
        require(balanceOf[src] >= amount);
        if (src != msg.sender) {
            require(allowance[src][msg.sender] >= amount);
            allowance[src][msg.sender] -= amount;
        }
        balanceOf[src] -= amount;
        balanceOf[toAds] += amount;
        emit Transfer(src, toAds, amount);
        return true;
    }
    fallback() external payable {
        deposit();
    }
    receive() external payable {
        deposit();
    }
}
```

添加注释后的代码
```solidity
// SPDX-License-Identifier: MIT
// 声明合约使用MIT开源许可证

pragma solidity ^0.8.17;
// 指定编译器版本，支持0.8.17及以上但低于0.9.0的版本

/**
 * @title Wrapped Ether (WETH) 合约
 * @dev 实现以太坊原生代币ETH的ERC20包装版本
 * @notice 允许用户将ETH转换为ERC20标准代币进行更灵活的操作
 */
contract WETH {
    // 代币基本信息
    string public name = "Wrapped Ether";  // 代币名称：Wrapped Ether
    string public symbol = "WETH";         // 代币符号：WETH
    uint8 public decimals = 18;            // 小数位数：18位，与ETH保持一致

    // 事件声明 - 用于区块链日志记录
    event Approval(address indexed src, address indexed delegateAds, uint256 amount);
    // 批准事件：记录授权地址(delegateAds)可动用src地址的代币数量
    event Transfer(address indexed src, address indexed toAds, uint256 amount);
    // 转账事件：记录从src到toAds的代币转移
    event Deposit(address indexed toAds, uint256 amount);
    // 存款事件：记录ETH存入并转换为WETH的操作
    event Withdraw(address indexed src, uint256 amount);
    // 取款事件：记录WETH兑换回ETH的操作

    // 存储结构
    mapping(address => uint256) public balanceOf;
    // 余额映射：记录每个地址持有的WETH数量
    mapping(address => mapping(address => uint256)) public allowance;
    // 授权映射：记录address1允许address2动用的WETH数量

    /**
     * @dev 存款函数 - 将ETH存入合约并生成等额WETH
     * @notice 调用时需随交易发送ETH
     */
    function deposit() public payable {
        balanceOf[msg.sender] += msg.value;  // 增加用户WETH余额
        emit Deposit(msg.sender, msg.value); // 触发存款事件
    }

    /**
     * @dev 取款函数 - 将WETH兑换回ETH
     * @param amount 要兑换的WETH数量（单位：wei）
     */
    function withdraw(uint256 amount) public {
        require(balanceOf[msg.sender] >= amount); // 检查余额是否足够
        balanceOf[msg.sender] -= amount;          // 减少用户WETH余额
        payable(msg.sender).transfer(amount);     // 向用户发送ETH
        emit Withdraw(msg.sender, amount);        // 触发取款事件
    }

    /**
     * @dev 获取总供应量
     * @return 返回合约当前持有的ETH总量，即WETH总供应量
     */
    function totalSupply() public view returns (uint256) {
        return address(this).balance; // 总供应量等于合约持有的ETH余额
    }

    /**
     * @dev 授权函数 - 允许其他地址动用自己的代币
     * @param delegateAds 被授权地址
     * @param amount 授权额度
     * @return 始终返回true（符合ERC20标准）
     */
    function approve(address delegateAds, uint256 amount) public returns (bool) {
        allowance[msg.sender][delegateAds] = amount; // 设置授权额度
        emit Approval(msg.sender, delegateAds, amount); // 触发授权事件
        return true;
    }

    /**
     * @dev 转账函数 - 从自己账户转出代币
     * @param toAds 接收地址
     * @param amount 转账金额
     * @return 始终返回true（符合ERC20标准）
     */
    function transfer(address toAds, uint256 amount) public returns (bool) {
        return transferFrom(msg.sender, toAds, amount); // 调用transferFrom实现
    }

    /**
     * @dev 授权转账函数 - 从其他已授权账户转出代币
     * @param src 源地址
     * @param toAds 目标地址
     * @param amount 转账金额
     * @return 始终返回true（符合ERC20标准）
     */
    function transferFrom(
        address src,
        address toAds,
        uint256 amount
    ) public returns (bool) {
        require(balanceOf[src] >= amount); // 检查源地址余额
        if (src != msg.sender) { // 如果不是自己转自己
            require(allowance[src][msg.sender] >= amount); // 检查授权额度
            allowance[src][msg.sender] -= amount; // 扣除授权额度
        }
        balanceOf[src] -= amount; // 减少源地址余额
        balanceOf[toAds] += amount; // 增加目标地址余额
        emit Transfer(src, toAds, amount); // 触发转账事件
        return true;
    }

    /**
     * @dev 回退函数 - 处理未匹配到函数选择器的调用
     * @notice 当向合约发送ETH且不带data时触发
     */
    fallback() external payable {
        deposit(); // 自动调用存款函数
    }

    /**
     * @dev 接收ETH的函数 - 专门处理纯ETH转账
     * @notice 当向合约发送ETH且不带data时触发
     */
    receive() external payable {
        deposit(); // 自动调用存款函数
    }
}
```