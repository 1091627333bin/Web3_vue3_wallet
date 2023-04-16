<template>
  <div>
    <Button></Button>
    <accountList :walletInfo="walletInfo"></accountList>
    
  </div>
  
</template>
<script setup>
import { ref ,onMounted, reactive} from 'vue'
import Button from './components/Button.vue'
import accountList from './components/accountList.vue';
import store2 from 'store2'

const walletInfo = reactive(store2.get('walletInfo') || [] )

onMounted(() => {
  
})
</script>

<style  lang="less">
  body{
    padding: 10px;
  }
</style>