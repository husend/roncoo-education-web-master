<template>
  <div class="">
    <div class="mask"></div>

    <div class="paymodal codemodal" v-if="payStep === 1">
      <div class="modal_head">
        <span class="fl">收银台</span>
        <a href="javascript:" @click="close()" class="close iconfont">&#xe616;</a>
      </div>
      <div class="modal_body">
        <div class="tip" v-if="orderInfo.payType === 2">支付宝支付 {{courseData.courseOriginal.toFixed(2)}}元</div>
        <div class="tip" v-else>微信支付 {{courseData.courseOriginal.toFixed(2)}}元</div>
        <div class="code" id="qrcode">
          <canvas id="canvas"></canvas>
        </div>
        <p class="tip2" v-if="orderInfo.payType === 2">请使用支付宝扫描<br> 二维码以完成订单</p>
        <p class="tip2" v-else>请使用微信扫描<br> 二维码以完成订单</p>
        <div class="tip_panel">
          提示: <br>
          支付成功前请勿手动关闭页面 <br>
          二维码30分钟内有效,请及时扫码支付
        </div>
      </div>
    </div>

    <div class="paymodal" v-else-if="payStep === 0">
      <div class="modal_head">
        <span class="fl">收银台</span>
        <a href="javascript:" @click="close()" class="close iconfont">&#xe616;</a>
      </div>
      <div class="modal_body" v-if="courseData">

        <table class="table">
          <tr>
            <th>课程名称</th>
            <th>课程价格</th>
          </tr>
          <tr>
            <td width="300">
              <div class="" style="width:320px;">
                <div class="img" v-if="courseData.courseLogo"><img :src="courseData.courseLogo" alt=""></div>
                <div class="name">{{courseData.courseName}}</div>
              </div>
            </td>
            <td class="c_orange">¥{{courseData.courseOriginal.toFixed(2)}}</td>
          </tr>
        </table>
        <div class="remark">
          <label>备注:</label>
          <input type="text" v-model="order.remarkCus" placeholder="请输入你需要的备注!">
        </div>
        <div class="pay_type">
          <input type="radio" id="payType2" name="payType" value="2" v-model="order.payType">
          <label for="payType2" class="pay pay_ali"></label>
          <input type="radio" id="payType1" name="payType" value="1" v-model="order.payType">
          <label for="payType1" class="pay pay_weixin"></label>
        </div>
      </div>
      <div class="modal_foot">
        <button class="solid_btn" @click="submit" :disabled="btntext === '正在提交...'">{{btntext}}</button>
      </div>
    </div>

    <div class="paymodal payrun" v-else-if="payStep === 2">
      <div class="modal_head">
        <span class="fl">收银台</span>
<!--        <a href="javascript:" @click="close()" class="close iconfont">&#xe616;</a>-->
        <span href="javascript:" class="close iconfont">&nbsp;</span>
      </div>
      <div class="modal_body">
        <div class="icon iconfont c_green minIcon">&#xe69f;<span class="c_333 font_14">订单支付成功!</span></div>
        <div class="address-box" @click="intoAddress">
          <el-card shadow="hover">
            <div class="car-text car-item">
              <div v-model="showAddress">
                <p style="margin-bottom: 12px">
                  <span>{{showAddress.receiverProvince}}</span>
                  <span>{{showAddress.receiverCity}}</span>
                  <span>{{showAddress.receiverDistrict}}</span>
                </p>
                <p style="margin-bottom: 12px">{{showAddress.receiverAddress}}</p>
                <p>
                  <span>{{showAddress.receiverName}}</span>&nbsp;
                  <span>{{showAddress.receiverPhone}}</span>
                </p>
              </div>
              <div class="address-into">
                更多地址&nbsp;>>>
              </div>
            </div>
<!--            <div v-else class="car-text car-item" @click="intoAddress">-->
<!--              <div>-->
<!--                <p>请选择收货地址！</p>-->
<!--              </div>-->
<!--              <div style="float: right;margin-top: -18px">-->
<!--                更多地址&nbsp;>>>-->
<!--              </div>-->
<!--            </div>-->
          </el-card>
        </div>
        <div class="center border_b">
          <a class="solid_btn order_btn" v-if="courseData.courseCategory == 1" :href="'/view/'+courseData.id">立即学习</a>
          <a class="solid_btn order_btn" v-else-if="courseData.courseCategory == 2" :href="'/live/detail/'+courseData.id">立即学习</a>
          <a class="solid_btn order_btn" v-else-if="courseData.courseCategory == 3" :href="'/live/bunch/'+courseData.id">立即学习</a>
          <a class="solid_btn order_btn" v-else href="javascript:" @click="reload">确定</a>
<!--          <a class="solid_btn order_btn" target="_blank" href="/account/order">查看订单</a>-->
        </div>
        <div class="tip_panel">
          提示: <br>
          支付成功后请确认收货地址！
        </div>
<!--        <div class="ewm_img" v-if="webInfo && webInfo.weixinXcx">-->
<!--          <img :src="webInfo.weixinXcx" alt="">-->
<!--          <p class="mgt10">微信扫码关注小程序</p>-->
<!--          <p>学习更便捷</p>-->
<!--        </div>-->
      </div>
    </div>

    <div class="paymodal payrun" v-else-if="payStep === 3">
      <div class="modal_head">
        <span class="fl">收银台</span>
        <a href="javascript:" @click="close()" class="close iconfont">&#xe616;</a>
      </div>
      <div class="modal_body">
        <div class="icon iconfont c_red">&#xe68c;</div>
        <div class="tip">支付失败</div>
        <div class="center">
          <button class="solid_btn" @click="payStep = 0">重新购买</button>
        </div>
      </div>
    </div>

    <div class="paymodal payrun" v-else-if="payStep === 4">
      <div class="modal_head">
        <span class="fl">收货地址</span>
        <a href="javascript:;" @click="addAddress">新增地址</a>
      </div>
      <div class="address_card_body"  v-infinite-scroll="load" infinite-scroll-disabled="disabled" style="overflow:auto">
        <div v-for="(item, index) in pageList" :key="index" @click="changeAddress(index, item)">
          <el-card class="box-card" shadow="hover">
            <div class="car-text car-item">
              <el-radio v-model="radio" :label="index" class="box-radio">&nbsp;</el-radio>
              <div class="address-info">
                <p>{{item.receiverProvince.trim()}}{{item.receiverCity.trim()}}{{item.receiverDistrict.trim()}}</p>
                <p>{{item.receiverAddress.trim()}}</p>
                <p>{{item.receiverName.trim()}}{{item.receiverPhone.trim()}}</p>
              </div>
              <div class="address-edit">
                <a href="javascript:;" @click="editAddress(item)">修改地址</a>
              </div>
            </div>
          </el-card>
        </div>
        <p v-if="loading" style="margin-top:10px;text-align: center" class="loading"><span></span></p>
        <p v-if="noMore" style="margin-top:10px;font-size:13px;color:#ccc;text-align: center">没有更多了</p>
      </div>
      <button class="address_btn" style="margin-left: 190px" @click="selAddress">确定</button>
    </div>

    <div class="paymodal payrun" style="width: 650px" v-else-if="payStep === 5">
      <div class="modal_head">
        <span class="fl">{{operateContent}}地址</span>
        <a href="javascript:;">&nbsp;</a>
      </div>
      <div>
        <form action="" @submit="regSubmit">
          <div class="form_group">
            <div class="label">收货人:</div>
            <div class="form_ctl">
              <input type="text" v-model="pobj.receiverName" class="form_input" placeholder="请输入姓名">
            </div>
          </div>
          <div class="form_group">
            <div class="label">手机号码:</div>
            <div class="form_ctl">
              <input type="text" v-model="pobj.receiverPhone" maxlength="11" class="form_input" placeholder="请输入手机号码">
            </div>
          </div>
          <div class="form_group">
            <div class="label">地区信息:</div>
            <div class="form_ctl">
              <v-distpicker @selected="onSelected" :province="pobj.receiverProvince" :city="pobj.receiverCity" :area="pobj.receiverDistrict"></v-distpicker>
            </div>
          </div>
          <div class="form_group">
            <div class="label">详细地址:</div>
            <div class="form_ctl">
              <input type="text" v-model="pobj.receiverAddress" class="form_input" placeholder="请输入详细地址">
            </div>
          </div>
          <div class="form_group">
            <div class="label">设为默认地址:</div>
            <div class="form_ctl">
              <el-switch v-model="pobj.isToleration" class="form_switch" :active-value="1" :inactive-value="0" active-color="#D51423" inactive-color="#999"></el-switch>
            </div>
          </div>
          <div class="form_group">
            <div class="label">&nbsp;</div>
            <div class="form_ctl" style="text-align: center">
              <button class="address_btn"  v-if="submitBtn">确认</button>
              <button class="address_btn" v-else disabled="disabled">确认</button>
              <button class="address_btn" @click="payStep = 4">取消</button>
            </div>
          </div>
        </form>
      </div>
<!--      <div style="text-align: center">-->
<!--        <button class="address_btn" @click="">确定</button>-->
<!--        <button class="address_btn" @click="payStep = 4">取消</button>-->
<!--      </div>-->
    </div>
  </div>
</template>
<script>
  import VDistpicker from '~/components/Distpicker'
  import {orderSave, orderInfo} from '~/api/order.js'
  import {getAddressList, saveAddress, updateAddress} from '~/api/account/address'
  import {completeOrderSite} from '~/api/account/course'
  import {myHttp} from '~/utils/myhttp.js'
  import QRCode from 'qrcode'
  export default {
    components: {
        VDistpicker
    },
    props: {
      data: {
        type: [Object, Array],
        default: null
      },
      no: {
        type: String,
        default: ''
      },
      isCourse: {
        type: Boolean
      }
    },
    data () {
      return {
        isVip: false,
        btntext: '下一步',
        checkPay: false,
        userInfo: this.$store.state.userInfo,
        courseData: '',
        order: {
          channelType: 1,
          courseId: 0,
          payType: 1,
          tradeType: 'NATIVE',
          remarkCus: ''
        },
        orderInfo: null,
        payStep: 0,
        player: null,
        radio: 0, // 地址单选按钮值
        count: 1, // 滚动加载页数
        loading: false,
        pageList: [], // 地址列表
        pageObj: { // 查询地址列表参数
            pageCurrent: '',
            pageSize: '',
            totalCount: '',
            totalPage: ''
        },
        showAddress: { // 第四步显示使用
            id: '',
            isToleration: 0,
            receiverAddress: '',
            receiverCity: '',
            receiverDistrict: '',
            receiverName: '',
            receiverPhone: '',
            receiverProvince: ''
        },
        // isDefaultAddress: false, // 是否有默认地址
        operateContent: '',
        pobj: { // 新增/修改 地址用
            id: '',
            receiverName: '',
            receiverPhone: '',
            receiverProvince: '',
            receiverCity: '',
            receiverDistrict: '',
            receiverAddress: '',
            receiverZip: '',
            isToleration: 0
        },
        submitBtn: true
      }
    },
    computed: {
          noMore () {
              return this.count > this.pageObj.totalPage - 1;
          },
          disabled () {
              return this.loading || this.noMore
          }
      },
    mounted () {
      this.getAddressList()
      // this.qrcode('weixin://wxpay/bizpayurl?pr=GlkCzqF');
      this.order.courseId = this.data.course.id;
      this.courseData = this.data.course;
      if (this.userInfo.isVip === 1) {
          this.isVip = true
      } else {
          this.isVip = false
      }
    },
    methods: {
      load () {
          this.loading = true
          setTimeout(() => {
              this.count += 1;
              this.getAddressList()
          }, 1000)
      },
      getAddressList () {
          myHttp.call(this, {
              method: getAddressList,
              params: {
                  pageCurrent: this.count,
                  pageSize: 5
              }
          }).then(res => {
              this.pageObj = res.data
              this.pageList = this.pageList.concat(res.data.list)
              this.loading = false
          })
      },
      selAddress () {
          this.payStep = 2
          if(this.showAddress.receiverName === ''){
              this.showAddress = this.pageList[0]
          }
          // this.isDefaultAddress = true
      },
      intoAddress () {
          this.payStep = 4
      },
      changeAddress (index, item) {
          this.radio = index
          this.showAddress = item
          // this.isDefaultAddress = true
      },
      addAddress () {
          this.operateContent = '新增'
          this.payStep = 5
      },
      editAddress (item) {
          this.operateContent = '修改'
          this.pobj = item
          this.payStep = 5
      },
      saveAddress () {
          saveAddress(this.pobj).then(res => {
              let result = res.data
              if (result.code === 200) {
                  this.$msgBox({
                      content: '新增成功',
                      isShowCancelBtn: false
                  }).then(async (val) => {
                      this.payStep = 4
                  }).catch(() => {
                      this.payStep = 4
                  })
              } else {
                  if (result.code >= 300 && result.code < 400) {
                      this.$msgBox({
                          content: '登录异常，请重新登录',
                          isShowCancelBtn: false
                      }).then(() => {
                          this.$store.dispatch('REDIRECT_LOGIN', result.code)
                      }).catch(() => {
                          this.$store.dispatch('REDIRECT_LOGIN', result.code)
                      })
                  } else {
                      this.$msgBox({
                          content: result.msg,
                          isShowCancelBtn: false
                      }).catch(() => {})
                  }
              }
          }).catch(msg => {
              this.$msgBox({
                  content: '新增地址失败！！',
                  isShowCancelBtn: false
              })
          })
      },
      updateAddress () {
            updateAddress(this.pobj).then(res => {
                let result = res.data
                if (result.code === 200) {
                    this.$msgBox({
                        content: '修改成功',
                        isShowCancelBtn: false
                    }).then(async (val) => {
                        this.payStep = 4
                    }).catch(() => {
                        this.payStep = 4
                    })
                } else {
                    if (result.code >= 300 && result.code < 400) {
                        this.$msgBox({
                            content: '登录异常，请重新登录',
                            isShowCancelBtn: false
                        }).then(() => {
                            this.$store.dispatch('REDIRECT_LOGIN', result.code)
                        }).catch(() => {
                            this.$store.dispatch('REDIRECT_LOGIN', result.code)
                        })
                    } else {
                        this.$msgBox({
                            content: result.msg,
                            isShowCancelBtn: false
                        }).catch(() => {})
                    }
                }
            }).catch(msg => {
                this.$msgBox({
                    content: '修改地址失败！！',
                    isShowCancelBtn: false
                })
            })
        },
      regSubmit: function (e) {
          e.preventDefault();
          if (!this.pobj.receiverName) {
              this.showMsg('请填写收货人')
              return false;
          }
          if (!this.pobj.receiverPhone) {
              this.showMsg('请填写手机号码')
              return false;
          }
          if (!(/^1[3|4|5|8|7|9][0-9]\d{4,8}$/.test(this.pobj.receiverPhone.trim()))) {
              this.showMsg('请输入正确手机号');
              return false;
          }
          if (this.pobj.receiverDistrict === '区' || this.pobj.receiverDistrict === '') {
              this.showMsg('请选择地区信息')
              return false;
          }
          if (!this.pobj.receiverAddress) {
              this.showMsg('请填写详细地址')
              return false;
          }
          if(this.operateContent === '修改'){
              this.submitBtn = false
              this.updateAddress()
              this.pobj = {}
              this.pageList.length = 0
              this.load ()
              this.submitBtn = true
          }else{
              this.submitBtn = false
              this.saveAddress()
              this.pobj = {}
              this.pageList.length = 0
              this.load ()
              this.submitBtn = true
          }

      },
      showMsg (msg) {
          this.$msgBox({
              content: msg,
              isShowCancelBtn: false
          }).catch(() => {})
      },
      onSelected(data) {
          this.pobj.receiverProvince = data.province.value
          this.pobj.receiverCity = data.city.value
          this.pobj.receiverDistrict = data.area.value
      },
      reload () {
        if(this.showAddress.id === ''){
            this.showMsg("请选择地址！")
            return false
        }
        completeOrderSite({orderNo: this.orderInfo.orderNo, shippingAddressId: this.showAddress.id}).then(res => {
            res = res.data;
            if(res.data > 0){
              window.location.reload();
            }
        }).catch(error => {
            this.$msgBox({
                content: error.data.data,
                isShowCancelBtn: false
            })
        })
      },
      close (event) {
        this.checkPay = true;
        this.$emit('hidefun', event);
      },
      sel (cno) {
        this.checkPay = true;
        this.$emit('btnClick', cno)
      },
      qrcode (url) {
        QRCode.toCanvas(document.getElementById('canvas'), url, {
          width: 180,
          height: 180
        }, function (error) {
          if (error) console.error(error)
          console.log('success!')
        })
      },
      startView () {

      },
      submit () {
        let that = this;
        that.btntext = '正在提交...';
        console.log(that.order)
        //orderSave
        orderSave(that.order).then(res => {
          res = res.data;
          console.log(res)
          that.btntext = '下一步';
          if (res.code === 200) {
            that.payStep = 1;
            that.orderInfo = res.data;
            this.ocl = setTimeout(function () {
              that.qrcode(res.data.payMessage);
            }, 100);
            that.getOrderInfo(res.data.orderNo, res.data.serialNumber, that.order.payType, this.userInfo.userNo)
          } else if (res.code >= 300 && res.code < 400) {
            this.$msgBox({
              content: res.msg,
              isShowCancelBtn: false
            }).then(() => {
              this.$store.dispatch('REDIRECT_LOGIN');
            }).catch(() => {})
          } else {
            this.$msgBox({
              content: res.msg,
              isShowCancelBtn: false
            }).catch(() => {})
          }
        }).catch(() => {
          this.$msgBox({
            content: '提交失败,请重试',
            isShowCancelBtn: false
          })
          that.btntext = '重新提交';
        })
      },
      getOrderInfo (no, serial, payType, userNo) {
        let that = this;
        if (this.checkPay) {
          return false;
        }
        orderInfo({orderNo: no, serialNumber: serial, payType: payType, userNo: userNo}).then(res => {
          res = res.data
          console.log(res)
          if (res.data.orderStatus === 1) {
            setTimeout(function () {
              that.getOrderInfo(no, serial,payType);
            }, 1000);
          } else if (res.data.orderStatus === 2) {
            if (that.order.courseType === 3) {
              this.close();
              this.$msgBox({
                content: '购买成功',
                isShowCancelBtn: false
              }).then(res => {
                window.location.reload();
              }).catch(() => {
                window.location.reload();
              })
            } else {
              that.payStep = 2;
              this.showAddress = this.pageList[0]
              // if (this.pageList[0].isToleration === 1) {
              //     this.isDefaultAddress = true
              // }
            }
          } else {
            that.payStep = 3;
          }
        })
      }
    }
  }
</script>
<style lang="scss" rel="stylesheet/scss" scoped>
.payrun{
  min-height: 180px;
  .modal_body{
    padding: 30px;
    // height: 180px;
  }
  .address_card_body {
    height: 370px;
    padding: 10px 0 10px 7px;
  }
  .ewm_img {
    width: 130px;
    display: block;
    margin: 10px auto 0;
    text-align: center;
    img {
      width: 130px;
      height: 130px;
    }
    p {
      line-height: 20px;
    }
  }
  .icon{
    font-size: 80px;
    width: 80px;
    margin: 0 auto;
    text-align: center;
    &.minIcon {
      font-size: 26px;
      width: 160px;
      span {
        position: relative;
        top: -5px;
        padding-left: 6px;
      }
    }
  }
  .order_btn {
    background: #fff;
    border: 1px solid rgb(180, 180, 180);
    color: #999;
    text-decoration: none;
    margin: 30px 5px;
  }
  .tip{
    font-size: 16px;
    text-align: center;
    padding-bottom: 20px;
  }
  .c_green{
    color: #05d005;
  }
}
.codemodal{
  .tip{
    margin: 30px auto 0;
    text-align: center;
    font-size: 16px;
  }
  .code{
    width: 182px;
    height: 182px;
    border: 1px solid #ddd;
    margin: 20px auto;
  }
  .tip2{
    margin: 0 auto;
    width: 200px;
    padding: 13px 0 13px 70px;
    height: 66px;
    line-height: 20px;
    font-size: 14px;
    color: #fff;
    background: url(data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABGAAD/4QMsaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcD0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLyIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAoTWFjaW50b3NoKSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDpBQUJFOEQ1MjRBQUExMUU4OEJBREIyOUE3Mjg5MjBCRCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDpBQUJFOEQ1MzRBQUExMUU4OEJBREIyOUE3Mjg5MjBCRCI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjMyOURBNEZGNEE4ODExRTg4QkFEQjI5QTcyODkyMEJEIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOjMyOURBNTAwNEE4ODExRTg4QkFEQjI5QTcyODkyMEJEIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+/+4ADkFkb2JlAGTAAAAAAf/bAIQABAMDAwMDBAMDBAYEAwQGBwUEBAUHCAYGBwYGCAoICQkJCQgKCgwMDAwMCgwMDQ0MDBERERERFBQUFBQUFBQUFAEEBQUIBwgPCgoPFA4ODhQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQU/8AAEQgAKQApAwERAAIRAQMRAf/EAIAAAAICAwEAAAAAAAAAAAAAAAYHAwQBAggFAQACAwEAAAAAAAAAAAAAAAABBAADBQIQAAEDAwMDAwQABwAAAAAAAAIBAwQRBQYAEgchMRNRIjJBUiMUYXGBkUIzFREAAgMAAQMEAwEAAAAAAAAAAQIAEQMxIUESYSIyE/BRcQT/2gAMAwEAAhEDEQA/AGXaIHE+IcNYpmWW4iN6nXUzjPOtLR0nN76oRbjFKILdOmtpjq+rKrVUzQEXMMRcXnIGY8S32wJBwrDXLDefO24s4yFU8IoW8Ojhd6p9NM456q1s1iUaOhHtFGK3cPqmnIvD/jXJuOseG5JnuMHkRSFZWAoKKeFAQ/JXcYfKo/20tsmjV4GpdkyD5C46cODhLki25MFjwUbbLs9uclo9IWvuID2qOxwuqKNeukNPuzItrsxtPrcGl4nJ+8/uXWzM2Gl05Cv13wq0YFLGOlksrqvwzACR9TXyfM1JUVPyl/imqFxUOX7mWnQlQvYStFjWyy2xm7XRlJtzmop223HVGQaRaed+lFJFVFQAr1pVenckljQ4gAAFmYTMco8ZEzIRqGCoJMsx2QjCpV2iooG3rRaJqfWsnm0maG35Uw82EduDk7Qk6ykcUbjzRBKkHjToDqIiqO32l2pXQ6p/Iejf2b4VyFfsDC7hZBjkN6jfpzP2QJyjXuT2bSGi+5fXU1xXSr7SJoUuu8EPCPqurpVJC+K09NSGEOYbEv8A46qkQI8MGVFK0Z/Xb+KVRPXVWfxnb8xiWO3ccFx7cSOY8UQnmCuEl1lkJjckAl+FAbSX6UpRFrVf47VWbT7B+fr0l6hPCLG1qDOSwFtDjroDLY/WcdAWnV949wAzRFr6EunG+JuLjnpK16Bhu83II1P1hlPizTt40dJBp/TRXgQNzKWuoIb3rjabZeO7FyG7Padh315Y7UEQJHW1Ty9SJVov+pe3rpddg2hSuJa2dIG/c81hIeS22PDdfCLkUAEZim8SAzLjJ8G1NeguBWg7uhD07p17Nqb7QfIesqrj2Wstu25LZNRh4wcdbBkybI2kJAJSFFFaIZUWv10fNebEHi0vRYzOIgdxuLglkqIo263NkhrGMkp531GqCo19gd69VpTryT59BxCB49TzL3HnG03kNu+uRJ7UL/hRP3nvMBOK6NDXaO1Uovs+uudthnXTmHPMvfpAPzJ9umZTOprbYcW5B4Mw7GJmYW2wzrc65KeGS6ybie+QCATZPNqNUOvXWOztnszBSbmiFV8wLqLDkLibH8LsCXm2Zvb8hkq+2x/z4ni8u1xCVT9j7i0Gn26bx/0M7UVIi+mQUWDcWgTpzbfhblPCz28YuGg0/ki003QlFmHHGnHdnzwbkV1yuFjKwVZRpJuz8/m31Ud7zXx29e/fS++xzql8pblmH5NR7YBhmH8X2zLJCZ7abw5drY5GbZbdYZISAHFSn53NyrupSms7XR9SvtIoxzNFQH3A9JyBtX0XW3MySl8l1IZkO+pJJNCSRn9NGSa6kkm0JJ//2Q==) no-repeat 20px center #445f85;
  }
  .tip_panel{
    margin-top: 20px;
    padding: 20px;
    line-height: 20px;
    border-top: 1px solid #f1f1f1;
  }
}
.paymodal{
  position: fixed;
  left: 50%;
  top: 20%;
  z-index: 999;
  width: 490px;
  margin-left: -240px;
  background-color: #fff;
  border-radius: 8px;
  overflow: hidden;
  .modal_head{
    line-height: 46px;
    background-color: #f9f9f9;
    text-align: right;
    padding: 0 20px;
    .close{
      font-size: 14px;
    }
  }
  .modal_foot{
    border-top: 1px solid #e7e7eb;
    padding: 20px;
    text-align: center;
  }
  .table{
    width: 100%;
  }
  th{
    line-height: 36px;
    background-color: #f8f8f8;
  }
  td{
    vertical-align: middle;
    padding: 20px;
  }
  .name{
    width: 140px;
    padding-left: 12px;
    font-size: 16px;
  }
  .img{
    display: inline-block;
    width: 120px;
    padding: 0;
  }
  .name{
    display: inline-block;
    vertical-align: top;
  }
  .tabs {
    height: 48px;
    .tab{
      float: left;
      line-height: 46px;
      text-align: center;
      font-size: 16px;
      width: 50%;
    }
    span{
      cursor: default;
      display: inline-block;
      line-height: 46px;
    }
    .on{
      color: #ee7e00;
      span{
        border-bottom: 2px solid #ee7e00;
      }
    }
  }
  .remark{
    padding: 20px;
    overflow: hidden;
    font-size: 14px;
    border-top: 1px solid #f1f1f1;
    input{
      text-indent: .5em;
      height: 22px;
      width: 350px;
      border: none;
    }
  }
  .pay_type{
    padding: 20px;
    overflow: hidden;
    border-top: 1px solid #f1f1f1;
    input{
      display: none;
      &:checked + .pay{
        border-color: #ec7f00;
      }
    }
  }
  .pay{
    width: 218px;
    height: 50px;
    border: 1px solid #fff;
    float: left;
    background-repeat: no-repeat;
    background-position: center center;
    &.pay_ali {
      background-image: url(data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABGAAD/4QMsaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcD0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLyIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAoTWFjaW50b3NoKSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDozMjlEQTRGOTRBODgxMUU4OEJBREIyOUE3Mjg5MjBCRCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDozMjlEQTRGQTRBODgxMUU4OEJBREIyOUE3Mjg5MjBCRCI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjMyOURBNEY3NEE4ODExRTg4QkFEQjI5QTcyODkyMEJEIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOjMyOURBNEY4NEE4ODExRTg4QkFEQjI5QTcyODkyMEJEIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+/+4ADkFkb2JlAGTAAAAAAf/bAIQABAMDAwMDBAMDBAYEAwQGBwUE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);
    }
    &.pay_weixin {
      background-image: url(data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABGAAD/4QMsaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcD0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLyIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAoTWFjaW50b3NoKSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDozMjlEQTRGRDRBODgxMUU4OEJBREIyOUE3Mjg5MjBCRCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDozMjlEQTRGRTRBODgxMUU4OEJBREIyOUE3Mjg5MjBCRCI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjMyOURBNEZCNEE4ODExRTg4QkFEQjI5QTcyODkyMEJEIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOjMyOURBNEZDNEE4ODExRTg4QkFEQjI5QTcyODkyMEJEIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+/+4ADkFkb2JlAGTAAAAAAf/bAIQABAMDAwMDBAMDBAYEAwQGBwUE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);
    }
  }
}
.address-box {
  margin-top: 10px;
  cursor: pointer;
}
.car-text {
  font-size: 14px;
}

.car-item {
  /*margin-bottom: 18px;*/
}

.car-clearfix:before,
.car-clearfix:after {
  display: table;
  content: "";
}
.car-clearfix:after {
  clear: both
}

.box-card {
  width: 465px;
  margin-bottom: 5px;
}
.address-into {
  float: right;
  margin-top: -44px;
}
.address-edit {
  display: inline-block;
  position:relative;
  right:0;
  float: right;
  top:15px;
}
.address-info {
  position: relative;
  left:-30px;
  text-align: left;
  display: inline-block
}
.box-radio {
  display: inline-block;
  width: 20px;
  bottom: 15px;
}
.address_btn {
  display: inline-block;
  line-height: 34px;
  height: 34px;
  width: 110px;
  border-radius: 3px;
  background-color: #D51423;
  color: #fff;
  border: 1px solid #D51423;
  text-align: center;
  margin-left: 0px;
  margin-bottom: 8px;
}
.loading span {
   display: inline-block;
   width: 20px;
   height: 20px;
   border: 2px solid #409eff;
   border-left: transparent;
   animation: zhuan 0.5s linear infinite;
   border-radius: 50%;
 }
@keyframes zhuan {
  0% {
    transform: rotate(0);
  }
  100% {
    transform: rotate(360deg);
  }
}
form .form_group {
  top: 10px;
}
form .form_group .label {
  clear: both;
  text-align: right;
  float: left;
  display: block;
  width: 80px;
  line-height: 36px;
  color: #333;
  margin-top: 10px;
}
form .form_group .form_ctl {
  margin-left: 12px;
  float: left;
  width: 550px;
  position: relative;
  z-index: 9;
  margin-top: 10px;
}
form .form_group .form_ctl .form_input {
  height: 34px;
  border: 1px solid #e7e7e8;
  width: 278px;
  text-indent: 1em;
}
form .form_group .form_ctl .form_switch {
  height: 34px;
  border: 0;
  text-indent: 1em;
}
</style>
