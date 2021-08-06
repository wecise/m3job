<template>
    <div class="container">
        <el-tabs value="install" type="border-card">
            <el-tab-pane name="deploy">
                <span slot="label">应用发布({{deployedApps.length}})</span>
                <el-container style="height:calc(100vh - 135px);">
                    <el-header style="line-height: 60px;">
                        <el-tooltip content="刷新">
                            <el-button type="default" icon="el-icon-refresh" @click="onRefresh">刷新</el-button> 
                        </el-tooltip>
                        <el-tooltip content="应用发布">
                            <el-button type="success" class="fileinput-button" >
                                <input type="file" name="loadfile" @change="onAppDeploy">
                                选择应用文件进行发布
                            </el-button>
                        </el-tooltip>

                    </el-header>
                    <el-main style="display: flex;flex-wrap: wrap;align-content: flex-start;justify-content: flex-start;padding:0 10px;">
                        <el-button type="default" 
                        style="position: relative;width: 14em;height:10em;border-radius: 8px!important;margin:10px;border: unset;box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);background:#262e48;"
                            v-for="(app,index) in deployedApps"
                            :key="index">
                            <el-image style="width:64px;height:64px;margin:5px;" src="/static/assets/images/apps/png/m3-app.png"></el-image>
                            <p style="color:#fff;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin:5px;text-align:center;">{{app.name}}</p>
                            <div style="position:absolute;top:0px;right:5px;">
                                <el-button type="text"  @click="onAppEdit(app)">
                                    <span class="el-icon-close"  style="color:#ffffff;" @click="onAppUndeploy(app)"></span> 
                                </el-button>
                            </div>
                        </el-button>
                    </el-main>

                </el-container>
            </el-tab-pane>
            <el-tab-pane name="install">
                <span slot="label">应用安装({{model.list.length}})</span>
                <el-container style="height:calc(100vh - 135px);">
                
                    <el-header style="line-height: 60px;">
                        <el-tooltip content="刷新">
                            <el-button type="default" icon="el-icon-refresh" @click="onRefresh">刷新</el-button> 
                        </el-tooltip>
                        <el-tooltip content="应用安装">
                            <el-button type="success" icon="el-icon-plus" @click="dialog.appInstall.show = true;">应用安装</el-button> 
                        </el-tooltip>
                        <el-dropdown style="float:right;">
                            <span class="el-dropdown-link">
                                <i class="el-icon-setting el-icon--right"></i>
                            </span>
                            <el-dropdown-menu slot="dropdown">
                                <el-dropdown-item @click.native="onAppExport('mql')">导出应用(MQL)</el-dropdown-item>
                                <el-dropdown-item @click.native="onAppExport('xlsx')">导出应用(Excel)</el-dropdown-item>
                                <el-dropdown-item @click.native="dialog.appImport.show = true;" divided>导入应用</el-dropdown-item>
                            </el-dropdown-menu>
                        </el-dropdown>
                    </el-header>
                    <el-main style="display: flex;flex-wrap: wrap;align-content: flex-start;justify-content: flex-start;padding:0 10px;">
                        <el-button type="default" 
                            style="position: relative;width: 14em;height:10em;border-radius: 8px!important;margin:10px;border: unset;box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);background:#262e48;"
                            v-for="(item,index) in model.list"
                            :key="index">
                            <el-image style="width:64px;height:64px;margin:5px;" :src="item.icon | pickIcon"></el-image>
                            <p style="color:#fff;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin:5px;text-align:center;">{{item.cnname}}</p>
                            <div style="position:absolute;top:0px;right:5px;">
                                <el-button type="text"  @click="onAppEdit(item)">
                                    <span class="el-icon-setting"  style="color:#ffffff;"></span> 
                                </el-button>
                            </div>
                        </el-button>
                        <el-dialog :title="'应用编辑 '+dialog.appEdit.item.name"  :visible.sync="dialog.appEdit.show" v-if="dialog.appEdit.show" destroy-on-close="true">
                            <el-tabs v-model="tabs.activeName" ref="tabs">
                                <el-tab-pane name="app">
                                    <span slot="label" style="font-size:14px;">
                                        <i class="el-icon-s-platform"></i> 应用信息
                                    </span>
                                    <el-container>
                                        <el-main style="padding:0px 20px;">
                                            <el-form ref="form" :model="dialog.appEdit.item" label-width="80px" >
                                                <el-form-item style="position:absolute;right:10px;">
                                                    <el-button type="text" @click="tabs.activeName='icon'" style="background:#444444;border-radius:15px!important;padding:20px;">
                                                        <el-image shape="square" fit="scale-down" style="width:64px;" :src="icon.value"></el-image>
                                                    </el-button>
                                                </el-form-item>
                                                <el-form-item label="中文名" style="width:75%;">
                                                    <el-input v-model="dialog.appEdit.item.cnname"></el-input>
                                                </el-form-item>
                                                <el-form-item label="英文名称" style="width:75%;">
                                                    <el-input v-model="dialog.appEdit.item.enname"></el-input>
                                                </el-form-item>
                                                <el-form-item label="Url" style="width:75%;">
                                                    <el-input v-model="dialog.appEdit.item.url"></el-input>
                                                </el-form-item>
                                                <el-form-item label="Target">
                                                    <el-radio-group v-model="dialog.appEdit.item.target">
                                                        <el-radio label="_blank">打开新窗口</el-radio>
                                                        <el-radio label="_parent">当前窗口打开</el-radio>
                                                    </el-radio-group>
                                                </el-form-item>
                                                <el-form-item label="分组">
                                                    <el-radio-group v-model="dialog.appEdit.item.groups.group">
                                                        {{dialog.appEdit.item.groups.group}}
                                                        <el-radio :label="item.name" v-for="item in model.groups" :key="item.name">{{item.title}}</el-radio>
                                                    </el-radio-group>
                                                </el-form-item>
                                            </el-form> 
                                        </el-main>
                                        <el-footer style="text-align:right;line-height:60px;">
                                            <el-button type="default" @click="dialog.appEdit.show = false;">取消</el-button>
                                            <el-button type="primary" @click="onAppUpdate(dialog.appEdit.item)">更新应用</el-button>
                                            <el-button type="danger" @click="onAppRemove(dialog.appEdit.item)">卸载应用</el-button>
                                        </el-footer>
                                    </el-container>
                                </el-tab-pane>
                                <el-tab-pane name="icon">
                                    <span slot="label" style="font-size:14px;">
                                        <i class="el-icon-picture"></i> 选择图标
                                    </span>
                                    <el-container style="height:100%;">
                                        <el-main style="height:300px;overflow:auto;padding:10px 0px;background:#666666;">
                                            <el-radio-group v-model="icon.value" class="mx-app-icon">
                                                <el-button type="default" style="border: unset;width:100px;height:120px;margin:5px;padding:0px;cursor:pointer;background:transparent;" 
                                                    v-for="icon in icon.list"
                                                    :key="icon.id"
                                                    @click="onTriggerRadioClick(icon)"> 
                                                    <el-image :src="icon | pickExtIcon" fit="fill"  style="width:48px;"></el-image> 
                                                    <p>
                                                        <el-radio :label="'/static'+icon.parent+'/'+icon.name" 
                                                            :ref="'radio_'+icon.id">
                                                        </el-radio>
                                                    </p>
                                                </el-button>
                                            </el-radio-group>
                                        </el-main>
                                        <el-footer style="padding:20px 0px 50px 0px;display:flex;height:auto;position:releative;">
                                            <span style="position:absolute;right:140px;">
                                                <el-button type="default" icon="el-icon-close" @click="tabs.activeName='app';">返回</el-button>
                                                <el-button type="primary" icon="el-icon-refresh" @click="initIconList">刷新图标</el-button>
                                            </span>
                                            <span style="position:absolute;right:20px;">
                                                <el-upload
                                                    multiple
                                                    :data="{index:true}"
                                                    :action="upload.root+'?issys=true'"
                                                    :before-upload="onBeforeUpload"
                                                    :on-success="onSuccess"
                                                    :on-error="onError"
                                                    :show-file-list="false"
                                                    name="uploadfile">
                                                    <el-button icon="el-icon-upload" type="primary" style="padding-left:20px;" :loading="upload.loading">上传图标</el-button>
                                                </el-upload>
                                            </span>
                                        </el-footer>
                                    </el-container>
                                </el-tab-pane>
                            </el-tabs>
                        </el-dialog>
                        <el-dialog title="应用发布" :visible.sync="dialog.appDeploy.show" v-if="dialog.appDeploy.show" destroy-on-close="true" :append-to-body="true">
                            <mx-app-deploy></mx-app-deploy>
                        </el-dialog>
                        <el-dialog title="应用安装" :visible.sync="dialog.appInstall.show" v-if="dialog.appInstall.show" destroy-on-close="true">
                            <mx-app-install :model="dialog.appInstall"></mx-app-install>
                        </el-dialog>
                        <el-dialog title="应用导入" :visible.sync="dialog.appImport.show" v-if="dialog.appImport.show" destroy-on-close="true">
                            <el-container>
                                <el-main>
                                    <el-upload
                                        class="upload-demo"
                                        drag
                                        :auto-upload="false"
                                        :on-change="onFilesChange"
                                        :file-list="dialog.appImport.files">
                                        <i class="el-icon-upload"></i>
                                        <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
                                        <div class="el-upload__tip" slot="tip">只能上传Mql/Excel文件</div>
                                    </el-upload>
                                </el-main>
                                <el-footer style="line-height:60px;text-align:right;">
                                    <el-button type="default" @click="dialog.appImport.show = false;">取消</el-button>
                                    <el-button type="primary" @click="onAppImport">导入</el-button>
                                </el-footer>
                            </el-container>
                        </el-dialog>
                    </el-main>
                </el-container>
            </el-tab-pane>
        </el-tabs>
    </div>
</template>

<script>
import _ from 'lodash';

export default{
    name: "AppView",
    data(){
        return {
            model: null,
            deployedApps: [],
            dialog: {
                appDeploy:{
                    show: false,
                    data: null
                },
                appInstall: {
                    show: false,
                    item: null
                },
                appEdit: {
                    show: false,
                    item: null
                },
                appImport: {
                    show: false,
                    files: null
                }
            },
            icon: {
                value: '',
                list: []
            },
            upload: {
                root: '/assets/images/apps/png',
                url: `${window.ASSETS_APP_ICON}`,
                fileList: [],
                loading: false
            },
            tabs: {
                activeName: "app"
            }
        }
    },
    mounted() {
        this.$nextTick( ()=>{
            this.init(); 
            this.initIconList();
        })
    },
    computed: {
        groupedModelList(){
            try{
                return _.groupBy(this.model.list, (v)=>{
                    return v.name;
                });
            } catch(err){
                return [];
            }
        }
    },
    filters:{
        pickExtIcon(icon) {
            return `/static${icon.parent}/${icon.name}`;
        },
        pickIcon(icon){
            return `/static/assets/images/apps/png/${icon}`;
        }
    },
    methods: {
        onRefresh(){
            this.init();
        },
        init(){
            this.m3.callFS("/matrix/m3appstore/appList.js",null).then( (rtn)=>{
                this.model = rtn.message;
            });

            this.m3.callFS("/matrix/m3appstore/getDeployedApps.js",null).then( (rtn)=>{
                this.deployedApps = rtn.message;
            });
        },
        initIconList(){
            /* fsHandler.fsListAsync(this.upload.root).then( (rtn)=>{
                this.icon.list = rtn;
                this.icon.value = `${this.upload.url}/creative.png`;
            } );    */
        },
        onTriggerRadioClick(item){
            this.$refs['radio_'+item.id][0].$el.click();
        },
        /* 取消应用发布 */
        onAppUndeploy(event){
            this.$confirm(`确定要取消发布应用 ${event.name}?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
                }).then(() => {
                    /* fsHandler.fsDeleteAsync(event.parent,event.name).then( (rtn)=>{
                        if (rtn == 1){
                            this.$message({
                                type: "success",
                                message: "取消发布成功！"
                            });
                            this.onRefresh();
                        } else {
                            this.$message({
                                type: "error",
                                message: "取消发布失败 " + rtn.message
                            })
                        }
                    } ); */

                }).catch(() => {
                    this.$message({
                        type: "info",
                        message: "操作已取消"
                    })
                });
        },
        /* 应用发布 */
        onAppDeploy(event){
    
            let file = event.target.files[0];
            
            const h = this.$createElement;
            this.$msgbox({
                    title: `应用发布`, 
                    message: h('span', null, [
                        h('h4', null, `提示：如果已发布该应用，默认将将覆盖原有应用，请确认！`),
                        h('p', null, `应用名称：${file.name}`),
                        h('p', null, `发布时间：${file.lastModifiedDate}`),
                        h('p', null, `应用大小：${this.m3.bytesToSize(file.size)}`)
                    ]),
                    showCancelButton: true,
                    confirmButtonText: '发布',
                    cancelButtonText: '取消',
                    type: 'warning',
                    customClass: "fs-custom-msgbox"
            }).then(() => {

                /* let rtn = fsHandler.fsUnZip("/app", file);

                this.$message({
                    type: "info",
                    message: "正在发布，请稍后。。。"
                })

                if(rtn == 1){
                    this.onRefresh();
                }

                event.target.value = null; */

            }).catch(() => {
                this.$message({
                    type: "info",
                    message: "发布操作已取消"
                })
            }); 
            
        },
        onAppUpdate(item){
            
            _.extend(item,{ icon: _.last(this.icon.value.split("/")), action: "update" } );
            
            this.m3.callFS("/matrix/m3appstore/app.js",encodeURIComponent(JSON.stringify(item))).then( (rtn)=>{
                if( _.lowerCase(rtn.status) == "ok"){
                    
                    this.$message({
                        type: "info",
                        message: "应用更新成功"
                    });
                    
                    this.dialog.appEdit.show = false;

                    /* eventHub.$emit("APP-REFRESH-EVENT");

                    // 刷新应用缓存，针对应用权限过滤
                    userHandler.refreshAppCache(); */
                }
            } );
        },
        onAppRemove(item) {
            
            this.$confirm(`确认要删除该应用：${item.name}？`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                
                this.m3.callFS("/matrix/m3appstore/app_delete.js",encodeURIComponent(JSON.stringify(item))).then( (rtn)=>{
                    
                    if(rtn.status === 'ok'){
                        
                        this.init();

                        /* // 刷新应用缓存，针对应用权限过滤
                        userHandler.refreshAppCache();

                        this.dialog.appEdit.show = false; */

                    } else {
                        this.$message({
                            type: "error",
                            message: rtn
                        })
                    }

                } );
                
            }).catch(() => {
                
            });

        },
        onAppEdit(item){
            this.dialog.appEdit.item = item;
            this.dialog.appEdit.show = true;
        },
        onAppExport(ftype){
            
            /* let template = ftype=='mql'?true:false;

            axios.get(`/mxobject/export?recursive=true&relation_defined=false&filetype=${ftype}&template=${template}&class=%2Fmatrix%2Fportal%2Ftools&ignoreclass=%2Fmatrix%2Ffilesystem&limit=-1`,{
                headers: {
                    "Content-type":"text/csv",
                    "Access-Control-Allow-Origin":"*"
                },
                responseType:"arraybuffer"
            })
            .then((response)=> {
                var blob = new Blob([response.data], ftype=='mql'?{type: "octet/stream"}:{type: "application/vnd.ms-excel"});
                saveAs(blob, `Apps-${moment().format('LLL')}.${ftype}`);
            })
            .catch((error)=> {
                console.error(error);
            }); */
        },
        onFilesChange(file){
            this.dialog.appImport.files = [file.raw];
        },
        onAppImport(){
            
            let fileName = this.dialog.appImport.files[0].name;

            this.$confirm(`确认要导入应用：${fileName}？`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                
                /* omdbHandler.classDataImport(this.dialog.appImport.files[0]).then( (rtn)=>{
                    if(_.lowerCase(rtn.status) == 'ok'){
                        this.$message({
                            type: "success",
                            message: "导入成功！"
                        });
                    } else {
                        this.$message({
                            type: "error",
                            message: "导入失败：" + rtn.message
                        });
                    }
                } ); */
                
            }).catch(() => {
                this.$message({
                    type: "info",
                    message: "取消导入操作！"
                });
            });
            
        },
        onBeforeUpload(){
            this.upload.loading = true;
        },
        onSuccess(res,file,FileList){
            this.upload.fileList = FileList;
            this.$message({
                type: "success",
                message: "上传成功！"
            })
            this.upload.loading = false;
            this.initIconList();
        },
        onError(err,file,FileList){
            this.$message({
                type: "error",
                message: "上传失败：" + err
            })
            this.upload.loading = false;
            this.initIconList();
        }
    }
}
</script>

<style scoped>
    .container{
        padding:0px;
    }

    .el-tabs--border-card {
        background: #fff;
        border: unset;
        box-shadow: unset;
    }
</style>