<template>
  <div>
    <h2>Child Popup</h2>

    <el-row>
      Data in IndexedDB:
      <el-card class="box-card">
        <div class="text item">
          {{ dbData }}
        </div>
      </el-card>
      <el-button type="primary" round @click="writeData">写入数据</el-button>
      <el-button type="success" round @click="clearData">清除数据</el-button>
    </el-row>

    <el-row>
      Message From Parent:
      <el-card class="box-card">
        <div class="text item">
          {{ msg }}
        </div>
      </el-card>

      <el-button type="primary" round @click="login">登录</el-button>
      <el-button type="success" round @click="sign">签名</el-button>
    </el-row>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Dexie from 'dexie'

interface Friend {
  id?: number
  name?: string
  age?: number
}

class FriendDatabase extends Dexie {
  public friends: Dexie.Table<Friend, number> // id is number in this case

  public constructor() {
    super('FriendDatabase')
    this.version(1).stores({
      friends: '++id,name,age',
    })
    this.friends = this.table('friends')
  }
}

export default Vue.extend({
  data() {
    const db = new FriendDatabase()

    return {
      msg: '',
      dbData: '',
      db,
    }
  },

  async created() {
    window.addEventListener('message', this.receiveMessage, false)
    window.opener?.postMessage('connect', document.referrer)

    await this.loadData()
  },

  methods: {
    receiveMessage(event: any) {
      console.log('received event', event)
      console.log(event.origin, document.referrer)
      if (event.origin !== new URL(document.referrer).origin) return

      console.log('receive message from parent', event.data)
      this.msg = event.data
    },
    login() {
      window.opener?.postMessage('login result', document.referrer)
    },
    sign() {
      window.opener?.postMessage('sign result', document.referrer)
    },
    async loadData() {
      const friends = await this.db.friends.toArray()
      this.dbData = JSON.stringify(friends)
    },
    async writeData() {
      await this.db.friends.add({
        name: 'Camilla',
        age: 25,
      })
      await this.loadData()
    },
    async clearData() {
      await this.db.friends.clear()
      await this.loadData()
    },
  },
})
</script>
