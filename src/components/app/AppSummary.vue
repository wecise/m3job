<template>
    <div style="height:calc(100vh - 135px);overflow:auto;" v-if="summary">
        <template v-if="summary.today.length > 0">
            <div style="background:#f2f2f2;padding:20px;">
                <div style="display:flex;">
                    <el-image style="width:32px;height:32px;margin:5px;" src="/static/assets/images/apps/png/called.png"></el-image>
                    <div style="padding-left:10px;"><h3>今日发布 {{summary.today.length}}</h3></div>
                </div>
                <div style="display: flex;flex-wrap: wrap;align-content: flex-start;justify-content: flex-start;"> 
                    <el-button type="default" 
                    style="position: relative;width: 10em;height:4em;border-radius: 8px!important;margin:10px;"
                        v-for="(app,index) in summary.today"
                        :key="index">
                        <p style="color:#333333;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin:5px;text-align:center;">{{app.title || app.name}}</p>
                    </el-button>
                </div>
            </div>
        </template>
        <template v-if="summary.week.length > 0">
            <div style="background:#f2f2f2;padding:20px;margin-top: 10px;">
                <div style="display:flex;">
                    <el-image style="width:32px;height:32px;margin:5px;" src="/static/assets/images/apps/png/all.png"></el-image>
                    <div style="padding-left:10px;"><h3>本周发布 {{summary.week.length}}</h3></div>
                </div>
                <div style="display: flex;flex-wrap: wrap;align-content: flex-start;justify-content: flex-start;"> 
                    <el-button type="default" 
                    style="position: relative;width: 10em;height:4em;border-radius: 8px!important;margin:10px;"
                        v-for="(app,index) in summary.week"
                        :key="index">
                        <p style="color:#333333;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin:5px;text-align:center;">{{app.title || app.name}}</p>
                    </el-button>
                </div>
            </div>
        </template>
        <template v-if="summary.month.length > 0">
            <div style="background:#f2f2f2;padding:20px;margin-top: 10px;">
                <div style="display:flex;">
                    <el-image style="width:32px;height:32px;margin:5px;" src="/static/assets/images/apps/png/m3-app.png"></el-image>
                    <div style="padding-left:10px;"><h3>本月发布 {{summary.month.length}}</h3></div>
                </div>
                <div style="display: flex;flex-wrap: wrap;align-content: flex-start;justify-content: flex-start;"> 
                    <el-button type="default" 
                    style="position: relative;width: 10em;height:4em;border-radius: 8px!important;margin:10px;"
                        v-for="(app,index) in summary.month"
                        :key="index">
                        <p style="color:#333333;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin:5px;text-align:center;">{{app.title || app.name}}</p>
                    </el-button>
                </div>
            </div>
        </template>

    </div>
</template>

<script>
import _ from 'lodash';

export default {
    name: 'AppSummary',
    data(){
        return {
            summary: null
        }
    },
    mounted(){
        this.init();
    },
    methods: {
        init(){
            this.m3.callFS("/matrix/m3appstore/appStoreSummary.js", null).then( (rtn)=>{
            
                this.summary = rtn.message;

            })
        }
    }
}
</script>

<style scoped>
    
</style>
