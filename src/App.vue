<template>
  <div id="app">
    <h1>Random User</h1>
    <div class="person" v-for="(person, index) in persons" :key="index">
      <div class="left">
        <img :src="person.picture.large" alt="">
      </div>
      <div class="right">
        <p>{{ person.name.first}} {{ person.name.last }}</p>
        <ul>
          <li>
            <strong>Birthday:</strong> {{ formatDate(person.dob.date)}}
          </li>
          <div class="text-capitalize">
            <strong>Location:</strong> {{ person.location.city}}, {{ person.location.state }}
          </div>
        </ul>
      </div>

    </div>
  </div>
</template>

<script>
import axios from 'axios'
import moment from 'moment'
export default {
  name: 'app',
  // 创建一个存放用户数据的数组
  data() {
    return {
      persons: []
    }
  },
  methods: {
    // axios请求接口获取数据
    getInitialUsers() {
      for (var i = 0; i < 5; i++) {
        axios.get(`https://randomuser.me/api/`).then(response => {
          this.persons.push(response.data.results[0])
        })
      }
    },
    formatDate(date) {
      if (date) {
        return moment(String(date)).format('MM/DD/YYYY')
      }
    },
    scroll(person) {
      let isLoading = false
      window.onscroll = () => {
        // 距离底部200px时加载一次
        let bottomOfWindow = document.documentElement.offsetHeight - document.documentElement.scrollTop - window.innerHeight <= 200
        if (bottomOfWindow && isLoading == false) {
          isLoading = true
          axios.get(`https://randomuser.me/api/`).then(response => {
            person.push(response.data.results[0])
            isLoading = false
          })
        }
      }
    }
  },
  beforeMount() {
    // 在页面挂载前就发起请求
    this.getInitialUsers()
  },
  mounted() {
    this.scroll(this.persons)
  }
}
</script>

<style lang="scss">
h1 {
  text-align:center;
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
</style>
