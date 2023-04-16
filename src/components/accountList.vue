<template>
    <div>
        <van-cell :title="item.address" icon="user-o" v-for="item in newWalletInfo" :key="item.id">
            <!-- 使用 right-icon 插槽来自定义右侧图标 -->
            
            <template #right-icon>
                <van-button size="small" @click="getPassword(item.keyStore)">{{item.balance}}</van-button>
            </template>
        </van-cell>

        <van-dialog v-model:show="show" title="请输入密码" show-cancel-button @confirm='confirmPassword'>
                <van-cell-group inset>
                    <van-field v-model="password" label="密码" placeholder="请输入密码" type="password" />
                    <van-field v-model="toaddress" label="对象地址" placeholder="请输入地址" />
                    <van-field v-model="number" label="金额" placeholder="请输入金额数量" />
                </van-cell-group>
            </van-dialog>
    </div>
</template>


<script setup>
import { ref, defineProps ,computed} from 'vue'
import Web3 from 'web3'
import ethwallet from 'ethereumjs-wallet'
import Tx from 'ethereumjs-tx'
const show = ref(false)
const password = ref('')
const keyStore = ref('')
const toaddress = ref('')
const number = ref('')
var web3 = new Web3(Web3.givenProvider || "wss://goerli.infura.io/ws/v3/4c5b70e59c254df093e0cf9c0c6e2036");
const props = defineProps(['walletInfo'])
const newWalletInfo =   computed(()=>{
    props.walletInfo.forEach(async (item,index)=>{

        //  const res = await web3.eth.getBalance(item.address)
         
        
        const address = item.address

        const former = item.address.slice(0,8)
        const latter = item.address.slice(item.address.length-8)
        item.address = former + '......' + latter
        const res =await web3.eth.getBalance(address)
        item.balance =web3.utils.fromWei(res,'ether')
        
    })
    return props.walletInfo
})

const send = async (keystore,pass)=>{
    let walletobj;
    try {
        walletobj = await ethwallet.fromV3(keystore,pass)
    } catch (error) {
        alert('密码错误')
        return false;
    }
    let key = walletobj.getPrivateKey.toString("hex")
    var privateKey  = Buffer(key,"hex")
    console.log(privateKey,'privateKey');
    const fromaddress = "0x" + keystore.address

    //获取交易次数
    let nonce = await web3.eth.getTransactionCount(keystore.address)
    //获取预计转账gas费用
    let gasPrice = await web3.eth.getGasPrice();
    //转账金额为Wei为单位
    let balance = Web3.utils.toWei(number.value)

    var rawTx = {
        from:fromaddress,
        nonce,
        gasPrice,
        to:toaddress.value,
        value:balance,
        data:"0x00"
    }

    console.log(rawTx,'11');
    let gas = await web3.eth.estimateGas(rawTx)
    rawTx.gas = gas
    var tx = new Tx(rawTx)
    tx.sign(privateKey)
    var serializedTx  = tx.serialize();
    console.log(serializedTx);

    //Expected private key to be an Uint8Array with length 32
    web3.eth.sendSignedTransaction("0x"+serializedTx.toString("hex"))
    .on("transactionHash",(txid)=>{
        console.log('交易成功');
    })
    .on("error",(err)=>{
        console.log("error"+err);
    })
}



const getPassword = (keystore)=>{
    show.value = true
    keyStore.value = keystore
    password.value = ''
}

const confirmPassword = ()=>{
    console.log(keyStore.value.address,password.value)
    send(keyStore.value,password.value)
}

</script>


<style scoped lang="less"></style>