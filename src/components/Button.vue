<template>
    <div>
        <van-space>
            <van-button type="primary" @click="createWallet">创建钱包</van-button>
            <van-button type="primary">导入钱包</van-button>



            <van-dialog v-model:show="show" title="请输入密码" show-cancel-button @confirm='confirmPassword'>
                <van-cell-group inset>
                    <van-field v-model="password" label="密码" placeholder="请输入密码" type="password" />
                </van-cell-group>
            </van-dialog>
        </van-space>

        <template v-if="showMn">
            <p>
                {{ mnemontic }}
            </p>
            <van-button size="small" @click="comfirmSaveMnemontic">我已经保存</van-button>

            <van-dialog v-model:show="showMnDialog" title="请输入助记词" show-cancel-button @confirm='confirmMnemontic'>
                <van-cell-group inset>
                    <van-field v-model="mnemontic2" label="助记词" placeholder="请输入助记词" />
                </van-cell-group>
            </van-dialog>
        </template>

    </div>
</template>
<script setup>
import { ref } from 'vue'
import 'vant/es/dialog/style';
import 'vant/es/notify/style';
import { showDialog, showNotify } from 'vant';
import * as bip39 from 'bip39'
import ethwallet, { hdkey } from "ethereumjs-wallet"
import Web3 from 'web3';
import store2 from 'store2'

const show = ref(false);
const showMn = ref(false);
const showMnDialog = ref(false);
const password = ref('');
const mnemontic = ref('');
const mnemontic2 = ref('');
//创建钱包
const createWallet = () => {
    password.value = ""
    show.value = true
}
//确认密码
const confirmPassword = () => {
    console.log(password.value);
    if (password.value === '') {
        showNotify({ type: 'warning', message: '密码不能为空' });
    }
    else {

        const walletInfo = store2.get('walletInfo')
        console.log(walletInfo);

        if (walletInfo) {
            mnemontic.value = walletInfo[0].mnemontic
            console.log(mnemontic.value);
            confirmMnemontic()
        }
        else {
            mnemontic.value = bip39.generateMnemonic()
            showMn.value = true

        }

    }
}

//确认保存助记词
const comfirmSaveMnemontic = () => {
    mnemontic2.value = ''
    showMnDialog.value = true
}

//验证助记词
const confirmMnemontic = async () => {

    const walletList = store2.get('walletInfo') || []
    if (mnemontic2.value !== mnemontic.value && walletList.length===0 ) {
        showNotify({ type: 'warning', message: '助记词错误' });
        return;
    }

    showMn.value = false
    const seed = await bip39.mnemonicToSeed(mnemontic.value)
    const hdWallet = hdkey.fromMasterSeed(seed)

    const addressIndex = walletList.length
    const keyPair = hdWallet.derivePath(`m/44'/60'/0'/0/${addressIndex}`)
    const wallet = keyPair.getWallet()

    const lowerCaseAddress = wallet.getAddressString()
    const checkAddress = wallet.getChecksumAddressString()
    const priKey = wallet.getPrivateKey().toString("hex")
    const keyStore = await wallet.toV3(password.value)
    walletList.push({
        id: addressIndex,
        address: lowerCaseAddress,
        priKey,
        keyStore,
        mnemontic: mnemontic.value,
        balance: 0
    })
    store2('walletInfo', walletList)




}
</script>

<style scoped lang="less">
p {
    user-select: all;
}
</style>