<!--
 * @Author: Jerryk jerry@icewhale.org
 * @Date: 2022-05-30 20:15:05
 * @LastEditors: Jerryk jerry@icewhale.org
 * @LastEditTime: 2022-07-14 12:13:15
 * @FilePath: \CasaOS-UI\src\components\filebrowser\components\OperationStatusBar.vue
 * @Description: 
 * 
 * Copyright (c) 2022 by IceWhale, All Rights Reserved. 
-->
<template>
  <div class="mr-2 operation-status-bar" v-if="isShow">
    <popper trigger="clickToOpen" enter-active-class="animated fadeIn" leave-active-class=" fadeOut" :options="{
      placement: 'top',
      modifiers: { offset: { offset: '0,5px' } }
    }">
      <div class="popper">
        <div class="is-flex is-align-items-center">
          <div class=" is-flex-grow-1">
            <p class="card-header-title mb-0">{{$t('Current Tasks')}}</p>
          </div>
          <div class=" is-flex-shrink-0 mr-2">
            <b-button size="is-small" type="is-primary is-light" :label="$t('cancel-all')" class="mr-2" @click="cancel" :loading="isLoading" rounded />
          </div>
        </div>

        <operation-status-item v-for="task in taskList" :key="task.id" :data="task"></operation-status-item>
      </div>
      <b-button slot="reference" size="is-small" type="is-primary is-light" rounded>
        <b-icon icon="autorenew" custom-class="mdi-spin" custom-size=" mdi-18px"></b-icon>
      </b-button>
    </popper>
  </div>
</template>

<script>
import Popper from 'vue-popperjs';

import 'vue-popperjs/dist/vue-popper.css';
import OperationStatusItem from './OperationStatusItem.vue';
export default {
  name: "operation-status-bar",
  components: {
    Popper,
    OperationStatusItem,
  },
  data() {
    return {
      taskList: [],
      isLoading: false
    }
  },
  computed: {
    isShow() {
      return this.taskList.length > 0
    }
  },
  methods: {
    cancel() {
      this.isLoading = true
      this.$api.batch.deleteTask(0).then(res => {
        if (res.data.success == 200) {
          this.isLoading = false
        }
      })
    }
  },
  sockets: {
    file_operate(data) {
      const taskList = data.body.file_operate.data
      this.taskList = taskList.filter(task => {
        return !task.finished
      })
    }
  }
}
</script>

<style>
</style>