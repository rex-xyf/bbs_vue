<template>
    <div class="login-container">
      <el-form ref="registerForm" :model="registerForm" class="login-form" label-position="top">
        <h1>Register</h1>
        <el-form-item label="Username" prop="username">
          <el-input v-model="registerForm.username" placeholder="Username" clearable></el-input>
        </el-form-item>
        <el-form-item label="Password" prop="password">
          <el-input v-model="registerForm.password" placeholder="Password" type="password" clearable></el-input>
        </el-form-item>
        <el-form-item label="Confirm Password" prop="confirmPassword">
          <el-input v-model="registerForm.confirmPassword" placeholder="Confirm Password" type="password" clearable></el-input>
        </el-form-item>
        <el-form-item label="Email" prop="emailFont">
          <el-input v-model="registerForm.emailFont" placeholder="Email" style="width: 60%;"></el-input>
          <el-select v-model="emailSuffix" placeholder="Select" style="display: inline-block; width: 40%;">
            <el-option label="@qq.com" value="@qq.com" selected></el-option>
            <el-option label="@gmail.com" value="@gmail.com"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="Verification Code" prop="verificationCode" :style="{ display: showVerificationCode ? 'block' : 'none' }">
          <el-input v-model="registerForm.verificationCode" placeholder="Verification Code" clearable></el-input>
        </el-form-item>
        <el-form-item>
            <el-button @click="getVerificationCode" type="primary">Get Verification Code</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="register">Register</el-button>
          <el-button @click="goLogin" type="text">go back to login</el-button>
          <el-button @click="openMessage" icon="el-icon-question" type="text"></el-button>
        </el-form-item>
      </el-form>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    name: 'Register',
    data() {
      return {
        registerForm: {
          username: '',
          password: '',
          confirmPassword: '',
          emailFont: '',
          verificationCode: '',
          email:''
        },
        emailSuffix: '@qq.com',
        showVerificationCode: false
      };
    },
    methods: {
      async register() {
        const { username, password, confirmPassword, email, verificationCode ,emailFont} = this.registerForm;

        // Username validation
        if (!/^[a-zA-Z0-9._]{6,15}$/.test(username)) {
            this.$message.error('Username must be 6-15 characters and can only contain letters (case insensitive), numbers, ".", and "_"');
            return;
        }

        // Password validation
        if (!/^[a-zA-Z0-9._]{6,15}$/.test(password)) {
            this.$message.error('Password must be 6-15 characters and can only contain letters (case insensitive), numbers, ".", and "_"');
            return;
        }

        // Confirm password validation
        if (password !== confirmPassword) {
            this.$message.error('Passwords do not match');
            return;
        }

        if (emailFont === '') {
            this.$message.error('Please set email');
            return;
        }

        if (verificationCode === '') {
            this.$message.error('Please set verificationCode');
            return;
        }

        try {
          this.registerForm.email = `${this.registerForm.emailFont}${this.emailSuffix}`;
          const response = await axios.post('http://127.0.0.1:5000/api/register', this.registerForm);
          if (response.data.success) {
            this.$message.success('Registration successful');
            // Redirect to login page or other page
            this.$router.push({ name: 'login' });
          } else {
            this.$message.error(response.data.message);
          }
        } catch (error) {
          this.$message.error('Failed to register');
        }
      },
      async getVerificationCode() {
        const { username, password, confirmPassword, email, verificationCode,emailFont } = this.registerForm;

        // Username validation
        if (!/^[a-zA-Z0-9._]{6,15}$/.test(username)) {
            this.$message.error('Username must be 6-15 characters and can only contain letters (case insensitive), numbers, ".", and "_"');
            return;
        }

        // Password validation
        if (!/^[a-zA-Z0-9._]{6,15}$/.test(password)) {
            this.$message.error('Password must be 6-15 characters and can only contain letters (case insensitive), numbers, ".", and "_"');
            return;
        }

        // Confirm password validation
        if (password !== confirmPassword) {
            this.$message.error('Passwords do not match');
            return;
        }

        if (emailFont === '') {
            this.$message.error('Please set email');
            return;
        }
        this.registerForm.email = `${this.registerForm.emailFont}${this.emailSuffix}`;
        // Implement verification code logic here
        const response = await axios.post('http://127.0.0.1:5000/api/send_verification_code', this.registerForm);
        if (response.data.success) {
            this.showVerificationCode = true;
            this.$message.success('Verification code sent successfully');
          } else {
            this.$message.error(response.data.message);
          }
      },
      goLogin(){
      localStorage.setItem('refreshed','false')
      this.$router.push({ name: 'login' });
      },
      openMessage() {
        this.$notify.info({
          title: 'message',
          message: 'Username and Password must be 6-15 characters and can only contain letters (case insensitive), numbers, ".", and "_"'
        });
      },
    }
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
    margin-top: 50px;
  }
  
  .el-form-item {
    margin-bottom: 20px;
  }
  
  .el-button {
    width: 100%;
  }
  </style>
  