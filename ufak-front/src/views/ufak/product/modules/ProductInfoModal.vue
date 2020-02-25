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
              <a-input placeholder="请输入商品名称" v-decorator="['name', {}]" maxlength="200"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="商品描述">
              <a-input placeholder="请输入商品描述" v-decorator="['description', {}]" maxlength="200"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="商品分类">
              <a-input placeholder="请输入商品分类" v-decorator="['productType', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="月销量">
              <a-input v-decorator="[ 'salesVolume', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="服务">
              <a-input placeholder="请输入服务" v-decorator="['service', {}]"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item
              :labelCol="labelCol"
              :wrapperCol="wrapperCol"
              label="上下架">
              <a-input placeholder="请选择上下架" v-decorator="['state', {}]"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row :gutter="20" style="margin-left: 10px;margin-right: 10px">
          <a-tabs defaultActiveKey="1">
            <a-tab-pane key="1">
              <span slot="tab">
                <a-icon type="table"/>商品图片
              </span>
              <a-upload
                :action="uploadAction"
                listType="picture-card"
                :fileList="fileList"
                :headers="headers"
                :beforeUpload="beforeUpload"
                :remove="imageRemove"
                @preview="handlePreview"
                @change="imageChange"
              >
                <div v-if="fileList.length < 300">
                  <a-icon type="plus"/>
                  <div class="ant-upload-text">Upload</div>
                </div>
              </a-upload>
              <a-modal :visible="previewVisible" :footer="null" @cancel="imageCancel">
                <img alt="example" style="width: 100%" :src="previewImage"/>
              </a-modal>
            </a-tab-pane>
            <a-tab-pane key="2">
              <span slot="tab">
                <a-icon type="table"/>商品规格
              </span>
            </a-tab-pane>
            <a-tab-pane key="3">
              <span slot="tab">
                <a-icon type="table"/>图文详情
              </span>

            </a-tab-pane>
          </a-tabs>
        </a-row>


      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
    import {httpAction, postAction} from '@/api/manage'
    import pick from 'lodash.pick'
    import moment from "moment"
    import Vue from 'vue'
    import {ACCESS_TOKEN} from "@/store/mutation-types"

    export default {
        name: "ProductInfoModal",
        data() {
            return {
                title: "操作",
                visible: false,
                model: {},
                labelCol: {
                    xs: {span: 24},
                    sm: {span: 5},
                },
                wrapperCol: {
                    xs: {span: 24},
                    sm: {span: 16},
                },

                confirmLoading: false,
                form: this.$form.createForm(this),
                validatorRules: {},
                headers: {},
                uploadLoading: false,
                previewVisible: false,
                previewImage: '',
                fileList: [],
                removeFileList: [],
                url: {
                    add: "/productInfo/add",
                    edit: "/productInfo/edit",
                    fileUpload: window._CONFIG['domianURL'] + "/sys/common/upload",
                    imgerver: window._CONFIG['domianURL'] + "/sys/common/view",
                    removeFile: window._CONFIG['domianURL'] + "/sys/common/remove",
                },
            }
        },
        created() {
            const token = Vue.ls.get(ACCESS_TOKEN);
            this.headers = {"X-Access-Token": token}
        },
        computed: {
            uploadAction: function () {
                return this.url.fileUpload;
            }
        },
        methods: {
            add() {
                this.edit({});
            },
            edit(record) {
                this.form.resetFields();
                this.model = Object.assign({}, record);
                this.visible = true;
                this.$nextTick(() => {
                    this.form.setFieldsValue(pick(this.model, 'name', 'description', 'productType', 'salesVolume', 'service', 'state'))
                    //时间格式化
                });

                /** 图片渲染 **/
                if(this.model.image){
                    let tmpUrls = this.model.image.substring(0,this.model.image.length-1).split(',');
                    let tmpIdx = 0;
                    let baseUrl = window._CONFIG['domianURL'] + "/sys/common/view/";
                    for(let i=0;i<tmpUrls.length;i++){
                        let imgUrl = tmpUrls[i];
                        tmpIdx--;
                        this.fileList[i] = {
                            uid: tmpIdx, //根据官方API文档，该值最好给个负数值，以免与内部对象冲突
                            name: imgUrl.substring(imgUrl.lastIndexOf("/"),imgUrl.indexOf(".")),
                            status: 'done',
                            url: baseUrl + imgUrl,
                            thumbUrl: baseUrl + imgUrl,
                            isCommit: true, //图片是否已提交
                        }
                    }
                }else{
                    this.fileList = [];
                }

                console.log("图片渲染fileList:",this.fileList);
            },
            close() {
                this.$emit('close');
                this.visible = false;
            },
            handleOk() {
                const that = this;
                if (this.fileList.length == 0) {
                    that.$message.warning("请上传商品图片！");
                    return;
                }
                console.log("this.fileList=", this.fileList);
                let imageUrls = "";
                for (let i = 0; i < this.fileList.length; i++) {
                    let img = this.fileList[i];
                    if (img.response) {
                        imageUrls += img.response.message + ","
                    } else {
                        imageUrls += img.url.substring(img.url.indexOf("/files"), img.url.length) + ","
                    }
                }
                console.log("商品图片路径：" + imageUrls);

                // 触发表单验证
                this.form.validateFields((err, values) => {
                    if (!err) {
                        that.confirmLoading = true;
                        let httpurl = '';
                        let method = '';
                        if (!this.model.id) {
                            httpurl += this.url.add;
                            method = 'post';
                        } else {
                            httpurl += this.url.edit;
                            method = 'put';
                        }
                        let formData = Object.assign(this.model, values);
                        //时间格式化
                        formData.image = imageUrls;

                        console.log(formData)
                        httpAction(httpurl, formData, method).then((res) => {
                            if (res.success) {
                                // 提交成功后服务器删除移除的图片
                                this.handleRemoveFile(this.removeFileList);

                                that.$message.success(res.message);
                                that.$emit('ok');
                            } else {
                                that.$message.warning(res.message);
                            }
                        }).finally(() => {
                            that.confirmLoading = false;
                            that.close();
                        })
                    }
                })
            },
            handleCancel() {
                let files = [];
                for (let i = 0; i < this.fileList.length; i++) {
                    let img = this.fileList[i];
                    if (!img.isCommit) {
                        files.push(img.response.message);
                    }
                }
                console.log("未提交的图片：",files);
                // 关闭成功后服务器删除未提交的图片
                this.handleRemoveFile(files);

                this.close()
            },
            imageCancel() {
                this.previewVisible = false;
            },
            handlePreview(file) {
                this.previewImage = file.url || file.thumbUrl;
                this.previewVisible = true;
            },
            imageChange(info) {
                this.fileList = info.fileList;
                console.log(this.fileList);
            },
            imageRemove(file) {
                if(file.url){
                    let rmUrl = file.url.substring(file.url.indexOf("/files"), file.url.length);
                    this.removeFileList.push(rmUrl);
                }else{
                    this.removeFileList.push(file.response.message);
                }

                console.log("删除图片路径：", this.removeFileList);
            },
            beforeUpload: function (file) {
                const isJPG = file.type === 'image/jpeg';
                if (!isJPG) {
                    this.$message.error('请上传jpg格式图片!');
                }
                const isLt2M = file.size / 1024 / 1024 < 2;
                if (!isLt2M) {
                    this.$message.error('上传照片不要超过2MB!');
                }
                return isJPG && isLt2M;
            },
            handleRemoveFile(files){
                postAction(this.url.removeFile, {filePaths:files}).then((res) => {

                });
            }

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