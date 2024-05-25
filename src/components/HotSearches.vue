<template>
    <div class="hot-searches">
      <h1 class="title">Hot Searches</h1>
      <ul v-if="hotSearches.length > 0" class="hot-list">
        <div class="title" style="color: #999;">{{ hotSearches[0].time }}</div>
        <li class="hot-item" v-for="(item, index) in hotSearches[0].items" :key="index">
          <div class="item-content">
            <span class="title">{{ item.title }}</span>
            <span class="label">{{ item.label }}</span>
          </div>
          <a :href="item.link" target="_blank" class="link">Link</a>
        </li>
      </ul>
      <p v-else class="loading">Loading...</p>
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
          })
          .catch(error => {
            console.error('Error fetching hot searches:', error);
          });
      }
    }
  };
  </script>

<style scoped>
  .hot-searches {
  font-family: Arial, sans-serif;
  padding: 20px;
}

.title {
  font-size: 24px;
  color: #333;
  margin-bottom: 10px;
}

.hot-list {
  list-style-type: none;
  padding: 0;
}

.hot-item {
  border-bottom: 1px solid #eee;
  padding: 10px 0;
}

.item-content {
  display: inline-block;
}

.title {
  font-size: 16px;
  color: #409EFF;
}

.label {
  font-size: 14px;
  color: #67C23A;
  margin-left: 5px;
}

.link {
  color: #999;
  text-decoration: none;
  margin-left: 10px;
}

.link:hover {
  color: #333;
}
.loading {
  color: #999;
}
</style>
  