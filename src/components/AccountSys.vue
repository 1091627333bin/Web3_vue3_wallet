<template>
    <div>
        <p>账号地址:0x1638585e807343a91bba1793d5a4de13fc6a2cd4</p>
        <p>账号私钥:7c7618a2a80c0f65666b5f67eefb1f59352a37f642eb0d9aea85b7a83fa2acdf</p>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import * as bip39 from 'bip39'
import ethwallet, { hdkey } from "ethereumjs-wallet"
import Web3 from 'web3';
var web3 = new Web3(Web3.givenProvider || "wss://goerli.infura.io/ws/v3/4c5b70e59c254df093e0cf9c0c6e2036");
//获取助记词
// const mnemontic = bip39.generateMnemonic()
// console.log(mnemontic);
const mnemontic = ref("gauge unhappy taste alone raw dish daring depart twenty balcony camp weapon")
//生成密钥对
bip39.mnemonicToSeed(mnemontic.value).then(seed => {
    // console.log(res);
    const hdWallet = hdkey.fromMasterSeed(seed)
    const keyPair = hdWallet.derivePath("m/44'/60'/0'/0/0")
    // console.log(keyPair);

    //获取私钥
    //1.获取钱包对象
    const wallet = keyPair.getWallet()
    // console.log(wallet);
    //2.获取钱包地址
    const lowerCaseAddress = wallet.getAddressString()
    console.log(lowerCaseAddress);

    //3.获取钱包校验地址
    const checkAddress = wallet.getChecksumAddressString()
    // console.log(checkAddress);

    //4.获取私钥
    const priKey = wallet.getPrivateKey().toString("hex")
    console.log(priKey);

    //导出keystore
    //1.web3js
    const keyStore = web3.eth.accounts.encrypt(priKey, "111111")
    // console.log(keyStore);

    //2.wallet对象
    wallet.toV3('111111').then(keyStore2 => {
        // console.log(keyStore2,'111');

        //通过keystore获取私钥  // 2.wallet
        ethwallet.fromV3(keyStore2, '111111').then(wallet2 => {
            const key = wallet2.getPrivateKey().toString("hex")
            console.log(key, "key");
        })

    })

    //通过keystore获取私钥
    // 1.web3
    const res = web3.eth.accounts.decrypt(keyStore, '111111')
    console.log(res, "res");
    
    //通过私钥获取地址
    const priKey2 = Buffer(priKey,"hex")
    const wallet3 = ethwallet.fromPrivateKey(priKey2)
    const lowerCaseAddress3 = wallet3.getAddressString() 
    console.log(lowerCaseAddress3,'lowerCaseAddress3');
   


})
</script>

<style scoped lang="less"></style>