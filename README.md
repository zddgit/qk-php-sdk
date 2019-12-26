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
        $rpc = new \RpcService();
        $system_address = "xxxxxxxxxxxxxx";//转出方钱包地址
        $system_password = "xxxxxxxxxxxxxxx";//转出方钱包密码
        //判断托管账号余额
        $params = array(
            [$system_address,"latest"]
        );
        //获取转出方地址余额信息
        $res_data = $rpc->rpc("eth_getBalance",$params);
        $res_data = isset($res_data[0])?$res_data[0]:array();
        $qki_balance = bcdiv(gmp_strval($res_data['result']) ,gmp_pow(10,18),8);

        //判断转出方余额是否足够
        if(bccomp($qki_balance,bcadd($num,1,8),8) < 0)
        {
            echo "转出地址余额不足";
            return false;
        }

        //转出方地址解锁
        $unlock_address_data = $rpc->rpc('personal_unlockAccount', [[$system_address, $system_password, 2]]);
        if (isset($unlock_address_data[0]['result']) && $unlock_address_data[0]['result']) {
            //余额格式处理，乘以位数，再转为16进制
            $amount = bcmul($num, 1000000000000000000, 0);
            $final_amount = base_convert($amount, 10, 16);
            //转账
            $data = [[[
                'from' => $system_address,
                'to' => $address,
                'value' => '0x' . $final_amount
            ]]];

            $result = $rpc->rpc('eth_sendTransaction', $data);
            //转账成功，返回交易HASH
            if (strlen($result[0]['result']) == 66) {
                return $result[0]['result'];
            } else {
                echo "转账失败";
                return false;
            }
        }else{
            echo "转出方地址解锁失败";
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
        
        //todo  xxxx为服务器运行的夸克区块链节点端口号，如果不是调用的当前服务器的节点，请填写所调用的服务器IP地址
        $url = "http://127.0.0.1:xxxx";
        //合约地址
        $url_arr = parse_url($url);
        //实例化通证
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
        //设置gas
        $transaction->gas(90000,"0.000000001");
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