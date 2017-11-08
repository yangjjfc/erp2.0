<template>
    <div class="loginpage">
        <div class="loginbox">
            <section>
                <h3 class="logo"></h3>
                <div class="loginForm">
                    <div class="showinfo">
                         <h4 v-show="!errorMsg.length">用户账号登录</h4>
                        <div class="errorMsg"   v-show="errorMsg.length">
                            <i class="iconfont icon-tongyong-"></i>
                            <p class="error" v-text="errorMsg"></p>
                        </div>
                    </div>
                    <el-form ref="form" :model="form" >
                         <el-form-item >
                            <el-input prefix-icon="iconfont icon-denglu-1" placeholder="请输入用户名" v-model="form.username" @keyup.enter.native="login"></el-input>
                        </el-form-item>
                        <el-form-item >
                            <el-input prefix-icon="iconfont icon-denglu-" placeholder="请输入密码" v-model="form.password" type="password" @keyup.enter.native="login"></el-input>
                        </el-form-item>
                        <el-form-item class="custom-mgb">
                            <el-col :span="11">
                                <el-input  placeholder="验证码" v-model="form.verifycode" :maxlength="4" @keyup.enter.native="login"></el-input>
                            </el-col>
                            <el-col :span="13" class="verifyimg">
                                    <img id="imageCode" class="textbox"  :src="verifyImg" width="80" height="40" alt="验证码" title="点击刷新" />
                                    <a @click="refreshCode()" href="javascript:void(0)">换一张？</a>
                            </el-col>
                        </el-form-item>
                        <el-form-item class="custom-mgb">
                            <el-col :span="12">
                                <el-checkbox @change="rememberUser">记住用户名</el-checkbox>
                            </el-col>
                             <el-col :span="12" class="fg-passwd">
                                <el-button type="text">忘记密码</el-button>
                            </el-col>
                        </el-form-item>
                        <el-form-item class="submit">
                            <el-button type="primary" @click="login">立即登录</el-button>
                        </el-form-item>
                    </el-form>
                </div>
            </section>
        </div>
    </div>
</template>
<script type="text/javascript">
import { mapGetters, mapActions } from 'vuex';
import {encryption} from '~/global.common.js';
const URL = {
    VERIFY_CODE: '/verifyCode'
};
export default {
    data () {
        return {
            verifyImg: '',
            errorMsg: '',
            form: {
                username: 'yosemite',
                password: '123456',
                verifycode: '1243'
            }
        };
    },
    computed: {
        ...mapGetters(['userInfo'])
    },
    methods: {
        ...mapActions({
            'userLogin': 'login',
            'currentUser': 'currentUser',
            'getroles': 'getroles'
        }),
        // 记住用户名
        rememberUser (val) {
            if (val && this.form.username) {
                localStorage.setItem('name', this.form.username);  
            }
        },
        // 获取用户名
        getUser () {
            let name = localStorage.getItem('name');
            name && (this.form.username = name); 
        },
        // 刷新验证码  
        getRandomImg () {
            this.verifyImg = process.env.SERVER + URL.VERIFY_CODE + '?t=' + Math.round(Math.random() * 1000000);
        },
        // 获取用户信息
        refreshCode () {
            if (this.userInfo && this.userInfo.token) {
                this.getRandomImg(); // 通过验证码获取token
            } else {
                this.currentUser().then(() => {
                    this.getRandomImg();
                }, () => {
                    this.getRandomImg();
                });
            }     
        },
        // 登录
        login () {
            let xflag = false;
            let clientid = this.userInfo ? this.userInfo.clientId : null;
            let token = this.userInfo ? this.userInfo.token : null;
            let password = this.form.password;
            if (!/^[a-zA-Z0-9_-]{6,20}$/.test(this.form.username)) {
                xflag = true;
                this.errorMsg = this.form.username === '' ? '请输入用户名' : '用户名格式不正确';
                return;
            }

            if (!/^[a-zA-Z0-9_-]{6,20}$/.test(this.form.password)) {
                xflag = true;
                this.errorMsg = this.form.password === '' ? '请输入密码' : '密码长度6-20个字符';
                return;
            }
            if (!/^[a-zA-Z0-9]{4}$/.test(this.form.verifycode)) {
                xflag = true;
                this.errorMsg = this.form.verifycode === '' ? '请输入验证码' : '验证码格式不正确';
                return;
            }
            let param = Object.assign({}, this.form, {
                password: encryption(password, clientid, token)
            });
            if (!xflag) {
                this.userLogin(param).then(msg => {
                    this.$router.push('/');
                }, err => {
                    // 记录错误信息
                    this.errorMsg = err.message;
                    this.refreshCode();
                });
            }
        }
    },     
    mounted () {
        this.getUser();
        this.refreshCode();
    }    
};
</script>

<style  lang="scss" rel="stylesheet/scss">
    .loginpage{
        position: absolute;
        width: 100%;
        height: 100%;
        background: #cacbd2;
        display: flex;
        justify-content:center;
        align-items: center;
        overflow: hidden;
        .loginbox{
            position: relative;
            width: 1680px;
            height: 960px;
            background:url(img/login-bg.png) no-repeat;
            section{
                width: 366px;
                height: 500px;
                position:absolute;
                top: 203px;
                left: 973px;
                h3{
                    width: 100%;
                    height: 62px;
                    background: url(img/logo.png) no-repeat 50%;
                    margin-bottom: 15px;
                }
                .loginForm{
                    width: 366px;
                    background: #fff;
                    border:1px #a5b7c9 solid;
                    padding: 28px 32px 16px 32px;
                    .showinfo{
                        height: 35px;
                        overflow: hidden; 
                        margin-bottom: 10px;
                         h4{
                            color: #44667f;
                            font-size: 15px;
                            }
                        .errorMsg{
                                background-color: #fef2f2;
                                color: #6C6C6C;
                                line-height: 16px;
                                padding: 6px 10px;
                                border: #ffb4a8 1px solid;
                                clear: both;
                                overflow: hidden;
                            i{
                                color: #f40;
                                padding-right: 10px;
                                float: left;
                                font-size: 16px;
                            }
                            .error{
                                float: left;
                                white-space: normal;
                                word-wrap: break-word;
                                word-break: break-all;
                                width: 250px;
                            }
                        }
                    }
                   
                    .el-form-item {
                        margin-bottom: 18px;
                    }
                    .custom-mgb{
                         margin-bottom: 12px;
                    }
                    .verifyimg{
                        float: right;
                        display: flex;
                        align-items: center;
                        padding-left: 10px;
                        img{
                            margin-right: 8px;
                        }
                    }
                    .fg-passwd{
                        display: flex;
                        justify-content: flex-end;
                        padding-right: 3px;
                    }
                    .submit{
                        button{
                            width: 100%;
                        }
                    }
                }
            }
        }
    } 
</style>
