<template>
    <div class="table">
        <!-- 分销商类别设置 -->
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-menu"></i> 基础设置</el-breadcrumb-item>
                <el-breadcrumb-item>分销商类别设置</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="plugins-tips">
            <el-button icon="edit" type="primary" @click="dialogFormVisible=true">新 增</el-button>
            <el-dialog title="新增" :visible.sync="dialogFormVisible" size="tiny">
                <el-form :model="form" ref="numberValidateForm">
                    <el-form-item label="分销商类别:" :label-width="formLabelWidth" prop="name" :rules="[{ required: true, message: '分销商类别不能为空'}]">
                        <el-input v-model="form.name" auto-complete="off" placeholder="请选择活动区域"></el-input>
                    </el-form-item>
                    <el-form-item label="加价率(%):" :label-width="formLabelWidth" prop="condPercent" :rules="[{ required: true, message: '加价率不能为空'},{ type: 'number',max: 1.0, message: '加价率必须为不大于1.0的数字值'},]">
                        <el-input type="condPercent" v-model.number="form.condPercent" auto-complete="off" placeholder="请输入加价率(不超过1.0)"></el-input>
                    </el-form-item>
                    <el-form-item label="加价金额(元):" :label-width="formLabelWidth" prop="condYuan" :rules="[{ required: true, message: '加价金额不能为空'},{ type: 'number', message: '加价金额必须为数字值'},]">
                        <el-input v-model.number="form.condYuan" auto-complete="off" placeholder="请输入加价金额"></el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click="dialogFormVisible = false">取 消</el-button>
                    <el-button type="primary" @click="submitForm('numberValidateForm')">提 交</el-button>
                </div>
            </el-dialog>
        </div>
        <el-table :data="tableData" border stripe style="width: 100%" v-loading="loading" element-loading-text="玩儿命加载中···">
            <el-table-column type="index" align=center label="编号" width="100">
            </el-table-column>
            <el-table-column align=center prop="name" label="分销商类别">
            </el-table-column>
            <el-table-column align=center prop="condPercent" label="加价率(%)">
            </el-table-column>
            <el-table-column align=center prop="condYuan" label="加价金额(元)">
            </el-table-column>
            <el-table-column align=center label="操作">
                <template scope="scope">
                    <el-button type="success" size="small">修 改</el-button>
                    <el-button type="danger" size="small">删 除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <div class="grid-content bg-purple">
            <div class="block">
                <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="currentPage" :page-size="10" :page-sizes="[10, 20]" layout="total, prev, pager, next" :total="total">
                </el-pagination>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import Datasource from 'vue-datasource';
import Paginations from './Pagination.vue';     //分页组件
import common from '../../kits/commonapi.js';  //公共域名文件

export default {
    data: function() {
        const self = this;
        return {
            total: 1,
            pagingNowNumberList: 1,   //当前显示页码数据
            tableData: [],      //当前表格数据
            currentPage: 1,    //当前页码数
            pageCount: "",    //总的页码数
            loading: true,
            dialogFormVisible: false,
            form: {
                name: '',
                condPercent: '',
                condYuan: '',
                type: [],
            },
            formLabelWidth: '110px',
        };
    },
    components: {
        Datasource, Paginations,
    },
    watch: {

    },
    ready() {

    },
    created() {
        this.getimgs();
    },
    methods: {
        getimgs() {
            axios.get(common.apidomain + "/customType/findPageData.action?pageIndex=" + this.pagingNowNumberList).then((res) => {
                // console.log(res.data.data);
                this.tableData = res.data.data.datas;   //表格数据
                this.total = res.data.data.allCount;    //条数
                this.pageCount = res.data.data.pageCount;   //总的页码数
                this.pagingNowNumberList = res.data.data.currPage;   //当前页码数
                this.loading = false;
            })
        },
        handleSizeChange(val) {
            // console.log(`每页 ${val} 条`);
            this.pagingNowNumberList = `${val}`;
        },
        handleCurrentChange(val) {
            // console.log(`当前页: ${val}`);
            axios.get(common.apidomain + "/customType/findPageData.action?pageIndex=" + `${val}`).then((res) => {
                this.tableData = res.data.data.datas;   //表格数据
                this.total = res.data.data.allCount;    //条数
                this.pageCount = res.data.data.pageCount;   //总的页码数
                this.pagingNowNumberList = res.data.data.currPage;   //当前页码数
                this.loading = false;
            })
        },
        // 新增数据
        submitForm(formName) {
            this.$refs[formName].validate((valid) => {
                if (valid) {
                    this.$message({
                        showClose: true,
                        message: '提交成功',
                        type: 'success'
                    });
                    this.dialogFormVisible = false;
                } else {
                    this.$message({
                        showClose: true,
                        message: '提交失败,请重试',
                        type: 'warning'
                    });
                    return false;
                }
            });
            // 提交表单数据
            axios.post(common.apidomain + "/customType/add.action?name=" + this.form.name + "&condPercent=" + this.form.condPercent + "&condYuan=" + this.form.condYuan + "&pageIndex=" + this.pageCount).then((res) => {
                this.tableData = res.data.data;   //表格数据
                this.currentPage = this.pageCount;
                // console.log(res.data.data)
                // console.log(this.pageCount)
            })
                this.form.name = "",
                this.form.condPercent = "",
                this.form.condYuan = ""
        },
    },
    computed: {

    },
    beforeMount() {
    }
}
</script>

<style scoped>
body {
    font-family: "Segoe UI";
}

li {
    list-style: none;
}

a {
    text-decoration: none;
}

.pagination {
    position: relative;
}

.pagination li {
    display: inline-block;
    margin: 0 5px;
}

.pagination li a {
    padding: .5rem 1rem;
    display: inline-block;
    border: 1px solid #ddd;
    background: #fff;
    color: #0E90D2;
}

.pagination li a:hover {
    background: #eee;
}

.pagination li.active a {
    background: #0E90D2;
    color: #fff;
}









/* 分页组件层叠样式 */

.el-pagination {
    margin-left: 35%;
    margin-top: 4rem;
}

.el-pagination button,
.el-pagination span {
    display: inline-block;
    font-size: 13px;
    min-width: 48px;
    height: 45px !important;
    line-height: 48px;
    vertical-align: top;
    box-sizing: border-box;
}
</style>