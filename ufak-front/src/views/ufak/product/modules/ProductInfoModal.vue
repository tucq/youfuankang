<template>
  <a-modal
    :title="title"
    :width="900"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="商品名称">
              <a-input placeholder="请输入商品名称" v-decorator="['name', {}]" maxlength="200" />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="商品描述">
              <a-input placeholder="请输入商品描述" v-decorator="['describe', {}]" maxlength="200" />
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="商品分类">
              <a-input placeholder="请输入商品分类" v-decorator="['productType', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="月销量">
              <a-input v-decorator="[ 'salesVolume', {}]" />
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="服务">
              <a-input placeholder="请输入服务" v-decorator="['service', {}]" />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="上下架">
              <a-input placeholder="请选择上下架" v-decorator="['state', {}]" />
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20" style="margin-left: 10px;margin-right: 10px">
          <a-tabs defaultActiveKey="1">
            <a-tab-pane key="1">
              <span slot="tab">
                <a-icon type="table" />商品图片
              </span>
                <a-upload
                  :action="uploadAction"
                  listType="picture-card"
                  :fileList="fileList"
                  :headers="headers"
                  :beforeUpload="beforeUpload"
                  @preview="handlePreview"
                  @change="imageChange"
                >
                  <div v-if="fileList.length < 300">
                    <a-icon type="plus" />
                    <div class="ant-upload-text">Upload</div>
                  </div>
                </a-upload>
                <a-modal :visible="previewVisible" :footer="null" @cancel="imageCancel">
                  <img alt="example" style="width: 100%" :src="previewImage" />
                </a-modal>
            </a-tab-pane>
            <a-tab-pane key="2">
              <span slot="tab">
                <a-icon type="table" />商品规格
              </span>
            </a-tab-pane>
            <a-tab-pane key="3">
              <span slot="tab">
                <a-icon type="table" />图文详情
              </span>
            </a-tab-pane>
          </a-tabs>
        </a-row>


      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction,postAction,getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"
  import Vue from 'vue'
  import { ACCESS_TOKEN } from "@/store/mutation-types"

  export default {
    name: "ProductInfoModal",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        },
        headers:{},
        picUrl: "",
        uploadLoading:false,
        previewVisible: false,
        previewImage: '',
        // fileList: [
        //     {
        //         uid: '-1',
        //         name: 'xxx.png',
        //         status: 'done',
        //         url: 'https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png',
        //         thumbUrl:
        //             'https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png',
        //     },
        //     {
        //         uid: '-3',
        //         name: 'yyy.png',
        //         status: 'done',
        //         url: 'https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png',
        //         thumbUrl:
        //             'https://zos.alipayobjects.com/rmsportal/jkjgkEfvpUPVyRjUImniVslZfWPnJuuZ.png',
        //     },
        // ],
        fileList: [],
        url: {
          add: "/productInfo/add",
          edit: "/productInfo/edit",
          fileUpload: window._CONFIG['domianURL']+"/sys/common/upload",
          imgerver: window._CONFIG['domianURL']+"/sys/common/view",
        },
      }
    },
    created () {
        const token = Vue.ls.get(ACCESS_TOKEN);
        this.headers = {"X-Access-Token":token}
    },
    computed:{
        uploadAction:function () {
            return this.url.fileUpload;
        }
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'name','describe','image','price','virtualPrice','discount','productType','specs','salesVolume','service','detailDescribe','detailImages','state'))
		  //时间格式化
        });

      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {

          let image = "";
          for(let i=0;i<this.fileList.length;i++){
              image += this.fileList[i].response.message + ","
          }


        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            
            console.log(formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })



          }
        })
      },
      handleCancel () {
        this.close()
      },
      imageCancel() {
          this.previewVisible = false;
      },
      handlePreview(file) {
          this.previewImage = file.url || file.thumbUrl;
          this.previewVisible = true;
      },
      imageChange({ fileList }) {
          this.fileList = fileList;
      },
      beforeUpload: function(file){
          var fileType = file.type;
          if(fileType.indexOf('image')<0){
              this.$message.warning('请上传图片');
              return false;
          }
          //TODO 验证文件大小
      },
    }
  }
</script>

<style lang="less" scoped>
   /*you can make up upload button and sample style by using stylesheets*/
  .ant-upload-select-picture-card i {
    font-size: 16px;
    color: #999;
  }

  .ant-upload-select-picture-card .ant-upload-text {
    margin-top: 8px;
    color: #666;
  }
</style>