﻿<template>
<div>
  <el-row class="yl-order-header">
    <el-col :span="2">
      <img src="../../../common/image/zgztlogo.png" alt="logo" class="yl-order-logo">
      </el-col>
    <el-col :span="18" > 
     <div class="yl-order-title"><span class="yl-order-small-title">{{title}}&nbsp;&nbsp;&nbsp;</span>不合格登记</div>
    </el-col>
      <el-col :span="4">
      <div class="yl-order-code"><span style="color: red;">NO.</span>{{formModel.order.orderCode}}</div>
      </el-col>
 </el-row>
  <el-form  :model="formModel.order"  :rules="rules" ref="formModel" label-width="100px" >
<el-row>
        <el-col :span="8"> 
                   <el-form-item label="记账单位" prop="orgId">
                      <yl-organizeTree
                      v-model="formModel.order.orgId"
                      :placeholder="'请选择记账单位'"
                      :isexpand="true"
                      @getCurrentNode="_getCurrentNodeOrg"
                      style="width:80%">
                      </yl-organizeTree>
                   </el-form-item>
       </el-col>
        <el-col :span="8"> 
             <el-form-item label="账期" prop="orderDate">
                         <el-date-picker
                            v-model="formModel.order.orderDate"
                            @change="_dateSel"
                            type="month"
                            :editable="false"
                            placeholder="请选择账期"
                            style="width:80%">
                         </el-date-picker>
         </el-form-item>
       </el-col>
        <el-col :span="8"> 
         <el-form-item label="供应单位" prop="supplierName">
            <yl-comsupplier 
                    v-model="formModel.order.supplierName" 
                    placeholder="请选供应单位"
                    @selectNode="selectNodeSupplier" 
                     @clearBack="_siconClick"
                    style="width:80%;">
           </yl-comsupplier>
         </el-form-item>
       </el-col>
    </el-row>
    <el-row>
       <el-col :span="8"> 
            <el-form-item label="合同" prop="contractCode">
                 <yl-Contract v-model="formModel.order.contractCode" 
                 ref="contractref" 
                 :supplierId="formModel.order.supplierId" 
                 :disabled="contractDisable"  
                 placeholder="请选择合同"
                 @selectNode="selectNodeContract" 
                 style="width:80%"></yl-Contract>
            </el-form-item>
        </el-col>
        <el-col :span="8"> 
         <el-form-item label="车牌号" prop="wagonNumber">
             <el-input v-model="formModel.order.wagonNumber" style="width:80%" ></el-input>
         </el-form-item>
       </el-col>
        <el-col :span="8"> 
         <el-form-item label="运输单号" prop="orderNumber">
             <el-input v-model="formModel.order.orderNumber" style="width:80%" ></el-input>
         </el-form-item>
       </el-col> 
     </el-row>
     <el-row class="yl-row-TableEditor">
      <el-col :span="24"> 
        <yl-tableedit ref="tableedit"
            :tableConfig="tableEditConfig"
            :tableData="formModel.items"
            :sumRowConf="sumRowConf"
            :mapConfig="tableEditMapConfig"
            @delItem="_delItem"
        ></yl-tableedit>
       </el-col>
     </el-row>
   <el-row>
      <el-form-item label="备注" prop="remark">
          <el-input v-model="formModel.order.remark" type="textarea" row="5" style="width:95%"></el-input>
      </el-form-item>
   </el-row>
    <el-row>
        <el-col :span="8"> 
            <el-form-item label="签字人一：" prop="personOne">
            </el-form-item>
        </el-col>
        <el-col :span="8"> 
            <el-form-item label="签字人二：" prop="personTwo">
            </el-form-item>
        </el-col>
        <el-col :span="8"> 
            <el-form-item label="制单人：" prop="maker">
                <span>{{formModel.order.maker}}</span>
            </el-form-item>
        </el-col>
    </el-row>
   <el-form-item style="text-align:right;" >
      <el-button type="primary" @click="_onSelect"><i class="icon-plus"></i>进料登记选材</el-button>
      <el-button type="primary" @click="_onSubmit" :loading="loading"><i class="icon-floppy-disk"></i>保存</el-button>
  </el-form-item>
  
</el-form>


   <el-dialog ref="selectDialog2" title="选择"  
                 v-model="addFormVisible" size="large"  
                 top="8%" 
                 :modal="false"
                 :lock-scroll="true"
                 :close-on-press-escape="false"
                 :modal-append-to-body="false" 
                 :close-on-click-modal="false"  >
                <yl-selectFromRequestionIn    
                    :isSelectClose="true"
                    @close="_close"
                    v-if="addFormVisible">
                </yl-selectFromRequestionIn>
  </el-dialog>


</div>
</template>
<script type="text/babel">
import selectFromRequestionIn from 'ocomponents/selectmodule/selectFromRequestionIn'
import config from 'static/js/config';
import {
            requestCreateOrUpdateRequistionInNotQulifyOrder,
            requestGetRequistionInNotQulifyOrderForEdit,
            requestDeleteRequistionInNotQulifyOrderItem,
        }   from 'api/materials/requistioninnotqulifyorder';
import organizeTree from 'ocomponents/organize/organizeTree';
import ComSupplier from 'ocomponents/comsupplier/ComSupplier';
import Contract from 'ocomponents/contract/Contract'
export default {
    data(){
        return{             
            formModel:{
                    order:{
                             orderId:'',
                             orgId:this.getUserInfo().user.manageOrgId,
                             orderDate:new Date(),
                             orderCode:'',
                             contractId:'',
                             contractCode:'',
                             supplierId:'',
                             supplierName:'',
                             wagonNumber:'',
                             orderNumber:'',
                             isAudit:'',
                             auditor:'',
                             auditDate:'',
                             isState:'',
                             maker:'',
                             makerDate:'',
                             personOne:'',
                             personTwo:'',
                             personThree:'',
                             personFour:'',
                             personFive:'',
                             remark:'',
                        },
                        items:[],
                        config:{isGenerateCode:true},
            },
            rules: {
                            orgId: [
                              { required: true, message: '记账单位不能为空！', trigger: 'change' }
                            ],
                            orderDate: [
                              {type:'date', required: true, message: '账期不能为空！', trigger: 'blur' }
                            ],
                            supplierName: [
                              { required: true, message: '供应单位不能为空！', trigger: 'change' }
                            ],
                             contractCode: [
                              { required: true, message: '合同编号不能为空！', trigger: 'change' }
                            ],
                        
            },
             sumRowConf:{
                              disable:true, //是否显示
            //                 //sumText:'合计', //合计文本
            //                 //items:{
            //                   //合计项配置
            //                   //  planQuantity:{text:'数量合计',sumval:0.00,capitalCol:false},
            //                   //  lossQuantity:{text:'损耗量合计',sumval:0.00,capitalCol:false},
            //                   //  responseSum:{text:'金额合计',sumval:0.00,capitalCol:true}
            //                 }
                        },
            loading:false,
            addFormVisible:false,
            orderDateInital:'',
            title:config.Order.title,
            tableEditMapConfig:{},
            contractDisable:true,
         }       
    },
    props:{
        selectRow:{
            type:Object,
            default:{},
        },
        isVisible:{
             type:Boolean,
             default:false,
        }
    },
    computed: { 
        tableEditConfig:{
                get () {
                    return {
                        table: {
                            attr: {
                                highlightCurrent:true,
                                style:{width:'100%',height:'300px'},
                            }
                        },
                        columns: [
                            {attr: { type: 'index',label: '序号', width:65,align: 'center' }},
                              { attr: { prop: 'id', label: '操作', width: 65, scopedSlot: 'id', } },
                            //列配合
                                {attr: { prop: 'infoName', label: '名称', width:100,  } },   
                                {attr: { prop: 'infoModel', label: '规格', width:100,  } },   
                                {attr: { prop: 'infoUnit', label: '单位', width:100,  } },   
                                {attr: { prop: 'quantity', label: '进料数量', width:100,  scopedSlot:'quantity'} },   
                                {attr: { prop: 'notQualifyQuantity', label: '不合格数量', width:100, scopedSlot:'notQualifyQuantity' } },   
                                {attr: { prop: 'bookPrice', label: '账面价', width:100, scopedSlot:'bookPrice' } },  
                                {attr: { prop: 'taxRate', label: '税率', width:100,  scopedSlot:'taxRate' } },  
                                {attr: { prop: 'quantitySurfaceDesc', label: '数量及外观描述', width:150, scopedSlot:'quantitySurfaceDesc' } },   
                                {attr: { prop: 'notQualifyResult', label: '不合格处理结果', width:150, scopedSlot:'notQualifyResult' } },   
                                {attr: { prop: 'checkDate', label: '检查日期', width:150,  scopedSlot:'checkDate'} },   
                                {attr: { prop: 'remark', label: '备注', width:100,  scopedSlot:'remark'} },   
                                {attr: { prop: 'barCode', label: '条码', width:100,  } },   
                               // {attr: { prop: 'cId', label: '', width:100,  } },   
                                //{attr: { prop: 'classNodebh', label: '类别编码', width:100,  } },   
                                {attr: { prop: 'manufacturer', label: '生产厂商', width:100,  } },   
                                {attr: { prop: 'batchNo', label: '炉批号', width:100,  } },     
                            //注意：最后一列如果不设置列宽将会自动宽度补全
                        ],
                        
                        
                        //编辑列具名配置
                        solSlotConf:[ {  type:'input',   // 类型  目前 支持 input switch datepick
                                         name:'quantity',  //字段名称
                                        elmentConfig:{  //控件配置   参照elementui配置参数
                                                maxlength:1000000, 
                                                minlength:0,
                                                size:"small",
                                                disabled:false,
                                                type:"text",
                                                placeholder:"",
                                                icon:"",
                                                readonly:false, }, 
                                         eventconf:{     //事件配置  目前只支持input的 blur事件
                                                ison:true,  //开启事件
                                                event:(row,name)=>{  //事件逻辑
                                                    if(this.checkdecimalfn(row.quantity)){
                                                       
                                                    }else{
                                                         row.quantity=0.00;
                                                         this.$message.error('输入类型有误！');
                                                    }
                                                    this._sumCount();
                                                },
                                             }
                                         },
                                         {
                        type: 'input', name: 'taxRate',
                        elmentConfig: {
                            maxlength: 1000000,
                            minlength: 0,
                            size: "small",
                            disabled: false,
                            type: "text",
                            placeholder: "",
                            icon: "",
                            readonly: false,
                        },
                         eventconf:{  //事件配置  目前只支持input的 blur事件
                                                ison:true,  //开启事件
                                                event:(row,name)=>{  //事件逻辑
                                                    if(this.checkdecimalfn(row.taxRate)){
                                                    }else{
                                                         row.taxRate=17;
                                                         this.$message.error('输入类型有误！');
                                                    }
                                                    this._sumCount();
                                                },
                                             }
                                         },
                     {
                        type: 'input', name: 'notQualifyQuantity',
                        elmentConfig: {
                            maxlength: 1000000,
                            minlength: 0,
                            size: "small",
                            disabled: false,
                            type: "text",
                            placeholder: "",
                            icon: "",
                            readonly: false,
                        },
                         eventconf:{  //事件配置  目前只支持input的 blur事件
                                                ison:true,  //开启事件
                                                event:(row,name)=>{  //事件逻辑
                                                    if(this.checkdecimalfn(row.notQualifyQuantity)){
                                                    }else{
                                                         row.notQualifyQuantity=0.00;
                                                         this.$message.error('输入类型有误！');
                                                    }
                                                    this._sumCount();
                                                },
                                             }
                                         },
                     {
                        type: 'input', name: 'bookPrice',
                        elmentConfig: {
                            maxlength: 1000000,
                            minlength: 0,
                            size: "small",
                            disabled: false,
                            type: "text",
                            placeholder: "",
                            icon: "",
                            readonly: false,
                        },
                        eventconf:{  //事件配置  目前只支持input的 blur事件
                                                ison:true,  //开启事件
                                                event:(row,name)=>{  //事件逻辑
                                                    if(this.checkdecimalfn(row.bookPrice)){
                                                    }else{
                                                         row.bookPrice=0.00;
                                                         this.$message.error('输入类型有误！');
                                                    }
                                                    this._sumCount();
                                                },
                                             }
                                         },
                     {
                        type: 'input', name: 'quantitySurfaceDesc',
                        elmentConfig: {
                            maxlength: 1000000,
                            minlength: 0,
                            size: "small",
                            disabled: false,
                            type: "text",
                            placeholder: "",
                            icon: "",
                            readonly: false,
                        },
                        eventconf: {
                            ison: true,
                            event: (row) => {
                            },
                        }
                    },
                     {
                        type: 'input', name: 'notQualifyResult',
                        elmentConfig: {
                            maxlength: 1000000,
                            minlength: 0,
                            size: "small",
                            disabled: false,
                            type: "text",
                            placeholder: "",
                            icon: "",
                            readonly: false,
                        },
                        eventconf: {
                            ison: true,
                            event: (row) => {
                            },
                        }
                    },
                     {type: 'datePicker', 
                     name: 'checkDate',
                        elmentConfig: {
                            size: "small",
                            disabled: false,
                            type: "date",
                            placeholder: "",
                            readonly: false,
                        },       
                        eventconf: {
                            ison: false,
                            event: (row) => {        
                            },
                        }
                    },
                     {
                        type: 'input', name: 'remark',
                        elmentConfig: {
                            maxlength: 1000000,
                            minlength: 0,
                            size: "small",
                            disabled: false,
                            type: "text",
                            placeholder: "",
                            icon: "",
                            readonly: false,
                        },
                        eventconf: {
                            ison: true,
                            event: (row) => {
                            },
                        }
                    },
                                    
                        ],
                        
                    }
                }
            },
        MapConfig1:{
             get () {
                 return {
                     mapconf: [   
                                     //明细编辑列和选材控件列映射配置
                                { prop: 'id', isplace: false, defaultval:'0',selprop: '' },    //主键列
                                { prop: 'dataId', isplace: true, defaultval:'',selprop: 'id' }, //来源ID
                                { prop: 'dataType', isplace: false, defaultval:'0',selprop: '' },  //来源类型  // 0 常用材料库 1 进料登记选材 2 从总需用计划选材  3 进料登记（30进料登记PDA选材 31发料登记选材 32内部调拨登记选材 33外部调拨登记选材 ）4从磅单选材 5 合同选材 6从库存选材     
                                { prop: 'guid', isplace: false, defaultval:this.newGuid(),selprop: '' },  //临时ID唯一键
                                { prop: 'itemId', isplace: true,defaultval:'', selprop:'itemId',  },   
                                { prop: 'orderId', isplace: true,defaultval:'', selprop:'orderId',  },   
                                { prop: 'orgId', isplace: true,defaultval:'', selprop:'orgId',  },   
                                { prop: 'infoCode', isplace: true,defaultval:'', selprop:'infoCode',  },   
                                { prop: 'infoName', isplace: true,defaultval:'', selprop:'infoName',  },   
                                { prop: 'infoModel', isplace: true,defaultval:'', selprop:'infoModel',  },   
                                { prop: 'infoUnit', isplace: true,defaultval:'', selprop:'infoUnit',  },   
                                { prop: 'barCode', isplace: true,defaultval:'', selprop:'barCode',  },   
                                { prop: 'cId', isplace: true,defaultval:'', selprop:'cId',  },   
                                { prop: 'classNodebh', isplace: true,defaultval:'', selprop:'classNodeBh',  },   
                                { prop: 'manufacturer', isplace: true,defaultval:'', selprop:'manufacturer',  },   
                                { prop: 'batchNo', isplace: true,defaultval:'', selprop:'batchNo',  },   
                                { prop: 'taxRate', isplace: false,defaultval:'17', selprop:'',  },   
                                { prop: 'quantity', isplace: false,defaultval:'0', selprop:'',  },   
                                { prop: 'notQualifyQuantity', isplace: false,defaultval:0, selprop:'',  },   
                                { prop: 'bookPrice', isplace: true,defaultval:'', selprop:'bookPrice',  },   
                                { prop: 'quantitySurfaceDesc', isplace: false,defaultval:'', selprop:'',  },   
                                { prop: 'notQualifyResult', isplace: false,defaultval:'', selprop:'',  },   
                                { prop: 'checkDate', isplace:false,defaultval:'', selprop:'',  },   
                                { prop: 'remark', isplace: false,defaultval:'', selprop:'',  },   
                                    
                               
                               
                            ] 
                 }
             }
         },
    }, 
    methods:{
         _sumCount(colName){
             //计算列的合计值  合计方法计算逻辑
          //  let planQuantityTotal=0.00,lossQuantityTotal=0.00,responseSumTotal=0.00;
          //  this.formModel.items.map((item)=>{
                //凡是手动输入后的input都会转化为str  因此计算时要转化一下。
              //  planQuantityTotal=planQuantityTotal+parseFloat(item.planQuantity);
              //  lossQuantityTotal=lossQuantityTotal+parseFloat(item.lossQuantity);
              //  responseSumTotal=responseSumTotal+item.responseSum;
        //    })
           // this.sumRowConf.items.planQuantity.sumval=planQuantityTotal.toFixed(2);
           // this.sumRowConf.items.lossQuantity.sumval=lossQuantityTotal.toFixed(2);
          //  this.sumRowConf.items.responseSum.sumval=responseSumTotal.toFixed(2);
         },
         _formInitial(){
            //初始化表单
            requestGetRequistionInNotQulifyOrderForEdit(this.selectRow.id).then(data=>{
                if(data.success){
                     Object.assign(this.formModel,data.result); 
                     this.orderDateInital=this.formModel.order.orderDate;
                     this.formModel.order.orderDate=new Date(this.formModel.order.orderDate);
                     this._sumCount();
                }else{
                     this.$message.error('获取编辑表信息失败！');
                }
            });
          },
         _delItem(row){
                this.$confirm('确认要删除所选项目吗?', '提示', {
                type: 'warning',
                modal:false
                }).then(() => {
                        if(row.id!=="0"){
                            //后台删除
                            let obj={};
                            obj.id=row.id;
                            requestDeleteRequistionInNotQulifyOrderItem(obj).then(data=>{
                                if(data.success){
                                     let index=this.formModel.items.indexOf(row);
                                     this.formModel.items.splice(index,1);
                                      this._sumCount();
                                    this.$notify({
                                            title: '成功',
                                            message: '删除数据成功！',
                                            type: 'success'
                                            });
                            }else {
                                this.$notify.error({
                                    title: '错误',
                                      message: '删除数据失败！'+data.error.message,
                                    });
                             }
                            })
                        }else{
                         //前台删除
                            let index=this.formModel.items.indexOf(row);
                            this.formModel.items.splice(index,1);
                             this._sumCount();
                            this.$notify({
                                            title: '成功',
                                            message: '删除数据成功！',
                                            type: 'success'
                                            });
                        }
                       
                }).catch(()=>{});
            },
         _onSubmit() {
                let _this=this;
                this.$refs.formModel.validate((valid) => {
                    if (valid) {
                        this.formModel.items=this.$refs.tableedit.getEditData();
                        this.formModel.items.forEach((item,index,input)=>{
                            if(item.checkDate!=""){
                               input[index].checkDate=this.formatDate(item.checkDate,"YYYY-MM-DD");
                            }
                        });
                        if(this.formModel.items.length==0){
                              this.$message({ message: `请选择材料明细！`, type: 'warning'});
                              return;
                        }
                        this.formModel.order.orderDate=this.formatDate(this.formModel.order.orderDate,"YYYY-MM");
                        this.formModel.order.isAudit=false;
                        if(this.formModel.order.orderId!==""){
                            //修改
                             if(this.orderDateInital==this.formModel.order.orderDate){
                                 this.formModel.config.isGenerateCode=false;
                             }
                        }
                        this.loading=true;
                        requestCreateOrUpdateRequistionInNotQulifyOrder(this.formModel).then((data)=>{
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
                                 this.$message.error('保存数据失败！'+data.error.message);
                            }
                            this.loading=false;
                        }).catch(function(error){
                                _this.loading=false;
                        });
                    }else{
                         this.$message({ message: `请按照条件填写表单！`, type: 'warning'});
                    }
                });
            },
        selectNodeSupplier(node){
            this.formModel.order.supplierId=node.id;
            this.formModel.order.supplierName=node.supplierName;
            this.$refs.contractref.loadAll(node.id);
            this.contractDisable=false;

         },
        selectNodeContract(node){
            this.formModel.order.contractCode=node.contractCode;
            this.formModel.order.contractId=node.id;
        },
        _getCurrentNodeOrg(date){
            this.formModel.orgId=date.text;
            let val=this.formatDate(this.formModel.order.orderDate,"YYYY-MM");
            this._getCode(val);
        },
         _resetForm() {
                this.$refs.formModel.resetFields();
          },
          _siconClick(){
            this.contractDisable=true;
            this.formModel.order.supplierId='';
            this.formModel.order.contractCode='';
            this.formModel.order.contractId='';
        },
         _complete(){
                this.$emit('close');
           },
         _close(){
                //关闭选材框
                this.addFormVisible=false;
                this._sumCount();
          },
         _onSelect(){
             //打开选材框的权限判断
             this.tableEditMapConfig=this.MapConfig1;
             this.addFormVisible=true;
          },
        _getCode(val){
             this.generateOrderCode('M_RequistionInNotQulifyOrder',this.formModel.order.orgId,val,(val)=>{
                     this.formModel.order.orderCode=val;
             });
         },
        _dateSel(val){
           this._getCode(val);
        }
    },
    mounted(){
        if(this.selectRow.id!=undefined){
             //修改
             this._formInitial();
         } else{
             let val=this.formatDate(this.formModel.order.orderDate,"YYYY-MM");
             this._getCode(val);
         }
    },    
    components:{
       'yl-selectFromRequestionIn':selectFromRequestionIn,
       'yl-organizeTree':organizeTree,
       'yl-comsupplier':ComSupplier,
       'yl-Contract':Contract,

    },
    watch:{
        
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>
