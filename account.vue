<template>
  <div>
    <i-form :model="formData" ref="formData" :rules="formRules">
      <Row style="margin-top:20px;">
        <i-col span="24">
          <Form-item label="撬装点名称：" :label-width="150" prop="skidName">
            <i-input type="text" v-model="formData.skidName" style="width: 800px;"></i-input>
          </Form-item>
        </i-col>
        <i-col span="24">
          <Form-item label="营业执照编码：" :label-width="150" prop="cardId">
            <i-input type="text" v-model="formData.cardId"  :maxlength="18" style="width: 800px;"></i-input>
          </Form-item>
        </i-col>
        <i-col span="24">
          <Form-item  label="橇装归属人通行证号：" :label-width="150" prop="ownerId">
            <i-input type="text" v-model="formData.ownerId" :maxlength="10" @on-keyup="clearNoNum" style="width: 800px;"></i-input>
          </Form-item>
        </i-col>
        <i-col span="24">
          <Form-item  label="移动电话：" :label-width="150" prop="phoneNumber">
            <i-input type="text" v-model="formData.phoneNumber" :maxlength="11" style="width: 800px;"></i-input>
          </Form-item>
        </i-col>
        <i-col span="24">
          <Form-item  label="备用电话：" :label-width="150">
            <i-input type="text" style="width: 800px;" v-model="formData.phoneNumber2"></i-input>
          </Form-item>
        </i-col>
        <i-col span="24">
          <Form-item  label="地 址：" :label-width="150" prop="address">
            <i-input type="text" v-model="formData.address" style="width:400px;"></i-input>
            <Button @click="theLocation" style="margin-right:5px;">搜索</Button>
            <Button @click="theLocationClear" style="margin-right:5px;">清空</Button>
            经度：<i-input id="lat" type="text" v-model="formData.longitude" style="width:90px;"></i-input>
            &nbsp&nbsp纬度：<i-input id="lng" type="text" v-model="formData.latitude" style="width:90px;"></i-input>
            <p style="padding-left: 530px; color:red;">注：点击地图获取经纬度</p>
          </Form-item>
        </i-col>

        <i-col span="24">
          <Form-item  label="撬装点图片：" :label-width="150" prop="skidImg">
            <i-input type="text" v-model="formData.skidImg" style="width:50px;visibility:hidden;"></i-input>
            <div class="demo-upload-list" v-for="item in uploadList">
                  <template v-if="item.status === 'finished'">
                      <img :src="item.url">
                      <div class="demo-upload-list-cover">
                          <Icon type="ios-eye-outline" @click.native="handleView(item.name)"></Icon>
                          <Icon type="ios-trash-outline" @click.native="handleRemove(item)"></Icon>
                      </div>
                  </template>
                  <template v-else>
                      <Progress v-if="item.showProgress" :percent="item.percentage" hide-info></Progress>
                  </template>
              </div>
              <Upload ref="upload" :show-upload-list="false"
                  :action="uploadUrl"
                  :default-file-list="defaultList"
                  :on-success="handleSuccess"
                  :format="['jpg','jpeg','png']"
                  :max-size="2048"
                  :on-format-error="handleFormatError"
                  :on-exceeded-size="handleMaxSize"
                  :before-upload="handleBeforeUpload"
                  type="drag"
                  style="display: inline-block;width:58px;" :style="{display:uploadBtnFlag?'inline-block':'none'}">
                  <div style="width: 58px;height:58px;line-height: 58px;">
                      <Icon type="camera" size="20"></Icon>
                  </div>
              </Upload>
              <p style="color:red;margin-left:50px;">注：图片仅支持'jpg','jpeg','png'三种格式， 最大支持2MB文件</p>
          </Form-item>
        </i-col>

        <i-col span="24">
          <Form-item  label="地址图片库：" :label-width="150" prop="pics">
            <i-input type="text" style="width:50px;visibility:hidden;"></i-input>
            <div class="demo-upload-list" v-for="item in uploadList1">
              <template v-if="item.status === 'finished'">
                <img :src="item.url">
                <div class="demo-upload-list-cover">
                  <Icon type="ios-eye-outline" @click.native="handleView1(item.name)"></Icon>
                  <Icon type="ios-trash-outline" @click.native="handleRemove1(item)"></Icon>
                </div>
              </template>
              <template v-else>
                <Progress v-if="item.showProgress" :percent="item.percentage" hide-info></Progress>
              </template>
            </div>
            <Upload ref="upload1" :show-upload-list="false"
                    :action="uploadUrl1"
                    :default-file-list="defaultLists"
                    :on-success="handleSuccess1"
                    :format="['jpg','jpeg','png']"
                    :max-size="2048"
                    :on-format-error="handleFormatError1"
                    :on-exceeded-size="handleMaxSize1"
                    :before-upload="handleBeforeUpload1"
                    type="drag"
                    style="display: inline-block;width:58px;" :style="{display:uploadBtnFlag1?'inline-block':'none'}">
              <div style="width: 58px;height:58px;line-height: 58px;">
                <Icon type="camera" size="20"></Icon>
              </div>
            </Upload>
          </Form-item>
        </i-col>

        <i-col span="24">
          <Form-item  label="油站顾问：" :label-width="150">
            <i-input type="text" v-model="formData.agentId" style="width: 800px;"></i-input>
          </Form-item>
        </i-col>
        <i-col span="16" class="textAlignCenter">
              <Button type="primary" @click="register('formData')">立即注册</Button>
              <Button @click="reset('formData')">重置</Button>
        </i-col>
      </Row>
    </i-form>
    <Modal title="View Image" v-model="visible">
          <img :src="formData.skidImg" v-if="visible" style="width: 100%">
    </Modal>
    <Modal title="View Image" v-model="visible1">
      <img :src="formData.pics" v-if="visible1" style="width: 100%">
    </Modal>
    <div id="allmap"></div>
  </div>
</template>

<script>
import {sendRequest} from '../assets/js/comm';
import {oilUrl,uploadUrl} from '@/assets/config';
export default{
    methods:{
      theLocation() {             //地图搜索
        if (this.formData.address != "") {
          this.local.search(this.formData.address);
        }
      },
      theLocationClear(){    //地图条件置空
          this.formData.address = '';
          this.formData.longitude = '';
          this.formData.latitude = '';
      },
      clearNoNum(obj){
        obj.target.value = obj.target.value.replace(/[^\d.]/g,"");  //清除“数字”和“.”以外的字符
        obj.target.value = obj.target.value.replace(/\.{2,}/g,"."); //只保留第一个. 清除多余的
        obj.target.value = obj.target.value.replace(".","$#$").replace(/\./g,"").replace("$#$",".");
        obj.target.value = obj.target.value.replace(/^(\-)*(\d+)\.(\d\d).*$/,'$1$2.$3');//只能输入两个小数
        if(obj.target.value.indexOf(".")< 0 && obj.target.value !=""){//以上已经过滤，此处控制的是如果没有小数点，首位不能为类似于 01、02的金额
          obj.target.value= parseFloat(obj.target.value);
        }
      },
       handleView (name) {/*查看图片*/
                this.imgName = name;
                this.visible = true;
      },
      handleRemove (file) {/*移除图片*/
          const fileList = this.$refs.upload.fileList;
          this.$refs.upload.fileList.splice(fileList.indexOf(file), 1);
          this.formData.skidImg = '';
          this.uploadBtnFlag = true;
      },
      handleSuccess (res, file) {/*图片上传成功*/
          file.url = res.data;
          this.formData.skidImg = res.data;
          this.uploadBtnFlag = false;
      },
      handleFormatError (file) {
          this.$Notice.warning({
              title: '文件格式不正确',
              desc: '文件 ' + file.name + ' 不正确, 请选择 jpg 或 png.'
          });
      },
      handleMaxSize (file) {/*最大图片尺寸*/
          this.$Notice.warning({
              title: '超过文件最大限制',
              desc: '文件  ' + file.name + ' 过大, 不能超过 2M.'
          });
      },
      handleBeforeUpload () {
          const check = this.uploadList.length < 5;
          if (!check) {
              this.$Notice.warning({
                  title: 'Up to five pictures can be uploaded.'
              });
          }
          return check;
      },
      handleView1 (name) {/*查看图片*/
        this.imgName1 = name;
        this.visible1 = true;
      },
      handleRemove1 (file) {/*移除图片*/
        const fileList = this.$refs.upload1.fileList;
        this.$refs.upload1.fileList.splice(fileList.indexOf(file), 1);
        this.uploadBtnFlag1 = true;
        this.urlArr.splice(fileList.indexOf(file), 1);
      },
      handleSuccess1 (res, file) {/*图片上传成功*/
        file.url = res.data;
        this.urlArr.push(res.data);
      },
      handleFormatError1 (file) {
        this.$Notice.warning({
          title: '文件格式不正确',
          desc: '文件 ' + file.name + ' 不正确, 请选择 jpg 或 png.'
        });
      },
      handleMaxSize1 (file) {/*最大图片尺寸*/
        this.$Notice.warning({
          title: '超过文件最大限制',
          desc: '文件  ' + file.name + ' 过大, 不能超过 2M.'
        });
      },
      handleBeforeUpload1 () {
        const check = this.uploadList1.length < 5;
        if (!check) {
          this.$Notice.warning({
            title: 'Up to five pictures can be uploaded.'
          });
        }
        return check;
      },
      register(formRef){/*注册事件*/
        this.$refs[formRef].validate((valid)=>{
          let $this = this;
          if(valid){
            sendRequest(oilUrl + "/skid/register", {
                      skidName: $this.formData.skidName,
                      cardId: $this.formData.cardId,
                      ownerId: $this.formData.ownerId,
                      phoneNumber: $this.formData.phoneNumber,
                      phoneNumber2: $this.formData.phoneNumber2,
                      address: $this.formData.address,
                      longtitude:$this.formData.longitude,
                      latitude:$this.formData.latitude,
                      skidImg: $this.formData.skidImg,
                      pics: JSON.stringify($this.urlArr),
                      agentId: $this.formData.agentId
                  }, function (data) {
                      if (data) {
                        $this.$Modal.info({
                          title:'提示',
                          content:"注册成功,会员号：" + data.memberId
                        });
                        $this.$refs[formRef].resetFields();
                      }
                      $this.formData.agentId = '';
                      $this.theLocationClear();
                  }, function (info) {
                      console.log(info)
                  },this);
          }else{
          	this.$Message.error("请根据提示输入正确信息");
          }
        });
      },
      reset(formRef){
        this.$refs[formRef].resetFields();
      }
    },
    data(){
      return{
        local:'',
        uploadBtnFlag:true,
        uploadBtnFlag1:true,
        uploadUrl:uploadUrl+'/res/files/upload',
        uploadUrl1:uploadUrl+'/res/files/upload',
        defaultList: [],
        defaultLists: [],
        imgName: '',
        imgName1: '',
        visible: false,
        visible1: false,
        uploadList: [],
        uploadList1: [],
        urlArr:[],
        formData:{
          longitude:'', /*经度*/
          latitude:'', /*纬度*/
          skidName:'',/*橇装点名称*/
          cardId:'',/*营业执照编码*/
          ownerId:'',/*橇装归属人通行证号*/
          phoneNumber:'',/*移动电话*/
          phoneNumber2:'',/*备用电话*/
          address:'',/*地 址*/
          skidImg:'',/*撬装点图片*/
          agentId:''/*油站顾问*/
        },
         formRules:{/*表单验证规则*/
          skidName:[
            {required:true,message:'请输入橇装点名称',trigger:'blur'}
          ],
          cardId:[
            {required:true,message:'请输入营业执照编码',trigger:'blur'}
          ],
          ownerId:[
            {required:true,message:'请输入橇装归属人通行证号',trigger:'blur'}
          ],
          phoneNumber:[
            {required:true,message:'请输入移动电话',trigger:'blur'}
          ],
          address:[
            {required:true,message:'请输入地址',trigger:'blur'}
          ],
          skidImg:[
            {required:true,message:'请上传撬装点图片',trigger:'blur'}
          ]
        }
      }
    },
    mounted () {
      let that = this;
      this.uploadList = this.$refs.upload.fileList;
      this.uploadList1 = this.$refs.upload1.fileList;

      var map = new BMap.Map("allmap", {
        minZoom: 5,
        maxZoom: 19
      });

      // 初始化地图，设置中心点和显示级别
      map.centerAndZoom(new BMap.Point(121.36564, 31.22611), 19);

      // 开启鼠标滚轮缩放功能，仅对PC上有效
      map.enableScrollWheelZoom(true);

      // 将控件（平移缩放控件）添加到地图上
      map.addControl(new BMap.NavigationControl());

      var marker = null;
      // 为地图增加点击事件，为input赋值
      map.addEventListener("click", function(e) {
        that.formData.longitude = e.point.lat;
        that.formData.latitude = e.point.lng;
        map.removeOverlay(marker);
        var point = e.point;
        marker = new BMap.Marker(point);  // 创建标注
        map.addOverlay(marker);
        var getData = new BMap.Geocoder();
        getData.getLocation(point, function(rs){
          if(rs.surroundingPois.length>=1){
            that.formData.address = rs.address + rs.surroundingPois[0].title;
          }else{
            that.formData.address = rs.address;
          }
        });
      });

      // 创建位置检索、周边检索和范围检索
      this.local = new BMap.LocalSearch(map, {
        renderOptions: {
          map: map
        }
      });
    }
  }
</script>

<style scoped>
  body,
  html {
    width: 100%;
    height: 100%;
    margin: 0;
    font-family: "微软雅黑";
    /*overflow: hidden;*/
  }

  #allmap {
    margin-top: 50px;
    height:630px;
    width:400px;
    overflow: hidden;
    position: fixed;
    top:-735px;
    left:960px;
  }
.textAlignCenter{
  text-align: center;
}

/*=================================*/
.demo-upload-list{
        display: inline-block;
        width: 60px;
        height: 60px;
        text-align: center;
        line-height: 60px;
        border: 1px solid transparent;
        border-radius: 4px;
        overflow: hidden;
        background: #fff;
        position: relative;
        box-shadow: 0 1px 1px rgba(0,0,0,.2);
        margin-right: 4px;
    }
    .demo-upload-list img{
        width: 100%;
        height: 100%;
    }
    .demo-upload-list-cover{
        display: none;
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        background: rgba(0,0,0,.6);
    }
     .demo-upload-list:hover .demo-upload-list-cover{
        display: block;
    }
    .demo-upload-list-cover i{
        color: #fff;
        font-size: 20px;
        cursor: pointer;
        margin: 0 2px;
    }
</style>
