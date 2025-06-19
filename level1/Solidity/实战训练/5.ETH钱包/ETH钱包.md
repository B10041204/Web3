这一个实战主要是加深大家对 3 个取钱方法的使用。
- 任何人都可以发送金额到合约
- 只有 owner 可以取款
- 3 种取钱方式
```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;
contract EtherWallet {
    address payable public immutable owner;
    event Log(string funName, address from, uint256 value, bytes data);
    constructor() {
        owner = payable(msg.sender);
    }
    receive() external payable {
        emit Log("receive", msg.sender, msg.value, "");
    }
    function withdraw1() external {
        require(msg.sender == owner, "Not owner");
        // owner.transfer 相比 msg.sender 更消耗Gas
        // owner.transfer(address(this).balance);
        payable(msg.sender).transfer(100);
    }
    function withdraw2() external {
        require(msg.sender == owner, "Not owner");
        bool success = payable(msg.sender).send(200);
        require(success, "Send Failed");
    }
    function withdraw3() external {
        require(msg.sender == owner, "Not owner");
        (bool success, ) = msg.sender.call{value: address(this).balance}("");
        require(success, "Call Failed");
    }
    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }
}
```

添加注释后的内容

```js
// SPDX-License-Identifier: MIT
// 声明 Solidity 编译器版本，要求 >=0.8.17 且 <0.9.0
pragma solidity ^0.8.17;

/**
 * @title EtherWallet
 * @dev 以太坊钱包合约，实现资金存储、提取和余额查询功能
 */
contract EtherWallet {
    // 钱包所有者地址，部署时固定且仅能由所有者操作
    address payable public immutable owner;
    // 事件日志，记录资金操作信息（函数名、转账方、金额、附加数据）
    event Log(string funName, address from, uint256 value, bytes data);

    /**
     * @dev 构造函数，合约部署时执行
     * 功能：将部署者地址设置为钱包所有者
     */
    constructor() {
        owner = payable(msg.sender);
    }

    /**
     * @dev 接收以太币的默认函数（无函数名）
     * 触发条件：合约收到ETH时自动调用
     * 功能：记录接收事件
     */
    receive() external payable {
        emit Log("receive", msg.sender, msg.value, "");
    }

    /**
     * @dev 提取以太币的第一种方式（使用transfer）
     * 限制：仅所有者可调用
     * 特点：transfer会消耗2300 gas，失败时自动回滚
     */
    function withdraw1() external {
        require(msg.sender == owner, "Not owner"); // 验证调用者是否为所有者
        // 注释说明：使用owner.transfer会比msg.sender更消耗gas（示例中未启用）
        // owner.transfer(address(this).balance);
        payable(msg.sender).transfer(100); // 提取100 wei到所有者地址
    }

    /**
     * @dev 提取以太币的第二种方式（使用send）
     * 限制：仅所有者可调用
     * 特点：send返回布尔值，失败时不回滚但消耗2300 gas
     */
    function withdraw2() external {
        require(msg.sender == owner, "Not owner"); // 验证调用者身份
        bool success = payable(msg.sender).send(200); // 发送200 wei并获取执行结果
        require(success, "Send Failed"); // 若发送失败则回滚交易
    }

    /**
     * @dev 提取以太币的第三种方式（使用call）
     * 限制：仅所有者可调用
     * 特点：call可自定义gas，返回布尔值，失败时回滚
     */
    function withdraw3() external {
        require(msg.sender == owner, "Not owner"); // 验证所有者身份
        // 调用msg.sender地址并转移合约所有余额，data为空字节
        (bool success, ) = msg.sender.call{value: address(this).balance}("");
        require(success, "Call Failed"); // 检查调用是否成功
    }

    /**
     * @dev 查询合约当前以太币余额
     * 返回值：合约地址的ETH余额（单位：wei）
     */
    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }
}
```