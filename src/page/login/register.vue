<template>
  <div class="reg">
    <div class="login-header">
      <div>
        <span @click="$router.go(-1)">返回</span>
        <router-link to="/login"><span>登录</span></router-link>
      </div>
    </div>
    <div v-if="!success">
      <div class="logo">
        <img src="/static/images/logo.png" style="width: 100%"/>
      </div>
      <div class="form">
        <div>
          <span>手机号</span>
          <div style="width:75%;display: flex;justify-content: space-between">
            <input v-model="form.tel" type="tel" style="width: 100px"/>
            <mt-button size="small" :type="verify?'primary':'default'" @click="fetchCode">{{msg}}</mt-button>
          </div>
        </div>
        <div>
          <span>验证码</span><input v-model="form.verifyCode" type="number"/>
        </div>
        <div>
          <span>密码</span><input v-model="form.password" type="password"/>
        </div>
        <div>
          <span>确认密码</span><input v-model="form.repassword" type="password"/>
        </div>

      </div>
      <div>
        <mt-button class="btn" type="primary" size="normal" @click="send">立即注册</mt-button>
      </div>
    </div>
    <div class="success" v-if="success">
      <p>恭喜您，注册成功！</p>
      <div><img src="/static/icon/right.svg"/></div>
      <mt-button class="btn" type="primary" size="normal" @click="$router.push({path:'/login'})">立即登录</mt-button>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import {test,user,getVerifyCode} from '../../service/getData'
  export default {
    data(){
      return {
        verify: false,
        form: {
          username: null,
          password: null,
          repassword: null,
          tel: null,
          verifyCode:null,
        },
        msg:'获取验证码',
        success:false
      }
    },
    mounted(){
    },
    watch: {
      'form.tel'(val){
        if(this.checkTel(val)){
          this.verify = true
        }else{
          this.verify = false
        }
      }
    },
    methods: {
      async send(){
        //各种验证
        //验证密码以及确认密码
        if(this.form.password != this.form.repassword){
          this.$toast('两次密码不一致！')
          return false
        }
        //验证手机号
        if (!this.checkTel(this.form.tel)) {
          this.$toast('请输入正确的手机号码！')
        }
        let data = Object.assign({}, this.form)
        delete data.repassword
        let res = await user('POST', {}, data)
        if(res.error_code == 10006){
          this.$toast('验证码错误！')
          return false
        }
//        this.$toast({
//          message: '注册成功',
//          iconClass: 'icon icon-success'
//        });
        if(res.error_code == 0){
          this.success = true
        }
      },
      async fetchCode(){
        if (!this.verify) {
          this.$toast('请输入正确的手机号码！')
          return false
        }
        //去获取验证码
        this.verify = false
        this.countDown(3)
        //请求服务器验证码
        let res = await getVerifyCode('POST',{},{tel:this.form.tel})
        if(res.error_code == '10007'){
          this.$toast('手机号已被注册')
        }
        console.log(res)
      },
      //验证手机号
      checkTel(val){
        let validate = /^1[34578]\d{9}$/
        if (!validate.test(val)) {
          return false
        }else{
          return true
        }
      },
      countDown(times){
        if (!times || isNaN(parseInt(times))) {
          this.msg = '重新获取'
          this.verify = true
          return
        }
        this.msg = times + 'S'
        setTimeout(()=> {
          this.countDown(--times)
        }, 1000)
      }
    },
  }

</script>
<style>
  .icon-success{
    width: 50px;
    height: 50px;
    background: url("/static/icon/right.svg") 0 0 no-repeat;
    background-size: 100% 100%;
  }
</style>
<style lang="less" scoped>
  @theme-color : #758dcd;
  @placeholder : rgba(255,255,255,0.5);

  .reg{
    position: fixed;
    width: 100%;
    height: 100%;
    text-align: center;
    background:url('/static/images/login_background.png') 0px 0px no-repeat;
    background-size:100% 100%;
      .form {
          ::-webkit-input-placeholder { /* WebKit browsers */
            color: @placeholder;
          }
          :-moz-placeholder { /* Mozilla Firefox 4 to 18 */
            color: @placeholder;
            opacity: 1;
          }
          ::-moz-placeholder { /* Mozilla Firefox 19+ */
            color: @placeholder;
            opacity: 1;
          }
          :-ms-input-placeholder { /* Internet Explorer 10+ */
            color: @placeholder;
          }
          span{
            margin-right:10px;
            display:inline-block;
            line-height: 40px;
            width: 20%;
            font-size: 14px;
            text-align:justify;
            text-justify:distribute-all-lines;/*ie6-8*/
            text-align-last:justify;/* ie9*/
            -moz-text-align-last:justify;/*ff*/
            -webkit-text-align-last:justify;/*chrome 20+*/
          }
          >div{
            color:white;
            display:flex;
            margin:auto;
            width:80%;
            border-bottom: 1px solid white;

            input {
              /*background-color: rgba(255, 255, 255, 0.5);*/
              color: white;
              outline: none;
              display: inline;
              border:0px;
              background-color: rgba(255, 255, 255, 0);
              height: 40px;
              width: 70%;
              display: block;
              text-align: left;
              font-size: 15px;
            }
          }

      }
    .login-header{
      position: fixed;
      width: 100%;
      top: 0;
      height:auto;
      div{
        padding:20px;
        width: auto;
        display: flex;
        justify-content: space-between;
        color:white;
        span{
          color: white;
        }
      }
    }
    .btn{
      margin-top: 20px;
      border-radius: 20px;
      width: 40%;
    }
  }

  .logo{
    margin: 100px auto 30px auto;
    width: 100px;
    height:100px;
    border-radius: 50px;
    overflow: hidden;
    border:0.5px solid white;
  }
  .success{
    padding-top:100px;
    p{
      font-size: 16px;
      color: white;
    }
    img{
      width: 100px;
      height: 100px;
    }

  }

</style>
