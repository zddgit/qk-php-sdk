### 说明
需要有一个夸克区块链的全节点，并且同步到最新区块链。

### 安装
```
composer require chenjia404/qk-php-sdk
```

### QKI转账代码
```
<?php
use quarkblockchain\QkNodeRPC;
use quarkblockchain\QkToken;
    /**
     * QKI转账
     * @param $num //转账数量，1个就是1，100个就是100
     * @param $address //接收地址
     * @return bool
     */
    public function transfer($num, $address)
    {
        //xxxx为服务器运行的夸克区块链节点端口号，如果不是调用的当前服务器的节点，请填写所调用的服务器IP地址
        $url = "http://127.0.0.1:xxxx";
        $url_arr = parse_url($url);
        //实例化节点对象
        $qk_node = new QkNodeRPC($url_arr['host'], $url_arr['port']);
        //托管地址（发送方）
        $payer = "xxxxxxxxxxxxxxxxxxxxxxxxxxx";
        //转账
        $transaction = $qk_node->personal()->transaction($payer, $address)
            ->amount("0")
            ->data("");
        //XXXXXXXXX为发送方钱包密码
        $txId = $transaction->send("XXXXXXXXXXXX");

        if ($txId && strlen($txId) == 66) {
            //返回交易hash
            return $txId;
        } else {
            return false;
        }
    }
?>
```


### 通证转账
```
<?php
use quarkblockchain\QkNodeRPC;
use quarkblockchain\QkToken;
    /**
     * 通证转账
     * @param $num //转账数量，1个就是1，100个就是100
     * @param $address //接收地址
     * @param $contract_address  //token合约地址，适用于所有erc20的token
     */
    public function transfer($num, $address, $contract_address)
    {
        
        //xxxx为服务器运行的夸克区块链节点端口号，如果不是调用的当前服务器的节点，请填写所调用的服务器IP地址
        $url = "http://127.0.0.1:xxxx";
        $url_arr = parse_url($url);
        //实例化节点对象
        $qk_node = new QkNodeRPC($url_arr['host'], $url_arr['port']);
        $QkToken = new QkToken($qk_node);
        $token = $QkToken->token($contract_address);
        //托管地址（发送方）
        $payer = "xxxxxxxxxxxxxxxxxxxxxxxxxxx";
        //转账
        $data = $token->encodedTransferData($address, $num);
        $transaction = $qk_node->personal()->transaction($payer, $contract_address)
            ->amount("0")
            ->data($data);
        //XXXXXXXXX为发送方钱包密码
        $txId = $transaction->send("XXXXXXXXXXXX");

        if ($txId && strlen($txId) == 66) {
            //返回交易hash
            return $txId;
        } else {
            return false;
        }
    }
?>
```