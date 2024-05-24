<template>
    <div>
      <h1>Hot Searches</h1>
      <ul v-if="hotSearches.length > 0">
        <div>{{ hotSearches[0].time }}</div>
        <div>{{ hotSearches[0].top }}</div>
        <li v-for="(item, index) in hotSearches[0].items" :key="index">
          <span>{{ item.title }}</span>
          <span style="color: #67C23A; margin-left: 5px; margin-right: 5px;">{{ item.label }}</span>
          <a :href="item.link" target="_blank">Link</a>
        </li>
      </ul>
      <p v-else>Loading...</p>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        hotSearches: []
      };
    },
    mounted() {
      this.fetchHotSearches();
    },
    methods: {
      fetchHotSearches() {
        axios.get('http://127.0.0.1:8081/api/hotsearch')
          .then(response => {
            this.hotSearches = response.data;
            console.log(this.hotSearches)
          })
          .catch(error => {
            console.error('Error fetching hot searches:', error);
          });
      }
    }
  };
  </script>
  