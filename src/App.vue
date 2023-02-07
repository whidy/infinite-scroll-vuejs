<template>
  <div id="app">
    <h1>Random User</h1>
    <div class="person" v-for="(person, index) in persons" :key="index">
      <div class="left">
        <img :src="person.picture.large" alt="" />
      </div>
      <div class="right">
        <p>{{ person.name.first }} {{ person.name.last }}</p>
        <ul>
          <li><strong>Birthday:</strong> {{ format(new Date(person.dob.date), 'MM/dd/yyyy') }}</li>
          <div class="text-capitalize"><strong>Location:</strong> {{ person.location.city }}, {{ person.location.state }}</div>
        </ul>
      </div>
    </div>
    <div class="loading" v-show="isLoading">Loading ...</div>
  </div>
</template>

<script>
import axios from 'axios'
import { format } from 'date-fns'
export default {
  name: 'app',
  // 创建一个存放用户数据的数组
  data() {
    return {
      persons: [],
      isLoading: false,
      format
    }
  },
  methods: {
    // axios请求接口获取数据
    getInitialUsers() {
      axios.get('https://randomuser.me/api/?results=10').then((response) => {
        this.persons = response.data.results
      })
    },
    scrollEvent() {
      // 距离底部200px时加载一次
      let bottomOfWindow = document.documentElement.offsetHeight - document.documentElement.scrollTop - window.innerHeight <= 200
      if (bottomOfWindow && this.isLoading == false) {
        this.isLoading = true
        axios.get(`https://randomuser.me/api/?results=5`).then((response) => {
          this.persons = this.persons.concat(response.data.results)
          this.isLoading = false
        })
      }
    },
    scroll() {}
  },
  beforeMount() {
    // 在页面挂载前就发起请求
    this.getInitialUsers()
  },
  mounted() {
    window.addEventListener('scroll', this.scrollEvent)
  },
  beforeDestroy() {
    window.removeEventListener('scroll', this.scrollEvent)
  }
}
</script>

<style lang="scss">
h1 {
  text-align: center;
}

.person {
  background: #ccc;
  border-radius: 2px;
  width: 20%;
  margin: 0 auto 15px auto;
  padding: 15px;

  img {
    width: 100%;
    height: auto;
    border-radius: 2px;
  }

  p:first-child {
    text-transform: capitalize;
    font-size: 2rem;
    font-weight: 900;
  }

  .text-capitalize {
    text-transform: capitalize;
  }
}
.loading {
  text-align: center;
  color: blueviolet;
}
</style>
