<template>
    <el-config-provider :locale="localLanguage">
        <router-view></router-view> 
    </el-config-provider>
</template>

<script setup lang="ts">
  import { getCurrentInstance, ComponentInternalInstance} from 'vue'
  import pinia from '@/store/store'
  import {useStore} from '@/store'
  import { storeToRefs } from 'pinia';
  import { useRoute } from 'vue-router';
  //国际化 -- 指定显示中文
  import localLanguage from 'element-plus/lib/locale/lang/zh-cn'//中文
  const route = useRoute();


  const store = useStore(pinia);
  

  const { proxy } = getCurrentInstance() as ComponentInternalInstance;


 
  


</script>

