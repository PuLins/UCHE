<template>
    <el-form :model="addForm" :rules="addFormRules" ref="addForm" class="assnew">
            <el-collapse v-model="activeNames">
                    <el-collapse-item title="关联车辆信息" name="1">
                        <el-row :gutter="30">
                            <el-col :span="10">
                                <el-form-item label="平台来源" prop="platform" >
                                    <el-select v-model="addForm.platform" filterable  clearable placeholder="请选择平台来源" @visible-change="platChange">
                                        <el-option
                                          v-for="item in platOptions"
                                          :key="item.value"
                                          :label="item.label"
                                          :value="item.value">
                                        </el-option>
                                    </el-select>
                                </el-form-item>
                            </el-col>
                        </el-row>
                        <el-row :gutter="30">
                            <el-col :span="7">
                                <el-form-item label="" prop="mobile">
                                    <el-autocomplete
                                      v-model="addForm.mobile"
                                      :fetch-suggestions="querySearchMobile"
                                      placeholder="请输入车主手机"
                                      @select="handleSelectMobile"
                                    >
                                        <template slot="prepend">车主手机</template>
                                    </el-autocomplete>
                                </el-form-item>
                            </el-col>
                            <el-col :span="7">
                                <el-form-item label="" prop="name">
                                    <el-input  v-model="addForm.name" auto-complete="off">
                                        <template slot="prepend">车主姓名</template>
                                    </el-input>
                                </el-form-item>
                            </el-col>
                            <el-col :span="10">
                                <el-form-item label="车牌号/车架号" prop="licenseplatenum" >
                                    <el-select v-model="addForm.licenseplatenum" filterable  clearable placeholder="请选择车牌号/车架号" remote
                                        reserve-keyword
                                        :remote-method="remoteMethod">
                                        <el-option
                                          v-for="item in vehOptions"
                                          :key="item.id"
                                          :label="item.landv"
                                          :value="item.id">
                                        </el-option>
                                    </el-select>
                                </el-form-item>
                            </el-col>
                        </el-row>
                    </el-collapse-item>
                    <el-collapse-item title="关联人信息" name="2">
                        <el-row :gutter="50">
                            <el-col :span="8">
                                <el-form-item label="" prop="remobile">
                                    <el-input  v-model="addForm.remobile" auto-complete="off">
                                        <template slot="prepend">关联人手机</template>
                                    </el-input>
                                </el-form-item>
                            </el-col>
                            <el-col :span="8">
                                <el-form-item label="" prop="rename">
                                    <el-input  v-model="addForm.rename" auto-complete="off">
                                        <template slot="prepend">关联人</template>
                                    </el-input>
                                </el-form-item>
                            </el-col>
                        </el-row>
                    </el-collapse-item>
            </el-collapse>
            <div class="footer">
                <Button type="primary" @click.native="addSubmit" :loading="addLoading" icon="ios-plus-outline">添加关联</Button>
                <Button @click="goback">返回</Button>
            </div>
        </el-form>

</template>

<script>
    import util from '../../../styles/js/util'
    import {getOwnerMobile,getOwnerVin,addAssVeh} from '../../../api/api';
    export default {
        data () {
            // 验证手机
              var checkMobile = (rule, value, callback) => {
                var reg = /^1\d{10}$/g,
                flag = reg.test(value);
                if(!flag){
                  return callback(new Error('请输入11位的手机号码'));
                }else{
                  callback();
                }
              };
            return {
                activeNames: ['1','2'],
                //新增界面数据
                addForm: {
                    custid:'',
                    name:'',
                    mobile:'',
                    licenseplatenum:'',
                    platform:'C',
                    remobile:'',
                    rename:'',
                },
                users:[],//关联车辆组
                addLoading: false,
                addFormRules: {
                    remobile: [
                        { required: true, message: '请输入关联人手机号', trigger: 'blur' },
                        { validator: checkMobile,trigger: 'blur'}
                    ],
                },
                vehOptions:[],//当前车主所拥有车辆组的车牌号/车架号信息
                relOptions:[{
                    value: '0',
                    label: '车主'
                  }, {
                    value: '1',
                    label: '夫妻'
                  }, {
                    value: '2',
                    label: '亲属'
                }, {
                    value: '3',
                    label: '朋友'
                }, {
                    value: '4',
                    label: '同事'
                }, {
                    value: '5',
                    label: '其他'
                }],
                platOptions:[{
                    value: 'V',
                    label: 'VAS'
                  }, {
                    value: 'C',
                    label: '优车'
                  }, 
                ],
            }
        },
        methods: {
            goback:function(){
                this.$router.go(-1);
            },
            // 获取车主手机下拉列表
            querySearchMobile:function(queryString, cb){
                    let para = {
                        mobile : queryString,
                        platform:this.addForm.platform
                    },mobileArray=[];
                    getOwnerMobile(para).then((res) => {
                        res.data.data.records.forEach( function(item, index) {
                            mobileArray.push({value:item.mandn,id:item.custid,name:item.name});
                        });
                        cb(mobileArray);
                    });
            },
            handleSelectMobile:function(item){
                  this.addForm.name = item.name;
                  this.addForm.custid = item.id;
                  let para = {
                    custid :this.addForm.custid,
                    platform:this.addForm.platform
                  }
                  getOwnerVin(para).then((res) => {
                    this.vehOptions = res.data.data.records;
                  });
             },
             // 平台来源选择
             platChange:function(r){
                // 表单更新
                this.addForm.mobile = '';
                this.addForm.name = '';
                this.addForm.licenseplatenum = '';
                
                // 车架号列表更新
                let para = {
                    platform:this.addForm.platform
                  }
                  getOwnerVin(para).then((res) => {
                    this.vehOptions = res.data.data.records;
                  });
             },
             // 查询车架号/车牌号
             remoteMethod(query) {
                if (query !== '') {
                  this.loading = true;
                  setTimeout(() => {
                    this.loading = false;
                    let para = {
                        platform:this.addForm.platform,
                        vin:query
                    }
                    getOwnerVin(para).then((res) => {
                        this.vehOptions = res.data.data.records;
                    });
                  }, 200);
                } else {
                  this.vehOptions = [];
                }
              },
             //编辑
            addSubmit: function () {
                this.$refs.addForm.validate((valid) => {
                    if (valid) {
                        if (this.addForm.licenseplatenum != '') {//选择车辆时
                                this.addLoading = true;
                                this.users.push({mobile:this.addForm.remobile,tempname:this.addForm.rename});//添加关联人
                                let para = {
                                  vehicleid:this.addForm.licenseplatenum,
                                  users:this.users,
                                  platform:this.addForm.platform
                                }
                                addAssVeh(para).then((res) => {
                                    if (res.data.result.code == 0) {
                                        this.addLoading = false;
                                        this.$message ({
                                            message : '添加关联成功！',
                                            center: true,
                                            type: 'success'
                                        });
                                        this.$refs['addForm'].resetFields();
                                        this.$router.push({
                                              name: 'associatedVehicle'
                                        });
                                    }
                                });
                        }else{//未选择车辆时
                            this.$message ({
                                message : '请选择车牌号/车架号！',
                                center: true,
                                type: 'error'
                            });
                        }
                    }else{
                        this.$message ({
                                message : '标红项有误！',
                                center: true,
                                type: 'error'
                        });
                    }
                });
            },
        }
    }
</script>
