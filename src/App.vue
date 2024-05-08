<template>
  <div>
    <el-container>
      <el-header>
        <h1>My Forum</h1>
      </el-header>
      <el-main>
        <!-- 帖子列表 -->
        <el-card v-if="!selectedPost">
          <div slot="header" class="clearfix">
            <span>Latest Posts</span>
            <el-button type="primary" @click="showDrawer">Add Post</el-button>
          </div>
          <div v-for="(post, index) in posts" :key="index">
            <el-card :body-style="{ padding: '20px' }">
              <h3>{{ post.title }}</h3>
              <p>{{ post.content }}</p>
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
            <p>Author: {{ selectedPost.author }}</p>
            <p>Posted at: {{ selectedPost.created_time }}</p>
            <el-divider></el-divider>
            <h3>Comments</h3>
            <div class="comment" v-for="(comment, index) in comments" :key="index">
              <p>{{ comment.text }}</p>
              <p>{{ comment.author }} - {{ comment.created_time }}</p>
            </div>
            <el-divider></el-divider>
            <el-form :model="newComment" inline>
              <el-form-item>
                <el-input v-model="newComment.author" placeholder="Your Name"></el-input>
              </el-form-item>
              <el-form-item>
                <el-input v-model="newComment.text" placeholder="Your Comment"></el-input>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="addComment">Add Comment</el-button>
              </el-form-item>
            </el-form>
            <el-button @click="goBack">Back to Posts</el-button>
          </div>
        </el-card>
      </el-main>
      <el-footer></el-footer>
    </el-container>

    <!-- 添加帖子的 Drawer -->
    <el-drawer title="Add Post" :visible.sync="drawerVisible" :with-header="false">
      <el-form :model="newPost" label-position="top">
        <el-form-item label="Title">
          <el-input v-model="newPost.title"></el-input>
        </el-form-item>
        <el-form-item label="Content">
          <el-input type="textarea" v-model="newPost.content"></el-input>
        </el-form-item>
        <el-form-item label="Author">
          <el-input v-model="newPost.author"></el-input>
        </el-form-item>
        <el-button type="primary" @click="addPost">Submit</el-button>
        <el-button @click="closeDrawer">Cancel</el-button>
      </el-form>
    </el-drawer>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      posts: [],
      selectedPost: null,
      comments: [],
      newComment: {
        author: '',
        text: ''
      },
      newPost: {
        title: '',
        content: '',
        author: ''
      },
      drawerVisible: false
    };
  },
  mounted() {
    this.fetchPosts();
  },
  methods: {
    fetchPosts() {
      axios.get('http://127.0.0.1:5000/api/posts')
        .then(response => {
          this.posts = response.data.posts;
        })
    },
    showPostDetails(post) {
      this.selectedPost = post;
      this.fetchComments(post.id);
    },
    fetchComments(postId) {
      axios.get(`http://127.0.0.1:5000/api/posts/${postId}/comments`)
        .then(response => {
          this.comments = response.data.comments;
        })
    },
    goBack() {
      this.selectedPost = null;
      this.comments = [];
    },
    addComment() {
      axios.post(`http://127.0.0.1:5000/api/posts/${this.selectedPost.id}/comments`, {
          author: this.newComment.author,
          text: this.newComment.text
        })
        .then(response => {
          this.comments.push(response.data.comment);
          this.newComment.text = '';
        })
    },
    showDrawer() {
      this.drawerVisible = true;
    },
    closeDrawer() {
      this.drawerVisible = false;
    },
    addPost() {
      axios.post('http://127.0.0.1:5000/api/posts', this.newPost)
        .then(response => {
          this.posts.push(response.data.post);
          this.newPost.title = '';
          this.newPost.content = '';
          this.newPost.author = '';
          this.drawerVisible = false;
        })
    }
  }
};
</script>

<style scoped>
.post-details {
  margin-top: 20px;
  padding: 20px;
}

.post-details h2 {
  margin-bottom: 10px;
}

.comment {
  margin-top: 20px;
  border-top: 1px solid #ccc;
  padding-top: 10px;
}
</style>

