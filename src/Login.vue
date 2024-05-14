<template>
    <div class="login-container">
      <el-form ref="loginForm" :model="loginForm" class="login-form">
        <el-form-item>
          <el-input v-model="loginForm.username" placeholder="Username"></el-input>
        </el-form-item>
        <el-form-item>
          <el-input v-model="loginForm.password" placeholder="Password" type="password"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="login">Login</el-button>
        </el-form-item>
      </el-form>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        loginForm: {
          username: '',
          password: ''
        }
      };
    },
    methods: {
      async login() {
        try {
          const response = await axios.post('http://127.0.0.1:5000/api/login', this.loginForm);
          if (response.data.message === 'Logged in successfully') {
            this.$emit('login-success', response.data.username);
            this.$router.push('/');
          }
        } catch (error) {
          this.$message.error('Invalid username or password');
        }
      }
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
  }
  </style>
  