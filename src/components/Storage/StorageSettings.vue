<!--
 * @Author: zhanghengxin hengxin.zhang@icewhale.org
 * @Date:  2022-09-13 17:01:37
 * @LastEditors: zhanghengxin ezreal.ice@icloud.com
 * @LastEditTime: 2022-09-29 20:35:39
 * @FilePath: /CasaOS-UI/src/components/Storage/StorageSettings.vue
 * @Description:
 *
 * Copyright (c) 2022 by IceWhale, All Rights Reserved.
-->
<template>
  <div class="modal-card">
    <!--    head-->
    <header :class="{'pri-head':currentStep<=1}" class="modal-card-head ">
      <div class="is-flex-grow-1">
        <h3 class=" title is-3">{{ $t(title) }}</h3>
      </div>
      <button class="delete" type="button" @click="$emit('close')"/>
    </header>
    <!--remind-->
    <div v-if="currentStep === 0" class="pri-mrl-2rem mt-5 mb-4 font">
      {{ $t('All the checked Storage will be merged into CasaOS HD.') }}
    </div>

    <section v-if="currentStep === 0"
             class="notification is-overlay mb-4 pri-mrl-2rem pr-0 pl-0 radius">
      <div v-for="(item, index) in storageData" :key="item.path+index" class="is-flex pri-mtr-3px ml-4 mr-4">
        <div class="ml-4 mr-4 is-flex is-align-items-center">
          <b-image :src="require('@/assets/img/storage/storage.png')" class="is-32x32"></b-image>
        </div>
        <div class="is-flex is-flex-grow-1 is-flex-direction-column is-justify-content-center ">
          <span class="is-uppercase one-line is-size-14px">{{ item.name || $t('undefined') }}</span>
        </div>
        <div class="is-flex is-flex-shrink-0 is-flex-direction-column is-justify-content-center mr-4">
          <span class="is-uppercase is-size-7 pri-text-color">{{
              renderSize(item.size - item.availSize)
            }}/{{
              renderSize(item.size)
            }}</span>
        </div>
        <b-checkbox v-model="checkBoxGroup" :disabled="item.persistedIn !== 'casaos'" :native-value="item.path"
                    class="mr-4"></b-checkbox>
      </div>
      <div v-for="(item, index) in storageMissData" :key="item.path+index" class="is-flex pri-mtr-3px ml-4 mr-4">
        <div class="ml-4 mr-4 is-flex is-align-items-center">
          <b-image :src="require('@/assets/img/storage/storage.png')" class="is-32x32"></b-image>
        </div>
        <div class="is-flex is-flex-grow-1 is-flex-direction-column is-justify-content-center ">
          <span class="is-uppercase one-line is-size-14px">{{ item.name || $t('undefined') }}</span>
        </div>
        <div class="is-flex is-flex-shrink-0 is-flex-direction-column is-justify-content-center mr-4">
          <span class="is-flex is-align-items-center has-text-danger small-font">
            <b-icon icon="danger" pack="casa" class="warn" custom-size="casa-16px"></b-icon>
            {{ $t('Missing') }}
          </span>
        </div>
        <b-checkbox v-model="checkBoxMissGroup" :native-value="item.path" class="mr-4"></b-checkbox>
      </div>
    </section>
    <section v-if="currentStep > 0"
             class="notification is-overlay non-backgroud mb-4 pri-mrl-2rem pr-0 pl-0">
      <template v-if="currentStep === 1">
        <div class="font">
          {{ $t('Enter the password to continue.') }}
        </div>
        <b-input ref="inputPassword" v-model="password" class="mt-4" type="password"
                 @keyup.enter.native="verifyPassword(password)"></b-input>
      </template>
      <div v-if="currentStep === 2" class="is-flex is-align-items-center font">
        <div class="message-danger left mr-2 is-flex is-align-items-center">
          <b-icon class="is-38x38" custom-size="is-size-2" icon="danger" pack="casa"></b-icon>
        </div>
        {{
          `${runName.split(',').length} ` + $t('APPs is running') + ` (${runName}), ` + $t('restart APPs to continue.')
        }}
      </div>
      <div v-if="currentStep === 3" class="is-flex is-align-items-center font">
        <div class="message-danger left mr-2 is-flex is-align-items-center">
          <b-icon class="is-38x38" custom-size="is-size-2" icon="danger" pack="casa"></b-icon>
        </div>
        {{ runName + $t(' is running, restart ') + runName + $t(' to continue.') }}
      </div>
    </section>
    <div v-if="currentStep === 0 && checkBoxGroup.length > 0" class="message-alert is-flex is-align-items-center font">
      <div class="is-flex left ml-4 mr-2 is-align-items-center">
        <b-icon class="is-16x16" icon="danger" pack="casa" custom-size="casa-19px"></b-icon>
      </div>
      {{ $t('If the chosen storage is not empty, format better first.') }}
    </div>

    <div v-if="currentStep === 0 && isSplit" class="pri-message-danger is-flex is-align-items-center font">
      <div class="is-flex left ml-4 mr-2 is-align-items-center">
        <b-icon icon="danger" pack="casa" custom-size="casa-19px"></b-icon>
      </div>
      {{ $t('Please back up your data in storage, otherwise the data may be lost.') }}
    </div>

    <footer :class="{'t-line': currentStep <= 1}" class="modal-card-foot is-flex is-align-items-center ">
      <div class="is-flex-grow-1"></div>
      <div class="mr-4">
        <b-button v-show="currentStep > 1" :label="$t('Cancel')" expaned rounded
                  @click="currentStep = 0"/>
      </div>
      <div>
        <b-button v-show="currentStep === 0" :label="$t(affirm)" expaned rounded
                  type="is-primary" @click="submit"/>
        <b-button v-show="currentStep === 1" :label="$t(affirm)" expaned rounded
                  type="is-primary" @click="verifyPassword(password)"/>
        <b-button v-show="currentStep === 2" :label="$t(affirm)" :loading="isConnecting" expaned rounded
                  type="is-primary" @click="restart"/>
        <b-button v-show="currentStep === 3" :label="$t(affirm)" :loading="isConnecting" expaned rounded
                  type="is-primary" @click="restart"/>
      </div>
    </footer>
  </div>
</template>

<script>
import {mixin} from "@/mixins/mixin";
import jwt_decode from "jwt-decode";
import MD5 from 'md5-es';
import events from '@/events/events';

export default {
  name: "StorageSettings",
  mixins: [mixin],
  async mounted() {
    // TODO: the part is repetition
    //  with APPs Installation Location requirement document
    // 获取merge信息
    try {
      this.mergeStorageList = await this.$api.local_storage.getMergerfsInfo().then((res) => res.data.data[0]['source_volume_paths'])
    } catch (e) {
      this.mergeStorageList = []
      console.log(e)
    }
    this.checkBoxGroup.push(...this.mergeStorageList)
    // this.checkBoxMissGroup.push(...this.mergeStorageList)
    this.getDiskList();
    // this.getMerageStorage();
  },
  watch: {
    // 0 default :mainstorage settings
    // 1 Data Protected
    // 2 APPs Restart
    // 3 APP Restart
    currentStep(val) {
      switch (val) {
        case 0:
          this.title = "Merge Storages";
          this.affirm = "Submit";
          break;
        case 1:
          this.title = "Data Protected";
          this.affirm = "Submit";
          this.$nextTick(() => {
            this.$refs.inputPassword.focus();
          });
          break;
        case 2:
          this.title = "APPs Restart";
          this.affirm = "Restart";
          break;
        case 3:
          this.title = "APP Restart";
          this.affirm = "Restart";
          break;
        default:
          break;
      }

    }
  },
  computed: {
    extended() {
      return this.checkBoxGroup.join(":")
    },
    isSplit() {
      return !this.mergeStorageList.every(item => this.checkBoxGroup.includes(item) || this.storageMissData.includes(item))
    }
  },
  data() {
    return {
      storageData: [],
      storageMissData: [],
      diskData: {},
      unDiskData: {},
      checkBoxGroup: [],
      checkBoxMissGroup: [],
      isConnecting: false,
      currentStep: 0,
      title: "Merge Storages",
      affirm: "Submit",
      mergeInfo: [],
      password: '',
      runName: '',
      mergeStorageList: [],
    }
  }
  ,
  methods: {
    /**
     * @description: Get disk list
     * @param {}
     * @return {void}
     * src/components/Storage/StorageManagerPanel.vue:234
     */
    async getDiskList() {
      // get storage list
      // TODO: the part is repetition
      //  with APPs Installation Location requirement document
      const storageRes = await this.$api.storage.list()
      const storageArray = []
      const storageMissArray = []
      let testMergeMiss = this.mergeStorageList
      storageRes.data.data.forEach(item => {
        item.children.forEach(part => {
          part.disk = item.path
          part.diskName = item.disk_name
          storageArray.push(part)
          testMergeMiss = testMergeMiss.filter(v => v !== part.path)
        })
      })
      this.checkBoxMissGroup.push(...testMergeMiss);
      testMergeMiss.forEach(item => {
        storageMissArray.push({
          "mount_point": "",
          "size": "",
          "avail": "",
          "type": "",
          "path": item,
          "drive_name": "",
          "label": "",
          "persisted_in": "",
          "disk": "",
          "diskName": ""
        })
      })

      this.storageData = storageArray.map((storage) => {
        return {
          name: storage.label,
          isSystem: storage.diskName == "System",
          fsType: storage.type,
          size: storage.size,
          availSize: storage.avail,
          usePercent: 100 - Math.floor(storage.avail * 100 / storage.size),
          diskName: storage.drive_name,
          path: storage.path,
          mount_point: storage.mount_point,
          disk: storage.disk,
          persistedIn: storage.persisted_in,
        }
      })

      this.storageMissData = storageMissArray.map((storage) => {
        return {
          name: storage.label,
          isSystem: storage.diskName == "System",
          fsType: storage.type,
          size: storage.size,
          availSize: storage.avail,
          usePercent: 100 - Math.floor(storage.avail * 100 / storage.size),
          diskName: storage.drive_name,
          path: storage.path,
          mount_point: storage.mount_point,
          disk: storage.disk,
          persistedIn: storage.persisted_in,
        }
      })
    },

    updateMerge(dockerInfo) {
      // merge api
      this.$api.local_storage.updateMergerfsInfo({
        "fstype": "fuse.mergerfs",
        "mount_point": "/DATA",
        // "source_base_path": "/var/lib/casaos/files",
        "source_volume_paths": [
          ...this.checkBoxGroup, ...this.checkBoxMissGroup
        ]
      }).then(res => {
        Promise.all(dockerInfo.map(async item => {
          await this.$api.container.updateState(item.id, "start")
        })).catch(e => {
          this.$buefy.toast.open({
            message: e.response.data.data,
            type: "is-danger",
            position: "is-bottom-right",
            duration: 3000,
          });
        })
        // TODO : need to check the result by the states code
        switch (res.status) {
          case 200:
          case 400:
          default:
            // refresh local storage
            this.$EventBus.$emit(events.RELOAD_MOUNT_LIST)
            // close the modal
            this.$emit('close')
        }
      })
    }
    ,

    // get the storage list be mounted of mergerfs
    async getMerageStorage() {
      try {
        // TODO merge can't be used
        this.mergeInfo = await this.$api.local_storage.getMergerfsInfo().then(res => res.data.data[0]["source_volume_paths"]).catch(() => [])
        this.checkBoxGroup.push(...this.mergeInfo)
      } catch (e) {
        console.log(e)
      }
    },

    async submit(e, nextStep = false) {
      // operation : split the mergerfs
      let notSplit = this.mergeStorageList.every(item => this.checkBoxGroup.includes(item) || this.checkBoxMissGroup.includes(item))
      if (notSplit || nextStep) {
        // get docker info
        let dockerInfo = await this.$api.container.getInfo('').then(res => res.data.data.casaos_apps)
        if (dockerInfo.length === 1) {
          this.currentStep = 3
          this.runName = dockerInfo[0].name
          return
        } else if (dockerInfo.length > 1) {
          this.currentStep = 2
          this.runName = dockerInfo.map(item => item.name).join(',')
          return
        } else {
          this.restart()
          return
        }
      }
      this.currentStep = 1
    },

    async restart() {
      this.isConnecting = true
      try {
        // get docker info
        let dockerInfo = await this.$api.container.getInfo('').then(res => res.data.data.casaos_apps)
        let container = this.$api.container
        Promise.all(dockerInfo.map(async item => {
          await container.updateState(item.id, "stop")
        })).then(() => {
          this.updateMerge(dockerInfo)
          this.isConnecting = false
        }).catch((e) => {
          this.isConnecting = false
          this.$buefy.toast.open({
            duration: 5000,
            message: e.response.data.data,
            type: 'is-danger'
          })
        })
      } catch (e) {
        console.log(e)
      }

    },

    verifyPassword(password) {
      // get token from the local storage
      const token = localStorage.getItem('access_token')
      // decode the token
      const tokenJson = jwt_decode(token)
      if (MD5.hash(password) === tokenJson.password) {
        this.submit(null, true);
        return
      }
      this.$buefy.toast.open({
        duration: 3000,
        message: this.$t("Password is incorrect"),
        type: 'is-danger'
      })
    }

  }
  ,
}
</script>

<style lang="scss" scped>
.non-backgroud {
  background: none;
}

.pri-head {
  line-height: 1.875rem;
  border-bottom: rgba(0, 0, 0, 0.1) 1px solid !important;
}

.pri-margin {
  margin: 2rem;
}

.pri-mrl-2rem {
  margin-left: 2rem;
  margin-right: 2rem;

  .pri-mtr-3px {
    margin-top: 0.1875rem;
    margin-bottom: 0.1875rem;
    min-height: 2.75rem;
    border-radius: 0.25rem;
  }

  div:hover {
    background: hsla(215, 89%, 93%, 1);
  }
}

.font {
  font-family: 'Roboto';
  font-style: normal;
  font-weight: 400;
  font-size: 14px;
  line-height: 21px;
}

.small-font {
  font-family: Roboto;
  font-size: 12px;
  font-weight: 400;
  line-height: 18px;
  letter-spacing: 0em;
}

.radius {
  box-sizing: border-box;
  background: #F8F8F8;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 0.75rem;
}

.pri-text-color {
  color: hsla(0, 0%, 0%, 0.4);
}

.message-danger {
  color: hsla(348, 86%, 61%, 1);
}

.message-alert {
  height: 2rem;
  margin-top: 0rem;
  margin-bottom: 1rem;
  margin-left: 2rem;
  margin-right: 2rem;
  border-radius: 0.25rem;
  color: hsla(40, 100%, 43%, 1);
  font-size: 0.875rem;
  font-style: normal;
  background: hsla(40, 100%, 95%, 1);
}

.pri-message-danger {
  height: 2rem;
  margin-top: 0rem;
  margin-bottom: 1rem;
  margin-left: 2rem;
  margin-right: 2rem;
  border-radius: 0.25rem;
  color: hsla(348, 86%, 61%, 1);
  font-size: 0.875rem;
  font-style: normal;
  background: hsla(348, 100%, 95%, 1);
}

.t-line {
  border-top: rgba(0, 0, 0, 0.1) 1px solid !important;
}

.warn {
  color: hsla(348, 86%, 61%, 1);
}

.is-38x38 {
  width: 2.375rem;
  height: 2.375rem;
}
</style>
<style lang="scss">
.pri-mtr-3px .control-label {
  display: none;
}

</style>
