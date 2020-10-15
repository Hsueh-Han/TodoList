<template>
  <div class="position-fixed" style="z-index: 1030; right:20px; top:20px;">
    <div class="alert p-2 mb-1 rounded-left" :class="'alert-' + item.status"
    v-for="(item, key) in message" :key="key">
      <strong>{{item.msg}}</strong>
      <button type="button" class="close ml-2" data-dismiss="alert" aria-label="Close"
      @click="removeMessage(key)">
        <span aria-hidden="true">&times;</span>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      message: []
    }
  },
  methods: {
    updateMessage (msg, status) {
      const timeStamp = Math.floor(new Date())
      this.message.push({
        msg,
        status,
        timeStamp
      })
      this.removeMessageWithTiming(timeStamp)
    },
    removeMessage (num) {
      this.message.splice(num, 1)
    },
    removeMessageWithTiming (timeStamp) {
      const vm = this
      setTimeout(() => {
        vm.message.forEach(function (item, index) {
          if (item.timeStamp === timeStamp) {
            vm.message.splice(index, 1)
          }
        })
      }, 5000)
    }
  },
  created () {
    const vm = this
    vm.$bus.$on('pushMsg', (msg, status) => {
      vm.updateMessage(msg, status)
    })
  },
  beforeDestroy () {
    const vm = this
    vm.$bus.$off('pushMsg')
  }
}
</script>

<style leng="scss">

</style>
