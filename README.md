# ERC-3525  WTF

## `EIP3525`[](https://www.wtf.academy/solidity-application/ERC1155/#eip1155)

`ERC3525`集结了`ERC20`、`ERC721`、`ERC1155`三个代币标准的特征，属于一种新的基石性通用标准半匀质化通证（SFT），不但具有动态变化的能力，而且像 ERC-20 一样可以计算，像账户一样能够接受、存储、发送和编程数字资产，特别适合表达复杂的数字资产，如金融票据、积分卡、真实世界资产（RWA）等等。假设我们要在以太坊上发行一套结构化金融产品，这就需要一个通用容器性质的标准对金融产品进行表达，实现现金流的可编程，并把非标准的资产变成标准化的资产并实现可拆分组合。

简单来说，在`ERC20`中，每个代币都是同质化的，`value`对应着代币的数量；在`ERC721`中，每个代币都是非同质化的，每个`tokenId`作为唯一标识只对应一个代币；而在`ERC1155`中，每一种代币可以用`id`作为唯一标识表示非同质化，每个`id`下可以用`value`对应`id`下同质化代币的数量。

而在`ERC3525`中，相比于`ERC1155`的`id`和`value`两层数据结构，`ERC3525`有三层数据结构：`slot`、`id`、`value`；`slot`代表着一个类，每个类（不同的`slot`）下面的`id`都是非同质化的；`id`代表着一个类下的“容器”，而“容器”就是`ERC3525`特色之一，“容器”拥有账户的属性，可以接受、存储、发送和编程数字资产，还可以实现账户的`id`到`id`之间的转账，处在同一个`slot`下不同的`id`之间可以实现拆分组合，而不同`slot`下的`id`无法进行交互（主要根据需求设计）；而每个`id`下有一个`value`来表达同质化的代币数量，类似于`ERC20`的「余额」 属性。

这样，ERC3525就可以实现在同一个合约里管理多种复杂类型的数字资产，并且每个合约有三个网址`uri`来存储它的元数据，除了`ERC3525`的`contractURI`和`slotURI`，由于兼容`ERC721`，`ERC3525`还支持`ERC721`的`tokenURI`。

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.1;

import "../IERC3525.sol";
import "./IERC721Enumerable.sol";
/**

- @title ERC-3525 半可替换代币标准，可选扩展用于slot枚举
- @dev 任何希望支持枚举slots以及具有相同slot的tokens的合约的接口。
- 参见 [https://eips.ethereum.org/EIPS/eip-3525](https://eips.ethereum.org/EIPS/eip-3525)
- 注意：此接口的ERC-165标识符为 0x3b741b9e。
*/
interface IERC3525SlotEnumerable is IERC3525, IERC721Enumerable {
    
    /**
    
    - @notice 获取合约存储的slots的总量。
    - @return slots的总量
    */
    function slotCount() external view returns (uint256);
    
    /**
    
    - @notice 获取合约存储的所有slots中指定索引处的slot。
    - @param _index slot列表中的索引
    - @return 所有slots中`index`处的slot。
    */
    function slotByIndex(uint256 _index) external view returns (uint256);
    
    /**
    
    - @notice 获取具有相同slot的tokens的总量。
    - @param _slot 用于查询token供应量的slot
    - @return 具有指定`_slot`的tokens的总量
    */
    function tokenSupplyInSlot(uint256 _slot) external view returns (uint256);
    
    /**
    
    - @notice 获取具有相同slot的所有tokens中指定索引处的token。
    - @param _slot 用于查询tokens的slot
    - @param _index slot的token列表中的索引
    - @return 具有`_slot`的所有tokens中`_index`处的token ID
    */
    function tokenInSlotByIndex(uint256 _slot, uint256 _index) external view returns (uint256);
    }

下面是`ERC3525`的元数据接口合约`IERC3525Metadata`：

```solidity
/**
 * @title ERC-3525半可替代令牌标准，元数据的可选扩展
 * @dev 为任何希望支持查询ERC-3525合约统一资源标识符（URI）以及指定槽位的合约提供接口。
 */
interface IERC3525Metadata is IERC3525, IERC721Metadata {
		/**
		* @notice 返回当前ERC-3525合约的统一资源标识符（URI）。
		* @dev 此函数应返回以JSON格式表示的此合约的URI，以`data:application/json;`开头。
		* 有关合约URI的JSON模式，请参见https://eips.ethereum.org/EIPS/eip-3525。
		* @return 当前ERC-3525合约的JSON格式化URI
		*/
    function contractURI() external view returns (string memory);
		/**
		* @notice 返回指定slot的统一资源标识符（URI）。
		* @dev 此函数应返回以JSON格式表示的`_slot`的URI，以`data:application/json;`开头。
		* 请参阅https://eips.ethereum.org/EIPS/eip-3525了解有关slotURI的JSON模式。
		* @return `_slot` 的JSON格式化URI
		*/
    function slotURI(uint256 _slot) external view returns (string memory);
}
interface IERC721Metadata is IERC721 {
    /**
     * @notice 用于描述此合约中NFT集合的名称
     */
    function name() external view returns (string memory);

    /**
     * @notice 用于NFT在此合约中的简写名称
     */
    function symbol() external view returns (string memory);

    /**
     * @notice 给定资产的独特统一资源标识符（URI）。
     * @dev 如果 `_tokenId` 不是有效的 NFT，将抛出错误。URI在RFC 3986中定义。
     *  URI可能指向符合 "ERC721 Metadata JSON Schema"的JSON文件。
     */
    function tokenURI(uint256 _tokenId) external view returns (string memory);
}
```

ERC3525的结构相比于`ERC20`、`ERC721`、`ERC1155`会更复杂更难理解，下图可以让大家快速去理解ERC3525可以表达的数据结构和特征。

![Untitled](ERC-3525%20WTF%2008d213a9f6a04c2884b96bdce9a47c9c/Untitled.png)

## `IERC3525`接口合约[](https://www.wtf.academy/solidity-application/ERC1155/#ierc1155%E6%8E%A5%E5%8F%A3%E5%90%88%E7%BA%A6)

`IERC3525`接口合约抽象了`EIP3525`需要实现的功能，其中包含`3`个事件和`7`个函数以及继承了`IERC721`中的所有功能。

```solidity
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/introspection/IERC165.sol";
import "./IERC721.sol";

/**
 * @title ERC-3525 半同质化代币标准
 * @dev 参见 https://eips.ethereum.org/EIPS/eip-3525
 * 注意：这个接口的 ERC-165 标识符是 0xd5358140。
 */
interface IERC3525 is IERC165, IERC721 {
    /**
     * @dev 当一个代币的value被转移到具有相同 slot 的另一个代币时，必须发出此事件，
     *  包括零值转账（_value == 0），以及在代币被创建（`_fromTokenId` == 0）或销毁（`_toTokenId` == 0）时的转账。
     * @param _fromTokenId 从哪个代币转移value
     * @param _toTokenId 转移到哪个代币
     * @param _value 转移的value
     */
    event TransferValue(uint256 indexed _fromTokenId, uint256 indexed _toTokenId, uint256 _value);

    /**
     * @dev 当一个代币的批准value被设置或更改时，必须发出此事件。
     * @param _tokenId 需要批准的代币
     * @param _operator 需要批准的操作者
     * @param _value `_operator` 被允许管理的最大value
     */
    event ApprovalValue(uint256 indexed _tokenId, address indexed _operator, uint256 _value);

    /**
     * @dev 当一个代币的 slot 被设置或更改时，必须发出此事件。
     * @param _tokenId slot 被设置或更改的代币
     * @param _oldSlot 代币的旧 slot
     * @param _newSlot 代币的新 slot
     */ 
    event SlotChanged(uint256 indexed _tokenId, uint256 indexed _oldSlot, uint256 indexed _newSlot);

    /**
     * @notice 获取代币对value使用的小数位数 - 例如，6，表示用户可以通过将代币value除以1,000,000来表示。
     *  考虑到与第三方钱包的兼容性，这个函数被定义为 `valueDecimals()` 而不是 `decimals()`，以避免与 ERC20 代币冲突。
     * @return value的小数位数
     */
    function valueDecimals() external view returns (uint8);

    /**
     * @notice 获取一个代币的value。
     * @param _tokenId 需要查询余额的value
     * @return `_tokenId` 的value
     */
    function balanceOf(uint256 _tokenId) external view returns (uint256);

    /**
     * @notice 获取一个代币的 slot。
     * @param _tokenId 需要查询的代币的标识符
     * @return 代币的 slot
     */
    function slotOf(uint256 _tokenId) external view returns (uint256);

    /**
     * @notice 允许一个操作者管理一个代币的value，最高到 `_value` 金额。
     * @dev 除非调用者是当前所有者，已授权的操作者，或者是对 `_tokenId` 已批准的地址，否则必须回滚。
     *  必须发出 ApprovalValue 事件。
     * @param _tokenId 需要批准的代币
     * @param _operator 需要批准的操作者
     * @param _value `_operator` 被允许管理的 `_toTokenId` 的最大value
     */
    function approve(
        uint256 _tokenId,
        address _operator,
        uint256 _value
    ) external payable;

    /**
     * @notice 获取一个操作者被允许管理的一个代币的最大value。
     * @param _tokenId 需要查询授权的代币
     * @param _operator 一个操作者的地址
     * @return `_operator` 被允许管理的 `_tokenId` 的当前授权value
     */
    function allowance(uint256 _tokenId, address _operator) external view returns (uint256);

    /**
     * @notice 从一个指定的代币转移value到具有相同 slot 的另一个指定的代币。
     * @dev 除非调用者是当前所有者，已授权的操作者，或者是对整个 `_fromTokenId` 或其部分已授权的操作者，否则必须回滚。
     *  如果 `_fromTokenId` 或 `_toTokenId` 是零代币 id 或不存在，必须回滚。
     *  如果 `_fromTokenId` 和 `_toTokenId` 的 slots 不匹配，必须回滚。
     *  如果 `_value` 超过了 `_fromTokenId` 的余额或其对操作者的授权，必须回滚。
     *  必须发出 `TransferValue` 事件。
     * @param _fromTokenId 需要从哪个代币转移value
     * @param _toTokenId 需要转移value到哪个代币
     * @param _value 转移的value
     */
    function transferFrom(
        uint256 _fromTokenId,
        uint256 _toTokenId,
        uint256 _value
    ) external payable;

    /**
     * @notice 从一个指定的代币转移value到一个地址。调用者应确认 `_to` 能够接收 ERC3525 代币。
     * @dev 这个函数必须为 `_to` 创建一个新的具有相同 slot 的 ERC3525 代币，以接收转移的value。
     *  如果 `_fromTokenId` 是零代币 id 或不存在，必须回滚。
     *  如果 `_to` 是零地址，必须回滚。
     *  如果 `_value` 超过了 `_fromTokenId` 的余额或其对操作者的授权，必须回滚。
     *  必须发出 `Transfer` 和 `TransferValue` 事件。
     * @param _fromTokenId 需要从哪个代币转移value
     * @param _to 需要转移value到哪个地址
     * @param _value 转移的value
     * @return 为 `_to` 创建的新代币的 ID，它接收转移的value
     */
    function transferFrom(
        uint256 _fromTokenId,
        address _to,
        uint256 _value
    ) external payable returns (uint256);
}
```

**`IERC3525`事件：**

`**TransferValue**`事件：单类代币转账事件，在同`slot`下一个`id`内的`value`进行转账时释放。

`**ApprovalValue**`事件：授权事件，在当某个`id`内的`value`进行授权时释放。

`**SlotChanged**`事件：slot更改事件，在某个`id`的`slot`被更改时释放。

**`IERC3525`函数：**

`**valueDecimals()**` ：获取代币使用的小数位数。

`**balanceOf(uint256 _tokenId)**` ：获取某个`id`的`value`余额。

`**slotOf(uint256 _tokenId)**` ：获取某个`id`属于哪一个`slot`。

`**approve(uint256 _tokenId, address _operator, uint256 _value)**` ：将调用者某个`id`内的`value`额度授权给`operator`地址。

`**allowance(uint256 _tokenId, address _operator)**` ：获取`operator`被允许管理的某个`id`内`value`的最大值。

`**transferFrom(uint256 _fromTokenId, uint256 _toTokenId, uint256 _value)**` ：将某一个`id`内的`value`转移至另外一个`id`内。

`**transferFrom(uint256 _fromTokenId, address _to, uint256 _value)**` ：将某一个`id`的`value`转移至`to`地址。

## **`ERC3525`接收合约**

与`ERC721`和`ERC1155`的`Receiver`不同，前两者是使用`safeTransferFrom`时对方合约需实现`Receiver`接口才可以接受代币；而`ERC3525`则是会先探测转账接收方有没有实现`Receiver`接口：

- 如果接收方（合约）实现该`Receiver`接口则调用通知功能，由接收方在通知接口功能中进行响应，即接收方可选择接受与否。
- 如果接收方（合约）没有实现这个功能，则正常进行转账操作。
- 如果接收方不是合约，则正常进行转账操作。

`ERC3525`之所以在`Receiver`接口中采用这种设计，实际上是改良了基于在实际使用中`ERC721`和`ERC1155`的设计缺陷：

**ERC-721：**

包含`SafeTransfer`与普通`Transfer`两套接口，带来了两个问题：

- 发起转账的钱包不知道应该调用哪个接口最合适，只能选一个最简单、通用的，导致实际上`SafeTransfer`被使用的频率十分少，并没有达到协议设计的目标。
- 一旦调用方采用`SafeTransfer`接口，接收方如果不实现`Receiver`接口，则调用必然失败，与旧合约兼容性差。

**ERC-1155：**

- 只保留了`SafeTransfer`接口，虽然消除了如何选择接口的问题，但是兼容问题更严重了，很多旧合约并没有兼容`ERC1155`，并且依赖于新合约兼容`ERC1155`接口。

`ERC3525`由于兼容`ERC721`的`Receiver`接口，所以只要兼容`ERC721`，即便是没有实现`IERC3525Receiver`的合约也可以正常接收`ERC3525`，而实现了`IERC3525Receiver`的合约则可以选择是否接收该代币，这在许多真实应用场景中十分有效（如供应链金融、AML反洗钱等）。

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.1;

/**
 * @title EIP-3525 代币接收者接口
 * @dev 当从任何地址或 EIP-3525 代币接收value时，EIP-3525 合约希望通知此智能合约。
 * 注意：此接口的 EIP-165 标识符是 0x009ce20b。
 */
interface IERC3525Receiver {
    /**
     * @notice 处理接收一个 EIP-3525 代币value。
     * @dev 一个 EIP-3525 智能合约必须检查接收者合约是否实现了这个函数，如果接收者合约实现了这个函数，
     *  EIP-3525 合约必须在value转移后调用这个函数（即 `transferFrom(uint256, uint256,uint256,bytes)`）。
     *  如果接受转移，则必须返回 0x009ce20b（即 `bytes4(keccak256('onERC3525Received(address,uint256,uint256,uint256,bytes)'))`）。
     *  如果拒绝转移，必须回滚或返回除 0x009ce20b 以外的任何值。
     * @param _operator 触发转移的地址
     * @param _fromTokenId 从哪个代币转移value
     * @param _toTokenId 转移value到哪个代币
     * @param _value 转移的value
     * @param _data 没有指定格式的额外数据
     * @return `bytes4(keccak256('onERC3525Received(address,uint256,uint256,uint256,bytes)'))` 除非转移被拒绝。
     */
    function onERC3525Received(address _operator, uint256 _fromTokenId, uint256 _toTokenId, uint256 _value, bytes calldata _data) external returns (bytes4);
}
```

## `ERC3525`可选接口定义合约（Optional）

### **IERC3525SlotEnumerable**

这是一个可选的扩展接口合约，添加了对`slot`进行枚举的支持：

**`slotCount`**: 此函数返回由合约存储的`slot`的总数。没有输入参数，返回值是一个 uint256 类型的数值，代表`slot`的总数。

**`slotByIndex`**: 此函数返回由合约存储的所有`slot`中指定索引处的`slot`。输入参数是一个 uint256 类型的数值 `**_index**`，代表`slot`列表中的索引。返回值是一个 uint256 类型的数值，代表指定索引处的`slot`。

**`tokenSupplyInSlot`**: 此函数返回具有相同`slot`的所有代币的总数。输入参数是一个 uint256 类型的数值 **`_slot`**，代表要查询代币供应量的`slot`。返回值是一个 uint256 类型的数值，代表指定`slot`的所有代币的总数。

**`tokenInSlotByIndex`**: 此函数返回具有相同`slot`的所有代币中指定索引处的代币。输入参数包括一个 uint256 类型的数值 **`_slot`**，代表要查询代币的`slot`，和另一个 uint256 类型的数值 **`_index`**，代表`slot`中代币列表的索引。返回值是一个 uint256 类型的数值，代表具有指定`slot`的所有代币中指定索引处的代币 `id`。

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.1;

import "../IERC3525.sol";
import "./IERC721Enumerable.sol";
/**
 * @title ERC-3525 半可替换代币标准，可选扩展用于slot枚举
 * @dev 任何希望支持枚举slots以及具有相同slot的tokens的合约的接口。
 *  参见 https://eips.ethereum.org/EIPS/eip-3525
 * 注意：此接口的ERC-165标识符为 0x3b741b9e。
 */
interface IERC3525SlotEnumerable is IERC3525, IERC721Enumerable {

    /**
     * @notice 获取合约存储的slots的总量。
     * @return slots的总量
     */
    function slotCount() external view returns (uint256);

    /**
     * @notice 获取合约存储的所有slots中指定索引处的slot。
     * @param _index slot列表中的索引
     * @return 所有slots中`index`处的slot。
     */
    function slotByIndex(uint256 _index) external view returns (uint256);

    /**
     * @notice 获取具有相同slot的tokens的总量。
     * @param _slot 用于查询token供应量的slot
     * @return 具有指定`_slot`的tokens的总量
     */
    function tokenSupplyInSlot(uint256 _slot) external view returns (uint256);

    /**
     * @notice 获取具有相同slot的所有tokens中指定索引处的token。
     * @param _slot 用于查询tokens的slot
     * @param _index slot的token列表中的索引
     * @return 具有`_slot`的所有tokens中`_index`处的token ID
     */
    function tokenInSlotByIndex(uint256 _slot, uint256 _index) external view returns (uint256);
}
```

### IERC3525SlotApprovable

这是一个可选的扩展接口合约，添加了一种可对某个`slot`内的所有代币进行授权的支持：

**`ApprovalForSlot`**: 这是一个事件，当一个操作者被授权或被取消授权管理一个所有者的具有相同slot的所有代币时，会触发此事件。事件参数包括代币的所有者`owner`，被授权的插槽`slot`，被授权或被取消授权的操作者`operator`，以及操作者是否被授权`approved`。

**`setApprovalForSlot`**: 此函数用于授权或取消授权一个操作者管理一个所有者的具有特定`slot`的所有代币。这个函数在执行时会触发 **`ApprovalSlot`** 事件。调用者应该是所有者或已经通过 **`setApprovalForAll`** 被授权的操作者。

**`isApprovedForSlot`**: 此函数用于查询一个操作者是否被授权管理一个所有者的具有特定`slot`的所有代币。如果操作者被授权管理所有者的特定`slot`的所有代币，函数返回 true，否则返回 false。

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.1;

import "../IERC3525.sol";

/**
 * @title ERC-3525 半同质化代币标准，支持slot级别批准的可选扩展
 * @dev 任何希望支持slot级别批准的合约的接口，允许一个操作者管理具有相同slot的代币。
 *  参见 https://eips.ethereum.org/EIPS/eip-3525
 * 注意：这个接口的 ERC-165 标识符是 0xb688be58。
 */
interface IERC3525SlotApprovable is IERC3525 {
    /**
     * @dev 当一个操作者被批准或被否决来管理所有者的具有相同slot的所有代币时，必须发出此事件。
     * @param _owner 代币被批准的地址
     * @param _slot 被批准的slot，所有者的所有这个slot的代币都被批准
     * @param _operator 被批准或被否决的操作者
     * @param _approved 标识操作者是否被批准
     */
    event ApprovalForSlot(address indexed _owner, uint256 indexed _slot, address indexed _operator, bool _approved);

    /**
     * @notice 批准或否决一个操作者管理所有者的指定slot的所有代币。
     * @dev 调用者应当是 `_owner` 或已通过 `setApprovalForAll` 授权的操作者。
     *  必须发出 ApprovalSlot 事件。
     * @param _owner 拥有 ERC3525 代币的地址
     * @param _slot 被查询批准的代币的slot
     * @param _operator 被查询批准的地址
     * @param _approved 标识操作者是否被批准
     */
    function setApprovalForSlot(
        address _owner,
        uint256 _slot,
        address _operator,
        bool _approved
    ) external payable;

    /**
     * @notice 查询操作者是否被授权管理所有者的指定slot的所有代币。
     * @param _owner 拥有 ERC3525 代币的地址
     * @param _slot 被查询批准的代币的slot
     * @param _operator 被查询批准的地址
     * @return 如果操作者被授权管理所有者的 `_slot` 的所有代币，则返回真，否则返回假。
     */
    function isApprovedForSlot(
        address _owner,
        uint256 _slot,
        address _operator
    ) external view returns (bool);
}
```

`ERC3525`相比于`ERC1155`在实际应用场景中有更灵活的授权机制，在协议设计上`ERC1155`只有`setApprovalForAll`，在实际应用中可能会产生资产安全风险。而`ERC3525`则有四层授权机制：

`ERC721 setApproveForAll`：对整个SFT合约进行授权。

`ERC3525 setApprovalForSlot`：对某个`Slot`进行授权。

`ERC-721 approve(tokenId)`：对某个`id`进行授权。

`ERC-3525 approve(tokenId, value)`：对某个id的特定`value`进行授权。

## `ERC3525`主合约[](https://www.wtf.academy/solidity-application/ERC1155/#erc1155%E4%B8%BB%E5%90%88%E7%BA%A6)

`ERC3525`主合约实现了`IERC3525`接口合约规定的函数。

### `ERC3525`结构体

`ERC3525`主合约包含2个结构体：

**`TokenData`**：这个结构体包含了关于代币的各种信息，包括代币的`id`、属于哪个`slot`、余额`balance`、所有者`owner`、被批准的地址`approved`，以及地址被批准的值`valueApprovals`。

**`AddressData`**：这个结构体包含了关于地址的信息，包括该地址所拥有的代币列表`ownedTokens`、每个代币在列表中的索引位置`ownedTokensIndex`，以及对该地址是否被批准`approvals`。

### `ERC3525`变量[](https://www.wtf.academy/solidity-application/ERC1155/#erc1155%E5%8F%98%E9%87%8F)

`ERC3525`主合约包含9个状态变量：

**`_name`**: 一个**`string`**类型的私有变量，代表了此合约的名称。

**`_symbol`**: 一个**`string`**类型的私有变量，代表了此合约的符号。

**`_decimals`**: 一个**`uint8`**类型的私有变量，代表了此合约的小数位数。

**`_tokenIdGenerator`**: **`Counters.Counter`**类型的私有变量，用于生成唯一的代币ID。

**`_approvedValues`**: 这是一个嵌套的**`mapping`**类型私有变量，用于存储每个代币ID对应的被批准的值（通过另一个映射将批准者地址映射到值）。

**`_allTokens`**: 这是一个**`TokenData`**结构体数组，存储了所有的代币。

**`_allTokensIndex`**: 这是一个**`mapping`**类型的私有变量，用于存储每个代币ID对应的索引。

**`_addressData`**: 这是一个**`mapping`**类型的私有变量，将地址映射到**`AddressData`**结构，存储了与每个地址相关的数据。

**`metadataDescriptor`**: 这是一个公开的**`IERC3525MetadataDescriptor`**类型变量，它是一个接口，用于描述元数据。

### `ERC3525`函数[](https://www.wtf.academy/solidity-application/ERC1155/#erc1155%E5%87%BD%E6%95%B0)

`ERC3525`主合约包含57个函数：

构造函数：初始化状态变量`name`和`symbol`和`decimals`。

**`supportsInterface(bytes4 interfaceId)`**: 实现 ERC165 标准，声明合约支持的接口，供其他合约检查。

**`name()`**: 返回合约的名称。

**`symbol()`**: 返回合约的符号。

**`valueDecimals()`**: 返回代币的小数位数。

**`balanceOf(uint256 tokenId_)`**: 查询指定代币`id`的余额。

**`ownerOf(uint256 tokenId_)`**: 查询指定代币`id`的所有者。

**`slotOf(uint256 tokenId_)`**: 查询指定代币`id`的`slot`。

**`_baseURI()`**: 用于构造代币的基础 URI。

**`contractURI()`**: 获取合约的 URI。

**`slotURI(uint256 slot_)`**: 获取存储`slot`的 URI。

**`tokenURI(uint256 tokenId_)`**: 获取特定代币`id`的 URI。

**`approve(uint256 tokenId_, address to_, uint256 value_)`**: 批准另一个地址转移指定代币`id`的特定`value`。

**`allowance(uint256 tokenId_, address operator_)`**: 查询指定地址对于另一个地址的代币`id`批准额度。

**`transferFrom(uint256 fromTokenId_, address to_, uint256 value_)`**: 从一个代币`id`转移特定值到一个地址。

**`transferFrom(uint256 fromTokenId_, uint256 toTokenId_, uint256 value_)`**: 从一个代币`id`转移特定值到另一个代币`id`。

**`balanceOf(address owner_)`**: 查询指定地址的余额。

**`transferFrom(address from_, address to_, uint256 tokenId_)`**: 从一个地址转移一个代币`id`到另一个地址。

**`safeTransferFrom(address from_, address to_, uint256 tokenId_, bytes memory data_)`**: 安全地从一个地址转移一个代币到另一个地址，同时传递额外的数据。

**`safeTransferFrom(address from_, address to_, uint256 tokenId_)`**: 安全地从一个地址转移一个代币`id`到另一个地址。

**`approve(address to_, uint256 tokenId_)`**: 批准另一个地址转移指定的代币`id`。

**`getApproved(uint256 tokenId_)`**: 获取特定代币`id`的已批准地址。

**`setApprovalForAll(address operator_, bool approved_)`**: 允许或禁止另一个地址转移用户的所有代币。

**`isApprovedForAll(address owner_, address operator_)`**: 检查一个地址是否被另一个地址全权批准。

**`totalSupply()`**: 返回合约中的代币总供应量。

**`tokenByIndex(uint256 index_)`**: 根据索引返回一个特定的代币。

**`tokenOfOwnerByIndex(address owner_, uint256 index_)`**: 根据所有者地址和索引返回一个特定的代币。

**`_setApprovalForAll(address owner_, address operator_, bool approved_)`**: 内部函数，用于设置对所有代币的批准。

**`_isApprovedOrOwner(address operator_, uint256 tokenId_)`**: 内部函数，检查是否已批准或是所有者。

**`_spendAllowance(address operator_, uint256 tokenId_, uint256 value_)`**: 内部函数，用于花费代币`id`的批准额度。

**`_exists(uint256 tokenId_)`**: 内部函数，检查特定的代币`id`是否存在。

**`_requireMinted(uint256 tokenId_)`**: 内部函数，要求特定的代币`id`必须已经铸造。

**`_mint(address to_, uint256 slot_, uint256 value_)`**: 内部函数，铸造一个新的代币并指定其`slot`和`value`，转移给一个地址。

**`_mint(address to_, uint256 tokenId_, uint256 slot_, uint256 value_)`**: 内部函数，铸造一个新的代币并指定其 `id`、`slot`和`value`，转移给一个地址。

**`_mintValue(uint256 tokenId_, uint256 value_)`**: 内部函数，用于增加代币`id`的`value`。

**`__mintValue(uint256 tokenId_, uint256 value_)`**: 内部函数，用于在铸造时增加代币`id`的`value`。

**`__mintToken(address to_, uint256 tokenId_, uint256 slot_)`**: 内部函数，铸造一个新的代币并指定其 `id` 和`slot`，转移给一个地址。

**`_burn(uint256 tokenId_)`**: 内部函数，用于销毁代币`id`。

**`_burnValue(uint256 tokenId_, uint256 burnValue_)`**: 内部函数，用于减少代币`id`的`value`。

**`_addTokenToOwnerEnumeration(address to_, uint256 tokenId_)`**: 内部函数，将代币`id`添加到所有者的枚举列表中。

**`_removeTokenFromOwnerEnumeration(address from_, uint256 tokenId_)`**: 内部函数，从所有者的枚举列表中移除代币`id`。

**`_addTokenToAllTokensEnumeration(TokenData memory tokenData_)`**: 内部函数，将代币`id`添加到所有代币的枚举列表中。

**`_removeTokenFromAllTokensEnumeration(uint256 tokenId_)`**: 内部函数，从所有代币的枚举列表中移除代币`id`。

**`_approve(address to_, uint256 tokenId_)`**: 内部函数，用于批准代币`id`的转移。

**`_approveValue(uint256 tokenId_, address to_, uint256 value_)`**: 内部函数，用于批准代币`id`内`value`的转移。

**`_clearApprovedValues(uint256 tokenId_)`**: 内部函数，用于清除已批准的代币`id`的`value`。

**`_existApproveValue(address to_, uint256 tokenId_)`**: 内部函数，检查是否存在已批准的代币`id`的`value`。

**`_transferValue(uint256 fromTokenId_, uint256 toTokenId_, uint256 value_)`**: 内部函数，用于从`id`到`id`的转移代币。

**`_transferTokenId(address from_, address to_, uint256 tokenId_)`**: 内部函数，用于转移代币`id`。

**`_safeTransferTokenId(address from_, address to_, uint256 tokenId_, bytes memory data_)`**: 内部函数，安全地从一个地址转移一个代币`id`到另一个地址，同时传递额外的数据。

**`_checkOnERC3525Received(uint256 fromTokenId_, uint256 toTokenId_, uint256 value_, bytes memory data_)`**: 内部函数，检查 **`onERC3525Received`** 接口的实现。

**`_checkOnERC721Received(address from_, address to_, uint256 tokenId_, bytes memory data_)`**: 内部函数，检查 **`onERC721Received`** 接口的实现。

**`_beforeValueTransfer(address from_, address to_, uint256 fromTokenId_, uint256 toTokenId_, uint256 slot_, uint256 value_)`**: 内部函数，用于在转移`value`之前执行操作。

**`_afterValueTransfer(address from_, address to_, uint256 fromTokenId_, uint256 toTokenId_, uint256 slot_, uint256 value_)`**: 内部函数，用于在转移`value`之后执行操作。

**`_setMetadataDescriptor(address metadataDescriptor_)`**: 内部函数，用于设置元数据描述符。

**`_createOriginalTokenId()`**: 内部函数，用于创建原始代币 `id`。

**`_createDerivedTokenId(uint256 fromTokenId_)`**: 内部函数，用于创建派生代币 `id`。

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

// 导入其他的智能合约或库
import "@openzeppelin/contracts/utils/Context.sol";
import "@openzeppelin/contracts/utils/introspection/IERC165.sol";
import "@openzeppelin/contracts/utils/Strings.sol";
import "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "./IERC721.sol";
import "./IERC3525.sol";
import "./IERC721Receiver.sol";
import "./IERC3525Receiver.sol";
import "./extensions/IERC721Enumerable.sol";
import "./extensions/IERC721Metadata.sol";
import "./extensions/IERC3525Metadata.sol";
import "./periphery/interface/IERC3525MetadataDescriptor.sol";

// 定义ERC3525半同质化代币合约，继承自Context、IERC3525Metadata和IERC721Enumerable接口
contract ERC3525 is Context, IERC3525Metadata, IERC721Enumerable {
    // 使用OpenZeppelin库提供的一些实用工具
    using Strings for address;
    using Strings for uint256;
    using Address for address;
    using Counters for Counters.Counter;

    // 定义了一个元数据描述符被设定的事件
    event SetMetadataDescriptor(address indexed metadataDescriptor);

    // 定义代币数据的结构体
    struct TokenData {
        uint256 id;
        uint256 slot;
        uint256 balance;
        address owner;
        address approved;
        address[] valueApprovals;
    }

    // 定义地址数据的结构体
    struct AddressData {
        uint256[] ownedTokens;
        mapping(uint256 => uint256) ownedTokensIndex;
        mapping(address => bool) approvals;
    }

    // 定义一些私有变量
    string private _name;
    string private _symbol;
    uint8 private _decimals;
    Counters.Counter private _tokenIdGenerator;

    // id => (approval => allowance)
    // @dev _approvedValues不能在TokenData中定义，因为含有映射的结构不能被构造。
    mapping(uint256 => mapping(address => uint256)) private _approvedValues;

    TokenData[] private _allTokens;

    // key: id
    mapping(uint256 => uint256) private _allTokensIndex;

    mapping(address => AddressData) private _addressData;

    // 元数据描述符
    IERC3525MetadataDescriptor public metadataDescriptor;

    // 合约的构造函数，传入名称、符号和小数位数
    constructor(string memory name_, string memory symbol_, uint8 decimals_) {
         _name = name_;
        _symbol = symbol_;
        _decimals = decimals_;
    }

    // 支持的接口判断函数
    function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
        return
            interfaceId == type(IERC165).interfaceId ||
            interfaceId == type(IERC3525).interfaceId ||
            interfaceId == type(IERC721).interfaceId ||
            interfaceId == type(IERC3525Metadata).interfaceId ||
            interfaceId == type(IERC721Enumerable).interfaceId || 
            interfaceId == type(IERC721Metadata).interfaceId;
    }

    /**
     * @dev 返回代币集合的名称。
     */
    function name() public view virtual override returns (string memory) {
        return _name;
    }

    /**
     * @dev 返回代币集合的符号。
     */
    function symbol() public view virtual override returns (string memory) {
        return _symbol;
    }

    /**
     * @dev 返回代币用于value的小数位数。
     */
    function valueDecimals() public view virtual override returns (uint8) {
        return _decimals;
    }

    // 获取代币的余额
    function balanceOf(uint256 tokenId_) public view virtual override returns (uint256) {
        _requireMinted(tokenId_);
        return _allTokens[_allTokensIndex[tokenId_]].balance;
    }

    // 获取代币的所有者
    function ownerOf(uint256 tokenId_) public view virtual override returns (address owner_) {
        _requireMinted(tokenId_);
        owner_ = _allTokens[_allTokensIndex[tokenId_]].owner;
        require(owner_ != address(0), "ERC3525: invalid token ID");
    }

    // 获取代币的slot值
    function slotOf(uint256 tokenId_) public view virtual override returns (uint256) {
        _requireMinted(tokenId_);
        return _allTokens[_allTokensIndex[tokenId_]].slot;
    }

    // 获取基础URI
    function _baseURI() internal view virtual returns (string memory) {
        return "";
    }

    // 获取合约URI
    function contractURI() public view virtual override returns (string memory) {
        string memory baseURI = _baseURI();
        return 
            address(metadataDescriptor) != address(0) ? 
                metadataDescriptor.constructContractURI() :
                bytes(baseURI).length > 0 ? 
                    string(abi.encodePacked(baseURI, "contract/", Strings.toHexString(address(this)))) : 
                    "";
    }

    // 获取slot URI
    function slotURI(uint256 slot_) public view virtual override returns (string memory) {
        string memory baseURI = _baseURI();
        return 
            address(metadataDescriptor) != address(0) ? 
                metadataDescriptor.constructSlotURI(slot_) : 
                bytes(baseURI).length > 0 ? 
                    string(abi.encodePacked(baseURI, "slot/", slot_.toString())) : 
                    "";
    }

    /**
     * @dev 返回`tokenId`代币的统一资源标识符 (URI)。
     */
    function tokenURI(uint256 tokenId_) public view virtual override returns (string memory) {
        _requireMinted(tokenId_);
        string memory baseURI = _baseURI();
        return 
            address(metadataDescriptor) != address(0) ? 
                metadataDescriptor.constructTokenURI(tokenId_) : 
                bytes(baseURI).length > 0 ? 
                    string(abi.encodePacked(baseURI, tokenId_.toString())) : 
                    "";
    }

    // 批准操作
    function approve(uint256 tokenId_, address to_, uint256 value_) public payable virtual override {
        address owner = ERC3525.ownerOf(tokenId_);
        require(to_ != owner, "ERC3525: approval to current owner");

        require(_isApprovedOrOwner(_msgSender(), tokenId_), "ERC3525: approve caller is not owner nor approved");

        _approveValue(tokenId_, to_, value_);
    }

    // 获取批准值
    function allowance(uint256 tokenId_, address operator_) public view virtual override returns (uint256) {
        _requireMinted(tokenId_);
        return _approvedValues[tokenId_][operator_];
    }

    // 转账操作
    function transferFrom(
        uint256 fromTokenId_,
        address to_,
        uint256 value_
    ) public payable virtual override returns (uint256 newTokenId) {
        _spendAllowance(_msgSender(), fromTokenId_, value_);

        newTokenId = _createDerivedTokenId(fromTokenId_);
        _mint(to_, newTokenId, ERC3525.slotOf(fromTokenId_), 0);
        _transferValue(fromTokenId_, newTokenId, value_);
    }

    // 转账操作
    function transferFrom(
        uint256 fromTokenId_,
        uint256 toTokenId_,
        uint256 value_
    ) public payable virtual override {
        _spendAllowance(_msgSender(), fromTokenId_, value_);
        _transferValue(fromTokenId_, toTokenId_, value_);
    }

    // 获取某个地址的余额
    function balanceOf(address owner_) public view virtual override returns (uint256 balance) {
        require(owner_ != address(0), "ERC3525: balance query for the zero address");
        return _addressData[owner_].ownedTokens.length;
    }

    // 转账操作
    function transferFrom(
        address from_,
        address to_,
        uint256 tokenId_
    ) public payable virtual override {
        require(_isApprovedOrOwner(_msgSender(), tokenId_), "ERC3525: transfer caller is not owner nor approved");
        _transferTokenId(from_, to_, tokenId_);
    }

    // 安全转账操作
    function safeTransferFrom(
        address from_,
        address to_,
        uint256 tokenId_,
        bytes memory data_
    ) public payable virtual override {
        require(_isApprovedOrOwner(_msgSender(), tokenId_), "ERC3525: transfer caller is not owner nor approved");
        _safeTransferTokenId(from_, to_, tokenId_, data_);
    }

    // 安全转账操作
    function safeTransferFrom(
        address from_,
        address to_,
        uint256 tokenId_
    ) public payable virtual override {
        safeTransferFrom(from_, to_, tokenId_, "");
    }

    // 批准操作
    function approve(address to_, uint256 tokenId_) public payable virtual override {
        address owner = ERC3525.ownerOf(tokenId_);
        require(to_ != owner, "ERC3525: approval to current owner");

        require(
            _msgSender() == owner || ERC3525.isApprovedForAll(owner, _msgSender()),
            "ERC3525: approve caller is not owner nor approved for all"
        );

        _approve(to_, tokenId_);
    }

    // 获取被批准的地址
    function getApproved(uint256 tokenId_) public view virtual override returns (address) {
        _requireMinted(tokenId_);
        return _allTokens[_allTokensIndex[tokenId_]].approved;
    }

    // 为所有的代币设置批准
    function setApprovalForAll(address operator_, bool approved_) public virtual override {
        _setApprovalForAll(_msgSender(), operator_, approved_);
    }

    // 检查是否为所有的代币都被批准
    function isApprovedForAll(address owner_, address operator_) public view virtual override returns (bool) {
        return _addressData[owner_].approvals[operator_];
    }

    // 获取总供应量
    function totalSupply() public view virtual override returns (uint256) {
        return _allTokens.length;
    }

    // 根据索引获取代币
    function tokenByIndex(uint256 index_) public view virtual override returns (uint256) {
        require(index_ < ERC3525.totalSupply(), "ERC3525: global index out of bounds");
        return _allTokens[index_].id;
    }

    // 根据所有者和索引获取代币
    function tokenOfOwnerByIndex(address owner_, uint256 index_) public view virtual override returns (uint256) {
        require(index_ < ERC3525.balanceOf(owner_), "ERC3525: owner index out of bounds");
        return _addressData[owner_].ownedTokens[index_];
    }

    // 设置所有者批准
    function _setApprovalForAll(
        address owner_,
        address operator_,
        bool approved_
    ) internal virtual {
        require(owner_ != operator_, "ERC3525: approve to caller");

        _addressData[owner_].approvals[operator_] = approved_;

        emit ApprovalForAll(owner_, operator_, approved_);
    }
}
```

```solidity
		// 检查给定的地址是代币的所有者还是被批准的操作者
    function _isApprovedOrOwner(address operator_, uint256 tokenId_) internal view virtual returns (bool) {
        address owner = ERC3525.ownerOf(tokenId_);
        return (
            operator_ == owner ||
            ERC3525.isApprovedForAll(owner, operator_) ||
            ERC3525.getApproved(tokenId_) == operator_
        );
    }

    // 用来检查操作者是否有足够的代币来进行操作，并更新批准的数量
    function _spendAllowance(address operator_, uint256 tokenId_, uint256 value_) internal virtual {
        uint256 currentAllowance = ERC3525.allowance(tokenId_, operator_);
        if (!_isApprovedOrOwner(operator_, tokenId_) && currentAllowance != type(uint256).max) {
            require(currentAllowance >= value_, "ERC3525: insufficient allowance");
            _approveValue(tokenId_, operator_, currentAllowance - value_);
        }
    }

    // 检查代币id是否存在
    function _exists(uint256 tokenId_) internal view virtual returns (bool) {
        return _allTokens.length != 0 && _allTokens[_allTokensIndex[tokenId_]].id == tokenId_;
    }

    // 检查代币id是否已经被铸造
    function _requireMinted(uint256 tokenId_) internal view virtual {
        require(_exists(tokenId_), "ERC3525: invalid token ID");
    }

    // 铸造新的代币
    function _mint(address to_, uint256 slot_, uint256 value_) internal virtual returns (uint256 tokenId) {
        tokenId = _createOriginalTokenId();
        _mint(to_, tokenId, slot_, value_);  
    }

    // 铸造新的代币
    function _mint(address to_, uint256 tokenId_, uint256 slot_, uint256 value_) internal virtual {
        require(to_ != address(0), "ERC3525: mint to the zero address");
        require(tokenId_ != 0, "ERC3525: cannot mint zero tokenId");
        require(!_exists(tokenId_), "ERC3525: token already minted");

        _beforeValueTransfer(address(0), to_, 0, tokenId_, slot_, value_);
        __mintToken(to_, tokenId_, slot_);
        __mintValue(tokenId_, value_);
        _afterValueTransfer(address(0), to_, 0, tokenId_, slot_, value_);
    }

    // 铸造代币的值
    function _mintValue(uint256 tokenId_, uint256 value_) internal virtual {
        address owner = ERC3525.ownerOf(tokenId_);
        uint256 slot = ERC3525.slotOf(tokenId_);
        _beforeValueTransfer(address(0), owner, 0, tokenId_, slot, value_);
        __mintValue(tokenId_, value_);
        _afterValueTransfer(address(0), owner, 0, tokenId_, slot, value_);
    }

    // 铸造代币的值
    function __mintValue(uint256 tokenId_, uint256 value_) private {
        _allTokens[_allTokensIndex[tokenId_]].balance += value_;
        emit TransferValue(0, tokenId_, value_);
    }

    // 铸造代币
    function __mintToken(address to_, uint256 tokenId_, uint256 slot_) private {
        TokenData memory tokenData = TokenData({
            id: tokenId_,
            slot: slot_,
            balance: 0,
            owner: to_,
            approved: address(0),
            valueApprovals: new address[](0)
        });

        _addTokenToAllTokensEnumeration(tokenData);
        _addTokenToOwnerEnumeration(to_, tokenId_);

        emit Transfer(address(0), to_, tokenId_);
        emit SlotChanged(tokenId_, 0, slot_);
    }

    // 销毁代币
    function _burn(uint256 tokenId_) internal virtual {
        _requireMinted(tokenId_);

        TokenData storage tokenData = _allTokens[_allTokensIndex[tokenId_]];
        address owner = tokenData.owner;
        uint256 slot = tokenData.slot;
        uint256 value = tokenData.balance;

        _beforeValueTransfer(owner, address(0), tokenId_, 0, slot, value);

        _clearApprovedValues(tokenId_);
        _removeTokenFromOwnerEnumeration(owner, tokenId_);
        _removeTokenFromAllTokensEnumeration(tokenId_);

        emit TransferValue(tokenId_, 0, value);
        emit SlotChanged(tokenId_, slot, 0);
        emit Transfer(owner, address(0), tokenId_);

        _afterValueTransfer(owner, address(0), tokenId_, 0, slot, value);
    }

    // 销毁代币的值
    function _burnValue(uint256 tokenId_, uint256 burnValue_) internal virtual {
        _requireMinted(tokenId_);

        TokenData storage tokenData = _allTokens[_allTokensIndex[tokenId_]];
        address owner = tokenData.owner;
        uint256 slot = tokenData.slot;
        uint256 value = tokenData.balance;

        require(value >= burnValue_, "ERC3525: burn value exceeds balance");

        _beforeValueTransfer(owner, address(0), tokenId_, 0, slot, burnValue_);
        
        tokenData.balance -= burnValue_;
        emit TransferValue(tokenId_, 0, burnValue_);
        
        _afterValueTransfer(owner, address(0), tokenId_, 0, slot, burnValue_);
    }

    // 将代币添加到所有者的枚举中
    function _addTokenToOwnerEnumeration(address to_, uint256 tokenId_) private {
        _allTokens[_allTokensIndex[tokenId_]].owner = to_;

        _addressData[to_].ownedTokensIndex[tokenId_] = _addressData[to_].ownedTokens.length;
        _addressData[to_].ownedTokens.push(tokenId_);
    }

    // 从所有者的枚举中移除代币
    function _removeTokenFromOwnerEnumeration(address from_, uint256 tokenId_) private {
        _allTokens[_allTokensIndex[tokenId_]].owner = address(0);

        AddressData storage ownerData = _addressData[from_];
        uint256 lastTokenIndex = ownerData.ownedTokens.length - 1;
        uint256 lastTokenId = ownerData.ownedTokens[lastTokenIndex];
        uint256 tokenIndex = ownerData.ownedTokensIndex[tokenId_];

        ownerData.ownedTokens[tokenIndex] = lastTokenId;
        ownerData.ownedTokensIndex[lastTokenId] = tokenIndex;

        delete ownerData.ownedTokensIndex[tokenId_];
        ownerData.ownedTokens.pop();
    }

    // 将代币添加到所有代币的枚举中
    function _addTokenToAllTokensEnumeration(TokenData memory tokenData_) private {
        _allTokensIndex[tokenData_.id] = _allTokens.length;
        _allTokens.push(tokenData_);
    }

    // 从所有代币的枚举中移除代币
    function _removeTokenFromAllTokensEnumeration(uint256 tokenId_) private {
        // To prevent a gap in the tokens array, we store the last token in the index of the token to delete, and
        // then delete the last slot (swap and pop).

        uint256 lastTokenIndex = _allTokens.length - 1;
        uint256 tokenIndex = _allTokensIndex[tokenId_];

        // When the token to delete is the last token, the swap operation is unnecessary. However, since this occurs so
        // rarely (when the last minted token is burnt) that we still do the swap here to avoid the gas cost of adding
        // an 'if' statement (like in _removeTokenFromOwnerEnumeration)
        TokenData memory lastTokenData = _allTokens[lastTokenIndex];

        _allTokens[tokenIndex] = lastTokenData; // Move the last token to the slot of the to-delete token
        _allTokensIndex[lastTokenData.id] = tokenIndex; // Update the moved token's index

        // This also deletes the contents at the last position of the array
        delete _allTokensIndex[tokenId_];
        _allTokens.pop();
    }

    // 批准操作
    function _approve(address to_, uint256 tokenId_) internal virtual {
        _allTokens[_allTokensIndex[tokenId_]].approved = to_;
        emit Approval(ERC3525.ownerOf(tokenId_), to_, tokenId_);
    }

    // 批准代币的值
    function _approveValue(
        uint256 tokenId_,
        address to_,
        uint256 value_
    ) internal virtual {
        require(to_ != address(0), "ERC3525: approve value to the zero address");
        if (!_existApproveValue(to_, tokenId_)) {
            _allTokens[_allTokensIndex[tokenId_]].valueApprovals.push(to_);
        }
        _approvedValues[tokenId_][to_] = value_;

        emit ApprovalValue(tokenId_, to_, value_);
    }

    // 清除所有已批准的值
    function _clearApprovedValues(uint256 tokenId_) internal virtual {
        TokenData storage tokenData = _allTokens[_allTokensIndex[tokenId_]];
        uint256 length = tokenData.valueApprovals.length;
        for (uint256 i = 0; i < length; i++) {
            address approval = tokenData.valueApprovals[i];
            delete _approvedValues[tokenId_][approval];
        }
        delete tokenData.valueApprovals;
    }

    // 检查批准的值是否存在
    function _existApproveValue(address to_, uint256 tokenId_) internal view virtual returns (bool) {
        uint256 length = _allTokens[_allTokensIndex[tokenId_]].valueApprovals.length;
        for (uint256 i = 0; i < length; i++) {
            if (_allTokens[_allTokensIndex[tokenId_]].valueApprovals[i] == to_) {
                return true;
            }
        }
        return false;
    }

    // 转移代币的值
    function _transferValue(
        uint256 fromTokenId_,
        uint256 toTokenId_,
        uint256 value_
    ) internal virtual {
        require(_exists(fromTokenId_), "ERC3525: transfer from invalid token ID");
        require(_exists(toTokenId_), "ERC3525: transfer to invalid token ID");

        TokenData storage fromTokenData = _allTokens[_allTokensIndex[fromTokenId_]];
        TokenData storage toTokenData = _allTokens[_allTokensIndex[toTokenId_]];

        require(fromTokenData.balance >= value_, "ERC3525: insufficient balance for transfer");
        require(fromTokenData.slot == toTokenData.slot, "ERC3525: transfer to token with different slot");

        _beforeValueTransfer(
            fromTokenData.owner,
            toTokenData.owner,
            fromTokenId_,
            toTokenId_,
            fromTokenData.slot,
            value_
        );

        fromTokenData.balance -= value_;
        toTokenData.balance += value_;

        emit TransferValue(fromTokenId_, toTokenId_, value_);

        _afterValueTransfer(
            fromTokenData.owner,
            toTokenData.owner,
            fromTokenId_,
            toTokenId_,
            fromTokenData.slot,
            value_
        );

        require(
            _checkOnERC3525Received(fromTokenId_, toTokenId_, value_, ""),
            "ERC3525: transfer rejected by ERC3525Receiver"
        );
    }
```

```solidity
	// 将代币ID从一个地址转移到另一个地址
    function _transferTokenId(
        address from_,
        address to_,
        uint256 tokenId_
    ) internal virtual {
        require(ERC3525.ownerOf(tokenId_) == from_, "ERC3525: transfer from invalid owner");
        require(to_ != address(0), "ERC3525: transfer to the zero address");

        uint256 slot = ERC3525.slotOf(tokenId_);
        uint256 value = ERC3525.balanceOf(tokenId_);

        _beforeValueTransfer(from_, to_, tokenId_, tokenId_, slot, value);

        _approve(address(0), tokenId_);
        _clearApprovedValues(tokenId_);

        _removeTokenFromOwnerEnumeration(from_, tokenId_);
        _addTokenToOwnerEnumeration(to_, tokenId_);

        emit Transfer(from_, to_, tokenId_);

        _afterValueTransfer(from_, to_, tokenId_, tokenId_, slot, value);
    }

    // 安全地将代币ID从一个地址转移到另一个地址，附带数据
    function _safeTransferTokenId(
        address from_,
        address to_,
        uint256 tokenId_,
        bytes memory data_
    ) internal virtual {
        _transferTokenId(from_, to_, tokenId_);
        require(
            _checkOnERC721Received(from_, to_, tokenId_, data_),
            "ERC3525: transfer to non ERC721Receiver"
        );
    }

    // 检查ERC3525接收器是否已经接收
    function _checkOnERC3525Received( 
        uint256 fromTokenId_, 
        uint256 toTokenId_, 
        uint256 value_, 
        bytes memory data_
    ) internal virtual returns (bool) {
        address to = ERC3525.ownerOf(toTokenId_);
        if (to.isContract()) {
            try IERC165(to).supportsInterface(type(IERC3525Receiver).interfaceId) returns (bool retval) {
                if (retval) {
                    bytes4 receivedVal = IERC3525Receiver(to).onERC3525Received(_msgSender(), fromTokenId_, toTokenId_, value_, data_);
                    return receivedVal == IERC3525Receiver.onERC3525Received.selector;
                } else {
                    return true;
                }
            } catch (bytes memory /** reason */) {
                return true;
            }
        } else {
            return true;
        }
    }

    // 内部函数，用于在目标地址上调用{IERC721Receiver-onERC721Received}。
    // 如果目标地址不是合约，则不执行调用。
    function _checkOnERC721Received(
        address from_,
        address to_,
        uint256 tokenId_,
        bytes memory data_
    ) private returns (bool) {
        if (to_.isContract()) {
            try 
                IERC721Receiver(to_).onERC721Received(_msgSender(), from_, tokenId_, data_) returns (bytes4 retval) {
                return retval == IERC721Receiver.onERC721Received.selector;
            } catch (bytes memory reason) {
                if (reason.length == 0) {
                    revert("ERC721: transfer to non ERC721Receiver implementer");
                } else {
                    assembly {
                        revert(add(32, reason), mload(reason))
                    }
                }
            }
        } else {
            return true;
        }
    }

    // 在值转移之前调用，可在派生合约中覆盖
    function _beforeValueTransfer(
        address from_,
        address to_,
        uint256 fromTokenId_,
        uint256 toTokenId_,
        uint256 slot_,
        uint256 value_
    ) internal virtual {}

    // 在值转移之后调用，可在派生合约中覆盖
    function _afterValueTransfer(
        address from_,
        address to_,
        uint256 fromTokenId_,
        uint256 toTokenId_,
        uint256 slot_,
        uint256 value_
    ) internal virtual {}

    // 设置元数据描述符
    function _setMetadataDescriptor(address metadataDescriptor_) internal virtual {
        metadataDescriptor = IERC3525MetadataDescriptor(metadataDescriptor_);
        emit SetMetadataDescriptor(metadataDescriptor_);
    }

    // 创建原始的代币ID
    function _createOriginalTokenId() internal virtual returns (uint256) {
         _tokenIdGenerator.increment();
        return _tokenIdGenerator.current();
    }

    // 创建派生的代币ID
    function _createDerivedTokenId(uint256 fromTokenId_) internal virtual returns (uint256) {
        fromTokenId_;
        return _createOriginalTokenId();
    }
}
```