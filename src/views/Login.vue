<template>
  <div class="login-container">
    <el-form ref="loginForm" :model="loginForm" class="login-form">
      <h1>Login</h1>
      <el-form-item label="Username">
        <el-input v-model="loginForm.username" placeholder="Username" required></el-input>
      </el-form-item>
      <el-form-item label="Password">
        <el-input v-model="loginForm.password" placeholder="Password" type="password" required></el-input>
      </el-form-item>
      <el-form-item>
        <div class="g-recaptcha" :data-sitekey="siteKey"></div>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="login">Login</el-button>
        <el-button @click="goRegister" type="text">Register</el-button>
        <el-button @click="goMain" type="text">Back to Forum</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import axios from 'axios';
import vueRecaptcha from 'vue3-recaptcha2';

export default {
  name: 'Login',
  data() {
    return {
      loginForm: {
        username: '',
        password: ''
      },
      siteKey: '6LeLzeUpAAAAADnjWtquUwOYFKcMqp_XxlawukOo' // Google reCAPTCHA
    };
  },
  components: {
	  vueRecaptcha
  },
  methods: {
    async login() {
      if (this.loginForm.username === '') {
        this.$message.warning('Please enter username');
      } else if (this.loginForm.password === '') {
        this.$message.warning('Please enter password');
      } else {
        const recaptchaResponse = grecaptcha.getResponse();
        if (!recaptchaResponse) {
          this.$message.warning('Please complete the reCAPTCHA');
          return;
        }

        try {
          const response = await axios.post('http://127.0.0.1:8085/api/login', {
            ...this.loginForm,
            recaptchaResponse
          });
          if (response.data.message === 'Logged in successfully') {
            const token = response.data.token;
            localStorage.setItem('jwtToken', token);
            localStorage.setItem('username', response.data.username);
            localStorage.setItem('refreshed','false')
            this.$router.push({ name: 'forum' });
          }
        } catch (error) {
          this.$message.error('Invalid username or password');
        }
      }
    },
    goRegister(){
      localStorage.setItem('refreshed','false')
      this.$router.push({ name: 'register' });
    },
    goMain(){
      localStorage.setItem('refreshed','false')
      this.$router.push({ name: 'forum' });
    }
    
    },
    mounted() {
    // 检查 localStorage 中是否存在已刷新标志
    const refreshed = localStorage.getItem('refreshed');
    console.log(refreshed)
    // 如果尚未刷新过，则执行刷新操作
    if (!refreshed || refreshed === 'false') {
      // 设置已刷新标志为 true
      localStorage.setItem('refreshed', 'true');

      // 执行页面刷新
      window.location.reload();
    }
  },
};
</script>

<style scoped>
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
.login-form {
  width: 300px;
}
</style>
