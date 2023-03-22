<!-- 修改员工 -->
<template>
    <div class="main">
        <div class="navbar">
            <el-button type="primary" plain @click="$router.push({path: '/admin/control/staff/list', query:{ page:($route.query.page != undefined ? $route.query.page:'')}})">返回</el-button>
        </div>
        <div class="data-form label-width-blank userModule" >
            <el-form label-width="auto"  @submit.native.prevent>
                <el-form-item label="账号" :error="error.userAccount">
                    {{state.userAccount}}
                </el-form-item>
                <el-form-item label="姓名" :required="true" :error="error.fullName">
                    <el-col :span="12"><el-input v-model.trim="state.fullName" maxlength="20" :clearable="true" show-word-limit></el-input></el-col>
                </el-form-item>
                <el-form-item label="密码" :required="true" :error="error.userPassword">
                    <el-col :span="12"><el-input v-model.trim="state.userPassword" type="password" maxlength="20" :clearable="true" show-word-limit></el-input></el-col>
                </el-form-item>
                <el-form-item label="重复密码" :required="true" :error="error.repeatPassword">
                    <el-col :span="12"><el-input v-model.trim="state.repeatPassword" type="password" maxlength="20" :clearable="true" show-word-limit></el-input></el-col>
                </el-form-item>
                <el-form-item label="超级管理员" :required="true" :error="error.issys" v-if="state.isSysAdmin == true">
                    <el-radio-group v-model="state.issys">
                        <el-radio :label="true">是</el-radio>
                        <el-radio :label="false">否</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="备注"  :error="error.userDesc"  >
                    <el-input v-model.trim="state.userDesc" maxlength="150" :clearable="true" show-word-limit></el-input>					
                </el-form-item>
                <el-form-item label="职位" :error="error.userDuty">
                    <el-col :span="12"><el-input v-model.trim="state.userDuty" maxlength="30" :clearable="true" show-word-limit></el-input></el-col>
                </el-form-item>
                <el-form-item label="是否使用" :required="true" :error="error.enabled">
                    <el-radio-group v-model="state.enabled">
                        <el-radio :label="true">启用</el-radio>
                        <el-radio :label="false">停用</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="用户角色" >
                    <el-checkbox-group v-model="state.sysRolesId">
                        <el-checkbox :label="roles.id" :disabled="roles.logonUserPermission == false" :checked="roles.selected" v-for="roles in state.rolesList">{{roles.name}}</el-checkbox>
                    </el-checkbox-group>
                </el-form-item>
                
                
                
                <el-form-item label=" ">
                    <el-button type="primary" size="large" class="submitButton" @click="submitForm" :disabled="state.submitForm_disabled">提交</el-button>
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
    import { SysRoles } from '@/types';

    const store = useStore(pinia);
    const { proxy } = getCurrentInstance() as ComponentInternalInstance;
    const router = useRouter();


    const state = reactive({
        isSysAdmin:false,//是否是超级管理员
        rolesList:[] as Array<SysRoles>,
        
        userId :'',
        userAccount :'',
        fullName :'',
        userPassword :'',
        repeatPassword :'',
        issys :false,
        userDesc :'',
        userDuty :'',
        enabled :true,
        sysRolesId:[],

        submitForm_disabled:false,//提交按钮是否禁用
    });
    const error = reactive({
        userAccount :'',
        fullName :'',
        userPassword :'',
        repeatPassword :'',
        issys :'',
        userDesc :'',
        userDuty :'',
        enabled :'',
    });


    //查询修改员工
    const queryEditStaff = () => {

        proxy?.$axios({
            url: '/control/staff/manage',
            method: 'get',
            params: {
                method : 'editStaff',
			    userId : state.userId,
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
			    		let mapData = returnValue.data;
			    		for(let key in mapData){
			    			if(key == "isSysAdmin"){
			    				state.isSysAdmin = mapData[key];
			    			}else if(key == "sysUsers"){
			    				let sysUsers = mapData[key];
			    				state.userAccount= sysUsers.userAccount;
			    				state.fullName = sysUsers.fullName;
			    				state.issys = sysUsers.issys;
			    				state.userDesc = sysUsers.userDesc;
			    				state.userDuty = sysUsers.userDuty;
			    				state.enabled  = sysUsers.enabled;
			    				
			    			}else if(key == "sysRolesList"){
			    				state.rolesList = mapData[key];
			    			}
			    			
			    		}
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
        formData.append('userId', state.userId);
        formData.append('fullName', state.fullName);
        formData.append('userPassword', state.userPassword);
        formData.append('repeatPassword', state.repeatPassword);
        formData.append('issys', state.issys.toString());
        formData.append('userDesc', state.userDesc);
        formData.append('userDuty', state.userDuty);
        formData.append('enabled', state.enabled.toString());
    

        
        for(let i=0; i<state.sysRolesId.length; i++){
            let rolesId = state.sysRolesId[i];
            formData.append('sysRolesId', rolesId);
        }

        proxy?.$axios({
            url: '/control/staff/manage?method=editStaff',
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
							path : '/admin/control/staff/list',
							query:{ page:(router.currentRoute.value.query.page != undefined ? router.currentRoute.value.query.page:'')}
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

        if(router.currentRoute.value.query.userId != undefined && router.currentRoute.value.query.userId != ''){
			state.userId = router.currentRoute.value.query.userId as string;
		}
		
		
		
		queryEditStaff();
    }) 

</script>