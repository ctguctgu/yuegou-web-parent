<template>
    <el-row>
        <el-col class="col col-left" span="8">
            <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
        </el-col>
        <el-col class="col col-right" span="16">
        </el-col>
    </el-row>
</template>

<script>
    export default {
        data(){
            return{
                data:[],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                }
            };
        },
        methods:{
            getProductTypes(){
                this.$http.post("/product/productType/loadTree").then(res=>{
                    this.data = res.data;
                })
            }
        },
        mounted() {
            this.getProductTypes();
        }
    }
</script>

<style scoped>
    .col{
        border:1px solid #ccc;
        height: 600px;
        margin-top: 20px;
    }
    .col-left{
        border-right: none;
        overflow-x: hidden;
        overflow-y: scroll;
    }
    .col-left::-webkit-scrollbar {
        display: none;
    }
    .el-tree{
        border:none
    }
</style>