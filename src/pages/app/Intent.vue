<!--
  -  Copyright 2022 Huawei Technologies Co., Ltd.
  -
  -  Licensed under the Apache License, Version 2.0 (the "License");
  -  you may not use this file except in compliance with the License.
  -  You may obtain a copy of the License at
  -
  -      http://www.apache.org/licenses/LICENSE-2.0
  -
  -  Unless required by applicable law or agreed to in writing, software
  -  distributed under the License is distributed on an "AS IS" BASIS,
  -  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  -  See the License for the specific language governing permissions and
  -  limitations under the License.
  -->

<template>
  <div>
    <el-row
      v-if="currentPage === 0"
    >
      <el-form
        label-width="auto"
        :inline="true"
      >
        <el-form-item
          :label="$t('system.edgeNodes.type')"
          style="width:100%;"
        >
          <el-checkbox-group
            v-model="checkedTypes"
          >
            <el-checkbox
              v-for="match in matchType"
              :key="match.label"
              :label="match.label"
              @change="selectType($event,match.label)"
            >
              {{ match.label === 'matchAll' ? $t('system.edgeNodes.matchAll') : $t('system.edgeNodes.matchAny') }}
              <el-tooltip
                class="item"
                effect="dark"
                :content="match.label === 'matchAll' ? $t('system.edgeNodes.matchAllCriteria') : $t('system.edgeNodes.matchAnyCriteria')"
                placement="top-start"
              >
                <em class="el-icon-info" />
              </el-tooltip>
            </el-checkbox>
          </el-checkbox-group>
        </el-form-item>
      </el-form>
    </el-row>
    <el-row
      v-else
    >
      <el-row style="margin-top: -25px; margin-bottom: 10px">
        <span style="font-size: 18px;font-weight: bold; color: #5844be;margin-left: 60px;"> {{ $t('system.edgeNodes')[getTitle()] }} </span>
      </el-row>
      <el-row
        v-if="getTitle() === 'matchAll'"
      >
        <el-form
          label-width="auto"
          :inline="true"
        >
          <el-form-item
            v-for="(value, name, index) in intenPropObjAll"
            :key="index"
            :label="$t('system.edgeNodes')[name]"
            style="width:100%;"
          >
            <el-input :value="value" />
          </el-form-item>
          <el-form-item
            :label="$t('system.edgeNodes.addProperties')"
          >
            <el-select
              v-model="custProps.selProp"
              :placeholder="$t('system.edgeNodes.selProperties')"
            >
              <el-option
                v-for="item in intentProps"
                :key="item.value"
                :label="$t('system.edgeNodes')[item.value]"
                :value="item.value"
                :disabled="getDisabled(item.value)"
              />
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-input
              :placeholder="$t('system.edgeNodes.entVal')"
              v-model="custProps.selPropValue"
            />
          </el-form-item>
          <el-form-item>
            <el-button
              type="primary"
              size="small"
              :disabled="isEmpty(custProps.selPropValue)"
              @click="addIntentProperties"
              class="addButton"
            >
              {{ $t('system.edgeNodes.add') }}
            </el-button>
          </el-form-item>
        </el-form>
      </el-row>
      <el-row
        v-if="getTitle() === 'matchAny'"
      >
        <el-form
          label-width="auto"
          :inline="true"
        >
          <el-form-item
            v-for="(value, name, index) in intenPropObjAny"
            :key="index"
            :label="$t(name)"
            style="width:100%;"
          >
            <el-input :value="value" />
          </el-form-item>
          <el-form-item
            :label="$t('system.edgeNodes.addProperties')"
          >
            <el-select
              v-model="custProps.selProp"
              :placeholder="$t('system.edgeNodes.selProperties')"
            >
              <el-option
                v-for="item in intentProps"
                :key="item.value"
                :label="$t('system.edgeNodes')[item.value]"
                :value="item.value"
                :disabled="getDisabled(item.value)"
              />
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-input
              :placeholder="$t('system.edgeNodes.entVal')"
              v-model="custProps.selPropValue"
            />
          </el-form-item>
          <el-form-item>
            <el-button
              type="primary"
              size="small"
              :disabled="isEmpty(custProps.selPropValue)"
              @click="addIntentProperties"
              class="addButton"
            >
              {{ $t('system.edgeNodes.add') }}
            </el-button>
          </el-form-item>
        </el-form>
      </el-row>
    </el-row>
    <div
      slot="footer"
      class="dialog-footer"
    >
      <el-button
        id="confirmBtn"
        type="primary"
        size="small"
        @click="confirm"
        v-if="currentPage === pageCount && currentPage !== 0"
      >
        {{ $t('common.confirm') }}
      </el-button>
      <el-button
        id="cancelBtn"
        size="small"
        @click="cancel"
      >
        {{ $t('common.cancel') }}
      </el-button>
      <el-button
        id="confirmBtn"
        type="primary"
        size="small"
        @click="back()"
        v-if="currentPage != 0"
      >
        {{ $t('system.edgeNodes.back') }}
      </el-button>
      <el-button
        id="nextmBtn"
        type="primary"
        size="small"
        @click="next()"
        v-if="currentPage != pageCount || currentPage === 0"
      >
        {{ $t('system.edgeNodes.next') }}
      </el-button>
    </div>
  </div>
</template>

<script>
function initialState () {
  return {
    matchType: [
      {
        label: 'matchAll'
      },
      {
        label: 'matchAny'
      }
    ],
    intentProps: [
      {
        value: 'mechostIp'
      },
      {
        value: 'mechostName'
      },
      {
        value: 'zipCode'
      },
      {
        value: 'city'
      },
      {
        value: 'address'
      },
      {
        value: 'affinity'
      },
      {
        value: 'cpuusage'
      },
      {
        value: 'memusage'
      },
      {
        value: 'vim'
      },
      {
        value: 'coordinates'
      }
    ],
    custProps: {},
    intenPropObj: {},
    intenPropObjAll: {},
    intenPropObjAny: {},
    hwcapabilities: [
      {
        hwType: '',
        hwVendor: '',
        hwModel: ''
      }
    ],
    swcapabilities: [
      {
        capabilityName: ''
      }
    ],
    isHWCapabilityEnable: false,
    isSWCapabilityEnable: false,
    pageCount: 1,
    currentPage: 0,
    checkedTypes: ['matchAll'],
    addedHWCap: '',
    addedSWCap: '',
    hwCapability: {},
    swCapability: {}
  }
}

export default {
  name: 'Intent',
  props: {
    cancel: {
      default: function () {
        return null
      },
      type: Function
    },
    confirmIntent: {
      default: function () {
        return null
      },
      type: Function
    }
  },
  data () {
    return initialState()
  },
  mounted () {
    Object.assign(this.$data, this.$options.data.call(this))
  },
  methods: {
    getDisabled (val) {
      if (this.intenPropObj[val]) {
        return true
      }
      return false
    },
    confirm () {
      let intentObj = {}
      if (Object.keys(this.intenPropObjAll).length > 0) {
        intentObj['matchAll'] = JSON.parse(JSON.stringify(this.intenPropObjAll))
      }
      if (Object.keys(this.intenPropObjAny).length > 0) {
        intentObj['matchAny'] = JSON.parse(JSON.stringify(this.intenPropObjAny))
      }
      if (Object.keys(intentObj).length > 0) {
        this.confirmIntent(intentObj)
      } else {
        this.$message.error(this.$t('system.edgeNodes.intentInfo'))
      }
    },
    getTitle () {
      let type = ''
      if ((this.currentPage === 1 && this.checkedTypes.length === 2) || (this.checkedTypes.length === 1 && this.checkedTypes.indexOf('matchAll') > -1)) {
        type = 'matchAll'
      } else if ((this.currentPage === 2 && this.checkedTypes.length === 2) || (this.checkedTypes.length === 1 && this.checkedTypes.indexOf('matchAny') > -1)) {
        type = 'matchAny'
      }
      return type
    },
    back () {
      this.currentPage--
    },
    next () {
      if (this.checkedTypes.length === 0) {
        this.$message.error('Please select type to proceed further.')
        return
      }
      this.currentPage++
    },
    selectType (evt, type) {
      this.pageCount = this.checkedTypes.length
      if (type === 'matchAll' && !evt) {
        this.removeProps(this.intenPropObjAll)
        this.intenPropObjAll = {}
      }
      if (type === 'matchAny' && !evt) {
        this.removeProps(this.intenPropObjAny)
        this.intenPropObjAny = {}
      }
    },
    removeProps (obj) {
      for (let i in obj) {
        delete this.intenPropObj[i]
      }
    },
    deleteRow (index, rows, cap) {
      if (cap === 'hw') {
        this.hwcapabilities.splice(index, 1)
        if (this.hwcapabilities.length === 0) {
          let hwCap = {
            hwType: '',
            hwVendor: '',
            hwModel: ''
          }
          this.hwcapabilities = [hwCap]
        }
      } else {
        this.swcapabilities.splice(index, 1)
        if (this.swcapabilities.length === 0) {
          let swCap = {
            capabilityName: ''
          }
          this.swcapabilities = [swCap]
        }
      }
    },
    saveRow (index, rows, cap) {
      if (cap === 'hw') {
        let hwCap = {
          hwType: '',
          hwVendor: '',
          hwModel: ''
        }
        this.hwCapability = this.hwcapabilities
        this.hwcapabilities = [...this.hwcapabilities, hwCap]
        this.addedHWCap = this.getTitle()
      } else {
        let swCap = {
          capabilityName: ''
        }
        this.swCapability = this.swcapabilities
        this.swcapabilities = [...this.swcapabilities, swCap]
        this.addedSWCap = this.getTitle()
      }
    },
    isEmptyRow (val) {
      return (!val || val.length === 0)
    },
    isEmpty (val) {
      return (!this.custProps.selProp || this.custProps.selProp.length === 0) || (!val || val.length === 0)
    },
    // Add intent Properties
    addIntentProperties () {
      if (this.getTitle() === 'matchAll') {
        this.intenPropObjAll[this.custProps.selProp] = this.custProps.selPropValue
      } else if (this.getTitle() === 'matchAny') {
        this.intenPropObjAny[this.custProps.selProp] = this.custProps.selPropValue
      }
      this.intenPropObj[this.custProps.selProp] = this.custProps.selPropValue
      this.custProps = {}
    }
  }
}
</script>

<style lang='less' scoped>
.addButton {
  background: rgb(88, 68, 190);
  color: rgb(255, 255, 255);
  padding: 5px 15px;
  height: 30px;
  margin-top: 5px;
}
.intentLbl {
  margin-top: 10px;
  margin-right: 5px;
  text-align:right;
}
.capabilityForm .el-form-item__content {
  width: 80% !important;
}
</style>
