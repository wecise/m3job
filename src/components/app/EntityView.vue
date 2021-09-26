<template>
    <el-container style="height:100%;">
        <el-header style="height:auto;line-height:40px;min-height:40px;background: #f2f2f2;">
            <el-checkbox v-model="control.ifCheckStrictly" label="节点关联"></el-checkbox>
            <el-checkbox v-model="control.ifRelation" label="导出关系"></el-checkbox>
            <el-checkbox v-model="control.ifData" label="导出数据" style="margin-left:20px;"></el-checkbox>
            <p v-if="control.ifData">
                <el-radio-group v-model="control.ifAllData">
                    <el-radio :label="true" border>导出所有数据</el-radio>
                    <el-radio :label="false" border>导出部分数据</el-radio>
                </el-radio-group>
                <el-input-number v-model="control.limit" v-if="!control.ifAllData && control.ifData != -1" style="width:30%;margin-left:10px;"></el-input-number>  
            </p>
        </el-header>
        <el-main :loading="tree.loading">
            <el-tree
                :data="tree.data"
                ref="tree"
                highlight-current
                show-checkbox
                accordion
                node-key="class"
                :check-strictly="!control.ifCheckStrictly"
                :default-expanded-keys="tree.defaultExpandedKeys"
                :props="tree.defaultProps"
                style="background-color:transparent;">
                <span slot-scope="{ node, data }" style="width:100%;height:30px;line-height: 30px;"  @mouseenter="onMouseEnter(data)" @mouseleave="onMouseLeave(data)">
                    <span v-if="data.children">
                        <i class="el-icon-folder" style="color:#FFC107;"></i>
                        <span v-if="data.alias==''"> {{node.class}}所有类({{data.children.length}})</span>
                        <span v-else> {{ node.label }}({{data.children.length}})</span>
                    </span>
                    <span v-else>
                        <i class="el-icon-c-scale-to-original" style="color:#0088cc;"></i>
                        <span draggable="true"> 
                            {{ node.label }}
                        </span>
                    </span>
                </span>  
            </el-tree>
        </el-main>
        <el-alert
            style="margin-bottom:10px;position:relative;"
            type="warning"
            v-if="['/matrix/filesystem'].includes(control.class)"
            description="文件系统类只支持导出DDL，如需导出数据，请在【我的文件】里按目录进行备份及恢复...">
        </el-alert>
    </el-container>
</template>

<script>
import _ from 'lodash';

export default {
    props: {
        root: String
    },
    data(){
        return {
            control: {
                ifCheckStrictly: true,
                ifRelation: true,
                ifData: false, 
                ifAllData: false,
                limit: 10,
                recursive: true,
                filetype: 'mql',
                template: true,
                class: '',
                ignoreClass: ''
            },
            tree:{
                data: [],
                defaultExpandedKeys: [],
                defaultProps: {
                    children: 'children',
                    label: 'alias'
                },
                loading: true
            },
            filterText: ""
        }
    },
    watch: {
        filterText(val) {
            if(_.isEmpty(val)){
                this.initData();
            } else {
                this.$refs.tree.filter(val);
            }
        }
    },
    created(){
        this.initData();
    },
    methods: {
        onMouseEnter(item){
            this.$set(item, 'show', true)
            this.$refs.tree.setCurrentKey(item.key);
        },
        onMouseLeave(item){
            this.$set(item, 'show', false)
        },
        initData(){
            this.tree.loading = true;
            this.m3.callFS("/matrix/m3appstore/entity_class.js",encodeURIComponent(this.root)).then( (rtn)=>{
                this.tree.data = rtn.message;
                this.$emit("treedata-loaded",this.tree.data);
                this.tree.loading = false;
            }).catch(err=>{
                this.tree.loading = false;
            });
        },
        onFilterNode:_.debounce((value)=>{
            
            if (!value) return true;
            
            try{
                this.m3.callFS("/matrix/m3appstore/entity-search-by-term.js",encodeURIComponent(value)).then( (val)=>{
                    let rtn = val.message
                    this.tree.data = _.map(rtn,(v)=>{
                        return _.extend(v,{ children:[], alias:_.last(v.class.split("/"))});
                    });
                } );
            } catch(err){
                this.tree.data = [];
            }
        },1000),
        onNodeClick(data){
            this.$emit('node-click',data);
        }
    }
}
</script>
<style scoped>
    .el-container{
        height: 70vh;
    }
</style>