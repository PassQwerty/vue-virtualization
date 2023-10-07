<template>
  <div class="app">
    <div class="containerPost">
      <h1>Страница с постами</h1>
      <my-input v-model="searchQuery" placeholder="Поиск..."/>
      <div class="appButtons">
        <my-button class="success btnSizeM" style="max-width: 10rem;" @click="showDialog">
          Создать пост
        </my-button>
        <my-select 
          v-model="selectedSort"
          :options="sortOptions"
        />
      </div>
      <my-dialog v-model:show="dialogVisible">
        <PostForm @create="createPost" />
      </my-dialog>
      <PostList v-if="isPostsLoading === false" :posts="sortedAndSearchedPosts" @remove="removePost" />
      <div v-else>
        <span class="loader"></span>
      </div>
      <div ref="observer" class="observer"></div>
    </div>
  </div>
</template>

<script>
import PostForm from "@/components/PostForm.vue";
import PostList from "@/components/PostList.vue";
import axios from '../node_modules/axios'

export default {
  components: {
    PostList,
    PostForm
  },
  data() {
    return {
      posts: [],
      dialogVisible: false,
      isPostsLoading: false,
      selectedSort: "",
      sortOptions: [
        {value: 'title', name: 'По названию'},
        {value: 'body', name: 'По описанию'},
      ],
      searchQuery: '',
      page: 1,
      limit: 10,
      totalPage: 0,
    };
  },
  methods: {
    createPost(post, second, third) {
      this.posts.push(post);
      this.dialogVisible = false;
    },
    removePost(post) {
      this.posts = this.posts.filter(currentPost => currentPost.id !== post.id)
    },
    showDialog() {
      this.dialogVisible = true;
    },
    async fetchPosts() {
      try {
        this.page += 1;
        const response = await axios.get('https://jsonplaceholder.typicode.com/posts', {
          params:{
            _page: this.page,
            _limit: this.limit,
          }
        });
        this.totalPage = Math.ceil(response.headers['x-total-count'] / this.limit);
        this.posts = [...this.posts, ...response.data];
      } catch (error) {
        alert(error)
      }
    },
  },
  mounted() { // выполнится один ращ в компоненте после того, как компонент был добавлен в DOM.
    this.fetchPosts();
    // console.log(this.$refs.observer)
    const options = {
      rootMargin: "0px",
      threshold: 1.0,
    };
    const callback = (entries, observer) => {
      if(entries[0].isIntersecting && this.page < this.totalPage){
        console.log("Пересечен");
        this.fetchPosts();
      }
    };
    const observer = new IntersectionObserver(callback, options);
    observer.observe(this.$refs.observer)
  },
  computed:{
    sortedPosts(){ // сортировка по алфавиту
      return [...this.posts].sort((post1, post2) =>  
        post1[this.selectedSort]?.localeCompare(post2[this.selectedSort]))
    },
    sortedAndSearchedPosts(){ // поиск через фильтер
      return this.sortedPosts.filter((currPost) => 
        currPost.title.toLowerCase().includes(this.searchQuery.toLowerCase()))
    },
  },
  watch:{ // свойство которое содержит методы наблюдателей за переменными

  }
};

</script>

<!-- свойство scoped применить стили только к выбранному компоненту -->
<style scoped>
.app {
  padding: 1rem;
}

.containerPost {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.appButtons{
  display: flex;
  justify-content: space-between;
}
.observer{
  /* height: 30px;
  
  /* визуальный вид конца строки */
  /* background-color: gray; */
}
</style>
