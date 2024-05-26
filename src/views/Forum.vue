<template>
  <div>
    <el-container>
      <el-header>
        <div>
          <h1 style="display:inline-block; margin-right: 20px;">My Forum</h1>
          <!-- 搜索框 -->
          <el-input
            v-model="searchKeyword"
            placeholder="Search Posts"
            style="width: 200px;"
          ></el-input>
          <el-button type="primary" @click="search">search</el-button>
          <el-button type="primary" @click="reset">reset</el-button>
          <span v-if="loggedIn" style="float: right; margin-top: 10px; margin-right: 1opx;">
            <el-avatar :size="40" :src="userAvatarSrc" style="margin-right: 10px; vertical-align: middle;"></el-avatar>
            <span style=" margin-right: 20px;">{{ username }}</span>
            <el-button type="primary" @click="logout">Logout</el-button>
          </span>
          <span v-else style="float: right; margin-top: 10px; margin-right: 1opx;">
            <el-avatar :size="40" icon="el-icon-user" style="margin-right: 10px; vertical-align: middle;"></el-avatar>
            <el-button type="primary" @click="goToLogin">login</el-button>
          </span>
        </div>
      </el-header>
      <el-container style="display: flex;flex-direction: row-reverse;">
      <el-aside width="25%">
        <el-card style="margin: 20px;">
          <HotSearches />
        </el-card>
      </el-aside>
      <el-main width="75%">
        <!-- 帖子列表 -->
        <el-card v-if="!selectedPost">
          <div slot="header" class="clearfix">
            <span style="margin-right: 30px;">Latest Posts</span>
            <el-button type="primary" @click="showDrawer">Add Post</el-button>
          </div>
          <div v-for="(post, index) in posts" :key="index">
            <el-card :body-style="{ padding: '20px' }">
              <h3>{{ post.title }}</h3>
              <p>{{ post.content }}</p>
              <div v-for="image in post.images" :key="image.id" style=" width: 150px;height: 150px;overflow: hidden; 
              display: inline-block; margin-right: 20px;">
                <el-image
                                style="width: 100%; height: auto; object-fit: cover;"
                                :src="image.url"
                                :preview-src-list="[image.url]"      
                        >
                </el-image>
              </div>
              <p>Author: {{ post.author }}</p>
              <p>Posted at: {{ post.created_time }}</p>
              <el-button type="text" @click="showPostDetails(post)">Read More</el-button>
            </el-card>
          </div>
        </el-card>

        <!-- 帖子详情 -->
        <el-card v-if="selectedPost">
          <div class="post-details">
            <h2>{{ selectedPost.title }}</h2>
            <p>{{ selectedPost.content }}</p>
            <div v-for="image in selectedPost.images" :key="image.id" style=" width: 150px;height: 150px;overflow: hidden; 
              display: inline-block; margin-right: 20px;">
                <el-image
                                style="width: 100%; height: auto; object-fit: cover;"
                                :src="image.url"
                                :preview-src-list="[image.url]"   
                        >
                </el-image>
            </div>
            <p>Author: {{ selectedPost.author }}</p>
            <p>Posted at: {{ selectedPost.created_time }}</p>
            <el-divider></el-divider>
            <h3>Comments</h3>
            <div v-if="aiCommentLoading" class="comment">
              <p style="color: #409EFF;">Loading AI comment...</p>
            </div>
            <div v-else-if="aiComment" class="comment">
              <p style="color: #409EFF;">{{ aiComment.content }}</p>
              <p style="color: #409EFF;">Comment by Ai</p>
            </div>
            <div v-else class="comment">
              <p style="color: #409EFF;">No AI comment available.</p>
            </div>
            <div class="comment" v-for="(comment, index) in comments" :key="index">
              <p>{{ comment.text }}</p>
              <div v-for="image in comment.images" :key="image.id" style=" width: 150px;height: 150px;overflow: hidden; 
              display: inline-block; margin-right: 20px;">
                <el-image
                                style="width: 100%; height: auto; object-fit: cover;"
                                :src="image.url"
                                :preview-src-list="[image.url]"   
                        >
                </el-image>
              </div>
              <p>Author: {{ comment.author }}</p>
              <p>Commented at: {{ comment.created_time }}</p>
            </div>
            <el-divider></el-divider>
            <el-input
              v-model="newComment.text"
              type="textarea"
              placeholder="Add a comment"
            ></el-input>
            <el-upload
            action="http://127.0.0.1:8083/upload"
            list-type="picture-card"
            :on-success="handleCommentImageUpload"
            :file-list="newComment.images"
            multiple
            :before-upload="beforeImageUpload"
          >
            <i class="el-icon-plus"></i>
          </el-upload>
            <el-button type="primary" @click="addComment">Submit</el-button>
            <el-button @click="backToList">Back to List</el-button>
          </div>
        </el-card>
      </el-main>
    </el-container>
    </el-container>

    <!-- 新帖子抽屉 -->
    <el-drawer title="Add Post" :visible.sync="drawerVisible" direction="rtl">
      <el-form ref="newPostForm" :model="newPost" style="margin:20px;">
        <el-form-item label="Title">
          <el-input v-model="newPost.title"></el-input>
        </el-form-item>
        <el-form-item label="Content">
          <el-input type="textarea" v-model="newPost.content"></el-input>
        </el-form-item>
        <el-upload
            action="http://127.0.0.1:8083/upload"
            list-type="picture-card"
            :on-success="handleImageUpload"
            :file-list="newPost.images"
            multiple
            :before-upload="beforeImageUpload"
          >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-button type="primary" @click="addPost">Submit</el-button>
      </el-form>
    </el-drawer>
  </div>
</template>

<script>
import axios from 'axios';
import { locale } from 'core-js/web/immediate';
import HotSearches from '../components/HotSearches.vue';

export default {
  name:'Forum',
  data() {
    return {
      posts: [],
      selectedPost: null,
      comments: [],
      newPost: {
        title: '',
        content: '',
        images: [],
        author:''
      },
      newComment: {
        text: '',
        images: [],
        author:''
      },
      aicomments: null,
      aiCommentLoading: true,
      drawerVisible: false,
      loggedIn: false,
      username: '',
      searchKeyword: ''
    };
  },
  computed: {
    userAvatarSrc() {
      return `https://ui-avatars.com/api/?name=${this.username}&background=random`;
    }
  },
  components: {
    HotSearches
  },
  methods: {
    async fetchPosts() {
      const response = await axios.get('http://127.0.0.1:8083/api/posts');
      this.posts = response.data.posts;
    },
    async search() {
      const response = await axios.get('http://127.0.0.1:8083/api/posts', {
        params: { search: this.searchKeyword }
      });
      this.posts = response.data.posts;
    },
    async reset() {
      this.searchKeyword = '';
      this.fetchPosts();
    },
    async addPost() {
      if (this.username === '' || this.loggedIn === false){
        this.$message({
          message: 'please log in',
          type: 'warning'
        });
      }else{
      const jwtToken = localStorage.getItem('jwtToken')
      axios.defaults.headers.common['Authorization'] = `Bearer ${jwtToken}`;
      const response = await axios.post('http://127.0.0.1:8083/api/posts', this.newPost);
      this.posts.push(response.data.post);
      this.drawerVisible = false;
      this.$message({message: 'success',type: 'success'});
      this.newPost = { title: '', content: '', images: [] , author:this.username};
      await axios.post('http://127.0.0.1:8082/api/getChat', response.data.post);
      }
    },
    async showPostDetails(post) {
      this.selectedPost = post;
      const response = await axios.get(`http://127.0.0.1:8083/api/posts/${post.id}/comments`);
      this.comments = response.data.comments;
      try {
        const res = await axios.get(`http://127.0.0.1:8082/api/posts/${this.selectedPost.id}/aiComments`);
        if (res.data && Object.keys(res.data).length > 0) {
          this.aiComment = res.data;
        } else {
          this.aiComment = null;
        }
      } catch (error) {
        console.error("Error loading AI comment:", error);
      } finally {
        this.aiCommentLoading = false; // 数据加载完成，设置加载状态为 false
      }
    },
    async addComment() {
      if (this.username === '' || this.loggedIn === false){
        this.$message({
          message: 'please log in',
          type: 'warning'
        });
      }else{
      const jwtToken = localStorage.getItem('jwtToken')
      axios.defaults.headers.common['Authorization'] = `Bearer ${jwtToken}`;
      const response = await axios.post(`http://127.0.0.1:8083/api/posts/${this.selectedPost.id}/comments`, this.newComment);
      this.comments.push(response.data.comment);
      this.$message({message: 'success',type: 'success'});
      this.newComment = { text: '', images: [] , author:this.username};
      }
    },
    async logout() {
      const jwtToken = localStorage.getItem('jwtToken')
      axios.defaults.headers.common['Authorization'] = `Bearer ${jwtToken}`;
      axios.post('http://127.0.0.1:8085/api/logout').then(response => {
        this.$message({
          message: 'logout success',
          type: 'success'
        });
        localStorage.removeItem('username');
        localStorage.removeItem('jwtToken')
        this.loggedIn = false;
        this.username = '';
        this.newPost.author = this.username;
        this.newComment.author = this.username;
    })
    .catch(error => {
        console.error('Logout failed:', error);
    });
    },
    goToLogin() {
      this.$router.push('/login')
    },
    backToList() {
      this.selectedPost = null;
      this.aicomments = null;
      this.aiCommentLoading = true;
    },
    showDrawer() {
      if (this.username === '' || this.loggedIn === false){
        this.$message({
          message: 'please log in',
          type: 'warning'
        });
      }else{
      this.drawerVisible = true;
      }
    },
    handleImageUpload(res, file) {
      this.newPost.images.push({ name: res.name ,url: URL.createObjectURL(file.raw)});
    },
    handleCommentImageUpload(res, file) {
      this.newComment.images.push({ name: res.name ,url: URL.createObjectURL(file.raw)});
    },
    beforeImageUpload(file) {
      const isImage = file.type.startsWith('image/');
      const isLt5M = file.size / 1024 / 1024 < 5;
      if (!isImage) {
        this.$message.error('只能上传图片格式文件');
      }
      if (!isLt5M) {
        this.$message.error('图片大小不能超过 5MB');
      }
      return isImage && isLt5M;
    },
  },
  async mounted() {
    this.fetchPosts();
    this.username = localStorage.getItem('username');
    this.loggedIn = !!this.username;
    this.newPost.author = this.username;
    this.newComment.author = this.username;
  }
};
</script>

<style scoped>
.el-header {
  background-color: #e2e2e2;
  color: #333;
  line-height: 15px;
}
.post-details {
  padding: 20px;
}
.comment {
  border-bottom: 1px solid #ddd;
  padding: 10px 0;
}
</style>
