<!-- 修改标签 -->
<template>
    <div class="main">
			<div class="navbar">
				<el-button type="primary" plain  @click="$router.push({path: '/admin/control/tag/list'})">返回</el-button>
			</div>
			<div class="data-form label-width-blank" >
				<el-form label-width="auto"  @submit.native.prevent>
					<el-form-item label="标签名称" :required="true" :error="error.name" >
						<el-input v-model.trim="state.name" maxlength="50" :clearable="true" show-word-limit></el-input>
					</el-form-item>
					<el-form-item label="排序" :required="true" :error="error.sort">
						<el-input-number v-model="state.sort" controls-position="right" :min="0" :max="999999999"></el-input-number>
						<div class="form-help" >数字越大越在前</div>
					</el-form-item>


					<el-form-item label=" ">
					    <el-button type="primary" size="large" class="submitButton" @click="submitForm" :disabled="state.submitForm_disabled">提交</el-button>
					    <div class="form-error" v-text="error.tag"></div>
					</el-form-item>
				</el-form>
				
				
			</div>
		</div>
</template>
<script lang="ts" setup>
    import { ComponentInternalInstance, getCurrentInstance, onMounted, reactive } from 'vue';
    import pinia from '@/store/store'
    import {useStore} from '@/store'
    import { AxiosResponse } from 'axios';
    import { useRouter } from 'vue-router';
    import { ElMessage } from 'element-plus';
    import { processErrorInfo } from '@/utils/tool';

    const store = useStore(pinia);
    const { proxy } = getCurrentInstance() as ComponentInternalInstance;
    const router = useRouter();


    const state = reactive({
        name :'',
        sort : 0,
        submitForm_disabled:false,//提交按钮是否禁用
    });
    const error = reactive({
        name :'',
        sort :'',
        tag :'',
    });

    //查询标签
    const queryTag = () => {
        
        if(router.currentRoute.value.query.tagId == undefined){
            ElMessage({
                duration :0,
                showClose: true,
                message: 'tagId不能为空',
                type: 'success',
                onClose: ()=>{
                    
                }
            })
        }

        proxy?.$axios({
            url: '/control/tag/manage',
            method: 'get',
            params: {
                method : 'edit',
                tagId: router.currentRoute.value.query.tagId,
            },
            //showLoading: false,//是否显示加载图标
            loadingMask:false,// 是否显示遮罩层
        })
        .then((response: AxiosResponse) => {
            if(response){
                const result: any = response.data;
                if(result){
                    let returnValue = JSON.parse(result);
                    if(returnValue.code === 200){//成功
                        let tag = returnValue.data;
			    		state.name = tag.name;
			    		state.sort = tag.sort;
                    }else if(returnValue.code === 500){//错误
                        //处理错误信息
                        processErrorInfo(returnValue.data as Map<string,string> , error,()=>{});

                    }
                }
            }
        })
        .catch((error: any) =>{
            console.log(error);
        });
    }



    //提交表单
    const submitForm = () => {
        state.submitForm_disabled = true;

        //清空error的属性值
        Object.keys(error).map(key => {
            Object.assign(error, {[key] : ''});
        })

        let formData = new FormData();
        if(router.currentRoute.value.query.tagId != undefined){
            formData.append('tagId', router.currentRoute.value.query.tagId as string);
            
        }
        if(state.name != null){
            formData.append('name', state.name);
            
        }
        if(state.sort != null){
            formData.append('sort', String(state.sort));
            
        }

        proxy?.$axios({
            url: '/control/tag/manage?method=edit',
            method: 'post',
            data: formData,
            //showLoading: false,//是否显示加载图标
            loadingMask:false,// 是否显示遮罩层
        })
        .then((response: AxiosResponse) => {
            if(response){
                const result: any = response.data;
			    if(result){
                    let returnValue = JSON.parse(result);
			    	if(returnValue.code === 200){//成功
                        ElMessage({
                            showClose: true,
                            message: '提交成功',
                            type: 'success',
                            onClose: ()=>{
                                
                            }
                        })
			    		//删除缓存
                        store.setCacheNumber();
			    		router.push({
							path : '/admin/control/tag/list',
						});
			    	}else if(returnValue.code === 500){//错误
			    		//处理错误信息
                        processErrorInfo(returnValue.data as Map<string,string> , error,()=>{});

			    		
			    	}
                }
                state.submitForm_disabled = false;//提交按钮disabled状态
            }
        })
        .catch((error: any) =>{
            console.log(error);
            state.submitForm_disabled = false;//提交按钮disabled状态
        });
    }

    onMounted(() => {
        //设置缓存
        store.setCacheComponents(String(router.currentRoute.value.name))
        queryTag()
    }) 

</script>