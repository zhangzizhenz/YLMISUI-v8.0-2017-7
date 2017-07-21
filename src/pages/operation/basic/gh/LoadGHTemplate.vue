<template>
    <div>
        <el-form  :model="formModel"    ref="formModel" label-width="100px">

            <el-tree :data="templateGHTreeData" 
            class="filter-tree"
            :expand-on-click-node="false" 
            :props="defaultProps" 
            @node-click="_handleNodeClick" 
            :filter-node-method="filterNode"
            :highlight-current="true"
            ref="tree2"
            style="height:250px;"></el-tree>

            <el-form-item style="text-align:right;" >
                <el-button type="primary" @click="_onSubmit" :loading="loading">导入</el-button>
            </el-form-item>

         </el-form>
    </div>
</template>

<script type="text/babel">
import {
            requestGetTemplateGHTreeList
        }   from 'api/templategh';
import {
            requestImportGHTemplate
        }   from 'api/gh'
export default {
    data(){
        return{
            templateGHTreeData:[],
            selectNode:{id:""},
            defaultProps: {
                        children: 'children',
                        label: 'text',
                        id: 'id'
                    },
            loading:false,
            formModel:{},
            filterText: '',
        }
    },
    props:{
        selectNds:{},
        isVisible:false
    },
    methods:{
        filterNode(value, data) {
             console.log(111111111111111111);
             if (!value) return true;
             return data.id == value;
        },
         _getTemplateGHTreeList(){
            requestGetTemplateGHTreeList().then(data => {
                            if(data.success){
                                this.templateGHTreeData=data.result;
                            }
                            else {
                                 this.$message.error('失败！'+data.error.message);
                            }
                        })
        },
         _handleNodeClick(result, resolve) {
                  this.selectNode=result; 
            },
        _onSubmit(){
                 if(this.selectNode.parent=='00000000-0000-0000-0000-000000000000'){
                        this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                            confirmButtonText: '确定',
                            cancelButtonText: '取消',
                            modal:false,
                            type: 'warning'
                            }).then(() => {
                            this.$message({
                                type: 'success',
                                message: '删除成功!'
                            });
                            this._submit();
                            }).catch(() => {
                            this.$message({
                                type: 'info',
                                message: '已取消删除'
                            });          
                            });
                 }
                else {
                    this.$message.error('失败！请选择父节点');
                }
        },
        _submit(){
             let params={};
            params.parentId=this.selectNode.id;
            requestImportGHTemplate(params).then(data=>{
            if(data.success){
                    this.$notify({
                        title: '成功',
                        message: '保存数据成功！',
                        type: 'success'
                        });
                    //关闭面板
                    this._complete();
                }
                else {
                    this.$message.error('失败！'+data.error.message);
                }
                    this.loading=false;
            })
        },
         _complete(){
                this.$emit('close');
            },
    },
    watch: {
         filterText(val) {
             this.$refs.tree2.filter(val);
             console.log(this.$refs);
      }
    },
    mounted(){
        //console.log(this.selectNds);
        this._getTemplateGHTreeList();
        this.filterText=this.selectNds.id;
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>
