<template>
  <div class="udp-model">
    <div class="scrollbar contpi-boxt">
      <div class="title">
        <span>{{$t('Set the DAG diagram name')}}</span>
      </div>

      <div>
        <x-input
                type="text"
                v-model="name"
                :disabled="router.history.current.name === 'projects-instance-details'"
                :placeholder="$t('Please enter name(required)')">
        </x-input>
      </div>

      <template v-if="router.history.current.name !== 'projects-instance-details'">
        <div style="padding-top: 12px;">
          <x-input
                  type="textarea"
                  v-model="desc"
                  :autosize="{minRows:2}"
                  :placeholder="$t('Please enter description(optional)')"
                  autocomplete="off">
          </x-input>
        </div>
      </template>

      <div class="title" style="padding-top: 6px;">
        <span>超时告警</span>
        <span style="padding-left: 6px;">
          <x-switch v-model="checkedTimeout"></x-switch>
        </span>
      </div>
      <div class="content" style="padding-bottom: 10px;" v-if="checkedTimeout">
        <span>
          <x-input v-model="timeout" style="width: 160px;" maxlength="9">
            <span slot="append">{{$t('Minute')}}</span>
          </x-input>
        </span>
      </div>

      <div class="title" style="padding-top: 6px;">
        <span>{{$t('Set global')}}</span>
      </div>
      <div class="content">
        <div>
          <m-local-params
                  ref="refLocalParams"
                  @on-local-params="_onLocalParams"
                  :udp-list="udpList"
                  :hide="false">
          </m-local-params>
        </div>
      </div>
    </div>
    <div class="bottom">
      <div class="submit">
        <template v-if="router.history.current.name === 'projects-instance-details'">
          <div class="lint-pt">
            <x-checkbox v-model="syncDefine">{{$t('Whether to update the process definition')}}</x-checkbox>
          </div>
        </template>
        <x-button type="text" @click="close()"> {{$t('Cancel')}} </x-button>
        <x-button type="primary" shape="circle" @click="ok()" v-ps="['GENERAL_USER']" >{{$t('Add')}}</x-button>
      </div>
    </div>
  </div>
</template>
<script>
  import _ from 'lodash'
  import i18n from '@/module/i18n'
  import mLocalParams from '../formModel/tasks/_source/localParams'
  import disabledState from '@/module/mixin/disabledState'
  import Affirm from '../jumpAffirm'

  export default {
    name: 'udp',
    data () {
      return {
        // dag name
        name: '',
        // dag desc
        desc: '',
        // Global custom parameters
        udpList: [],
        // Whether to update the process definition
        syncDefine: true,
        // Timeout alarm
        timeout: 0,
        // checked Timeout alarm
        checkedTimeout: true
      }
    },
    mixins: [disabledState],
    props: {
    },
    methods: {
      /**
       * udp data
       */
      _onLocalParams (a) {
        this.udpList = a
      },
      _verifTimeout () {
        const reg = /^[1-9]\d*$/
        if (!reg.test(this.timeout)) {
          this.$message.warning(`${i18n.$t('Please enter a positive integer greater than 0')}`)
          return false
        }
        return true
      },
      /**
       * submit
       */
      ok () {
        if (!this.name) {
          this.$message.warning(`${i18n.$t('DAG graph name cannot be empty')}`)
          return
        }

        let _verif = () => {
          // verification udf
          if (!this.$refs.refLocalParams._verifProp()) {
            return
          }
          // verification timeout
          if (this.checkedTimeout && !this._verifTimeout()) {
            return
          }

          // Storage global globalParams
          this.store.commit('dag/setGlobalParams', _.cloneDeep(this.udpList))
          this.store.commit('dag/setName', _.cloneDeep(this.name))
          this.store.commit('dag/setTimeout', _.cloneDeep(this.timeout))
          this.store.commit('dag/setDesc', _.cloneDeep(this.desc))
          this.store.commit('dag/setSyncDefine', this.syncDefine)
          Affirm.setIsPop(false)
          this.$emit('onUdp')
        }

        // Edit => direct storage
        if (this.store.state.dag.name) {
          _verif()
        } else {
          // New First verify that the name exists
          this.store.dispatch('dag/verifDAGName', this.name).then(res => {
            _verif()
          }).catch(e => {
            this.$message.error(e.msg || '')
          })
        }
      },
      /**
       * Close the popup
       */
      close () {
        this.$emit('close')
      }
    },
    watch: {
      checkedTimeout (val) {
        if (!val) {
          this.timeout = 0
          this.store.commit('dag/setTimeout', _.cloneDeep(this.timeout))
        }
      }
    },
    created () {
      this.udpList = this.store.state.dag.globalParams
      this.name = this.store.state.dag.name
      this.desc = this.store.state.dag.desc
      this.syncDefine = this.store.state.dag.syncDefine
      this.timeout = this.store.state.dag.timeout || 0
      this.checkedTimeout = this.timeout !== 0
    },
    mounted () {},
    components: { mLocalParams }
  }
</script>

<style lang="scss" rel="stylesheet/scss">
  .udp-model {
    width: 624px;
    min-height: 420px;
    background: #fff;
    border-radius: 3px;
    padding:20px 0 ;
    position: relative;
    .contpi-boxt {
      max-height: 600px;
      overflow-y: scroll;
      padding:0 20px;
    }
    .title {
      line-height: 36px;
      padding-bottom: 10px;
      span {
        font-size: 16px;
        color: #333;
      }
    }
    .bottom{
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      text-align: right;
      height: 56px;
      line-height: 56px;
      border-top: 1px solid #DCDEDC;
      background: #fff;
      .submit {
        padding-right: 20px;
        margin-top: -4px;
      }
      .lint-pt {
        position: absolute;
        left: 20px;
        top: -2px;
        >label {
          font-weight: normal;
        }
      }
    }
    .content {
      padding-bottom: 50px;
      .user-def-params-model {
        .add {
          a {
            color: #0097e0;
          }
        }
      }
    }

  }
</style>
