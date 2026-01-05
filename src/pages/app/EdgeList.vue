<!--
  -  Copyright 2020-2021 Huawei Technologies Co., Ltd.
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
    <div class="btnMain">
      <el-button
        type="primary"
        @click="distribute"
      >
        <span>{{ $t('app.packageList.distribute') }}</span>
      </el-button>
    </div>
    <div class="contentList">
      <Search
        :placeholder="$t('tip.fuzzyQuery')"
        @getSearchData="getSearchData"
      />
      <span class="btnSearch">
        <el-button
          type="primary"
          @click="multipleDeploy"
        >
          <span>{{ $t('app.distriList.multipleDeploy') }}</span>
        </el-button>
        <el-button
          type="success"
          @click="testDeployDialog"
          style="margin-left: 10px"
        >
          <span>【测试】打开部署对话框</span>
        </el-button>
      </span>
      <div class="tableDiv">
        <el-table
          class="mt20"
          size="small"
          :data="currPageTableData"
          v-loading="dataLoading"
          @selection-change="handleSelectionChange"
        >
          <el-table-column
            type="selection"
            revers-selection
            width="50"
          />
          <el-table-column
            prop="appPackageName"
            :label="$t('app.packageList.name')"
            width="180"
          >
            <template>
              <div>
                {{ this.appPackageName }}
              </div>
            </template>
          </el-table-column>
          <el-table-column
            prop="appVersion"
            :label="$t('app.packageList.version')"
            width="130"
          >
            <template>
              <div>
                {{ this.appVersion }}
              </div>
            </template>
          </el-table-column>
          <el-table-column
            prop="provider"
            :label="$t('app.packageList.vendor')"
            width="160"
          >
            <template>
              <div>
                {{ this.provider }}
              </div>
            </template>
          </el-table-column>
          <el-table-column
            prop="appAffinity"
            :label="$t('app.packageList.affinity')"
            width="120"
          >
            <template>
              <div>
                {{ this.appAffinity }}
              </div>
            </template>
          </el-table-column>
          <el-table-column
            prop="hostIp"
            :label=" $t('app.distriList.hostIp')"
          />
          <el-table-column
            prop="status"
            :label=" $t('app.distriList.status')"
          >
            <template slot-scope="scope">
              <span
                v-if="scope.row.status === 'Distributed'||scope.row.status === 'uploaded'"
                class="success"
              ><em class="el-icon-success" />{{ scope.row.status }}</span>
              <span
                v-else-if="scope.row.status === 'Processing'"
                class="primary"
              ><em class="el-icon-loading" />{{ scope.row.status }}</span>
              <span
                v-else
                class="error"
              ><em class="el-icon-error" />{{ scope.row.status }}</span>
            </template>
          </el-table-column>
          <el-table-column
            :label="$t('common.operation')"
            align="center"
          >
            <template slot-scope="scope">
              <el-button
                id="deleteBtn"
                @click.native.prevent="beforeDelete(scope.row)"
                type="text"
                size="small"
              >
                {{ $t('common.delete') }}
              </el-button>
              <el-button
                id="deployBtn"
                @click="deploy(scope.row,1)"
                :disabled="scope.row.status !=='Distributed' && scope.row.status !=='uploaded'"
                type="text"
                size="small"
              >
                {{ $t('app.distriList.deploy') }}
              </el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="pageBar">
          <Pagination
            :page-sizes="[10,15,20,25]"
            :table-data="paginationData"
            @getCurrentPageData="getCurrentPageData"
          />
        </div>
      </div>
      <el-dialog
        :show-close="false"
        :visible.sync="customDialogVisible"
        width="45%"
        class="deploy-dialog"
      >
        <CustomResource
          :cancel="cancelCustomDialog"
          :confirm="checkAvailability"
          :custom-resource-data="customResourceData"
        />
      </el-dialog>
      <el-dialog
        :show-close="false"
        :visible.sync="dialogVisible"
        width="55%"
        class="deploy-dialog"
      >
        <div class="secondLabel">
          {{ $t('app.distriList.deploymentConf') }}
        </div>
        <el-form
          label-width="auto"
          class="configForm"
          :model="configForm"
          ref="configForm"
          :rules="rules"
        >
          <p class="title">
            {{ $t('app.packageList.hostInfo') }}
          </p>
          <el-form-item
            :label="$t('app.packageList.ip')"
            label-width="140px"
          >
            <div
              v-for="(item,index) in hostList"
              :key="index"
              class="hostinfo"
            >
              <span
                class="hostip"
              >{{ item.hostIp }}</span>
              <span
                v-if="item.status === 'Distributed'||item.status === 'uploaded'"
                class="success"
              ><em class="el-icon-success" />{{ item.status }}</span>
            </div>
          </el-form-item>
          <!-- [新增] 2026-01-05 网络平面选择，移到IP地址下面 -->
          <el-form-item
            :label="$t('app.distriList.networkPlane')"
            prop="selectedNetworkPlane"
            label-width="140px"
          >
            <el-select
              v-model="configForm.selectedNetworkPlane"
              :placeholder="$t('app.distriList.selectNetworkPlane')"
              :loading="isLoadingNetworkPlanes"
              style="width: 100%"
            >
              <el-option
                v-for="option in networkPlaneOptions"
                :key="option.value"
                :label="option.label"
                :value="option.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item
            :label="$t('app.distriList.appName')"
            prop="appName"
            label-width="140px"
          >
            <el-input
              id="appname"
              maxlength="20"
              v-model="configForm.appName"
            />
          </el-form-item>
          <el-form-item
            :label="$t('app.distriList.appDesc')"
            prop="appInstanceDescription"
            label-width="140px"
          >
            <el-input
              id="appdesc"
              maxlength="120"
              v-model="configForm.appInstanceDescription"
            />
          </el-form-item>
          <el-form-item
            :label="$t('system.edgeNodes.hwCapability')"
            prop="hwCapabilities"
            label-width="140px"
          >
            <el-checkbox-group
              v-model="configForm.hwCapabilities"
            >
              <el-checkbox
                v-for="item in capabilities"
                :label="item"
                :key="item"
              >
                {{ item }}
              </el-checkbox>
            </el-checkbox-group>
          </el-form-item>
          <div
            v-if="templateInputs.length>0"
            class="apptemplate"
          >
            <p
              class="title"
            >
              {{ $t('app.packageList.appTemplate') }}
            </p>
            <el-collapse
              v-model="activeNames"
              accordion
            >
              <el-collapse-item
                :title="$t('app.packageList.vmNetCongig')"
                name="1"
              >
                <div
                  v-for="(item,index) in this.VmDataGroup"
                  :key="index"
                  class="vmcofig"
                >
                  <p class="first-title">
                    {{ $t('app.packageList.vmConfig') }}{{ index+1 }}:
                  </p>
                  <el-row>
                    <div
                      v-for="(vmSourceItem) in item.VmResourceData"
                      :key="vmSourceItem.label"
                    >
                      <el-col
                        :span="7"
                      >
                        <el-form-item
                          :label="vmSourceItem.label.substring(5)"
                          class="apptemplate-form"
                          size="small"
                        >
                          <el-input
                            id="podsel"
                            maxlength="30"
                            v-model="vmSourceItem.value"
                            size="small"
                          />
                        </el-form-item>
                      </el-col>
                      <el-col
                        :span="1"
                        class="unit"
                        v-if="vmSourceItem.label.substring(5)==='vCPU'"
                      >
                        U
                      </el-col>
                      <el-col
                        :span="1"
                        class="unit"
                        v-else
                      >
                        GB
                      </el-col>
                    </div>
                  </el-row>
                  <div
                    v-for="(netItem,netIndex) in item.data"
                    :key="netIndex"
                  >
                    <p class="second-title">
                      {{ $t('app.packageList.netWork') }}{{ netIndex+1 }}:
                    </p>
                    <el-col
                      :span="8"
                      v-for="(netLabel,netIndexLabel) in netItem.data"
                      :key="netIndexLabel"
                    >
                      <el-form-item
                        :label="netLabel.label.substring(17)"
                        class="apptemplate-form"
                        size="small"
                      >
                        <el-input
                          id="podsel"
                          maxlength="30"
                          v-model="netLabel.value"
                          size="small"
                        />
                      </el-form-item>
                    </el-col>
                  </div>
                </div>
              </el-collapse-item>
              <el-collapse-item
                :title="$t('app.packageList.netWorkConfig')"
                name="2"
              >
                <div>
                  <div
                    v-for="(item,index) in this.netWorkSetData"
                    :key="index"
                  >
                    <p class="second-title">
                      {{ $t('app.packageList.netConfig') }}{{ index+1 }}:
                    </p>
                    <el-col
                      :span="8"
                      v-for="(netWork,netIndex) in item.data"
                      :key="netIndex"
                    >
                      <el-form-item
                        class="apptemplate-form"
                        :label="handleNetName(netWork.label)"
                        size="small"
                      >
                        <el-input
                          id="podsel"
                          maxlength="30"
                          v-model="netWork.value"
                          size="small"
                        />
                      </el-form-item>
                    </el-col>
                  </div>
                </div>
              </el-collapse-item>
              <el-collapse-item
                :title="$t('app.packageList.otherConfig')"
                name="3"
              >
                <div>
                  <el-row>
                    <el-col
                      :span="8"
                      v-for="(item,index) in otherData"
                      :key="index"
                    >
                      <el-form-item
                        class="apptemplate-form"
                        :label="item.label"
                        size="small"
                      >
                        <el-input
                          id="podsel"
                          maxlength="30"
                          v-model="item.value"
                          size="small"
                        />
                      </el-form-item>
                    </el-col>
                  </el-row>
                </div>
              </el-collapse-item>
            </el-collapse>
          </div>
        </el-form>
        <span
          slot="footer"
          class="dialog-footer"
        >
          <el-button
            id="confirmBtn"
            type="primary"
            size="small"
            @click="confirmToDeploy('configForm')"
            :loading="loading"
          >{{ $t('common.confirm') }}</el-button>
          <el-button
            id="cancelBtn"
            size="small"
            @click="dialogVisible = false,loading=false"
          >{{ $t('common.cancel') }}</el-button>
        </span>
      </el-dialog>

      <!-- 分发 -->
      <el-dialog
        :show-close="false"
        :visible.sync="distributionDialogVisible"
      >
        <div class="secondLabel">
          {{ $t('app.packageList.slectEdgeNodes') }}
        </div>
        <hr>
        <el-row class="el-row-search">
          <el-col
            :span="7"
          >
            <el-checkbox
              v-model="knowEdge"
              style="margin-top: 10px"
            >
              {{ $t('system.edgeNodes.knowEdge') }}
              <el-tooltip
                class="item"
                effect="dark"
                :content="$t('system.edgeNodes.edgeCriteria')"
                placement="top-start"
              >
                <em class="el-icon-info" />
              </el-tooltip>
            </el-checkbox>
          </el-col>
          <el-col
            :span="8"
          >
            <el-input
              id="nodesearch"
              class="el-input-search"
              v-model="edgeNodeSearchInput"
              v-if="knowEdge"
            >
              <em
                slot="suffix"
                class="el-input__icon el-icon-search"
              />
            </el-input>
          </el-col>
          <el-col
            :span="6"
          >
            <span
              v-if="knowEdge"
              class="btnSearch"
              style="top: 0px; right: 0%"
            >
              <el-button
                type="primary"
                @click="getResourceList"
              >
                {{ $t('resource.custResCriteria') }}
              </el-button>
            </span>
          </el-col>
        </el-row>
        <el-row
          v-if="knowEdge"
        >
          <el-row class="el-row-table">
            <el-col :span="24">
              <el-table
                ref="multipleEdgeNodeTable"
                :data="currPageEdgeNodeTableData"
                class="mt20"
                size="small"
                @selection-change="handleEdgeNodeSelectionChange"
              >
                <el-table-column
                  type="selection"
                />
                <el-table-column
                  prop="mechostName"
                  sortable
                  :label="$t('app.packageList.name')"
                />
                <el-table-column
                  prop="mechostIp"
                  :label="$t('app.packageList.ip')"
                />
                <el-table-column
                  prop="city"
                  :label="$t('app.packageList.city')"
                />
                <el-table-column
                  prop="affinity"
                  width="120"
                  :label="$t('app.packageList.affinity')"
                />
                <el-table-column
                  prop="mepmIp"
                  width="120"
                  :label="$t('system.edgeNodes.mepmIp')"
                />
                <el-table-column
                  :label="$t('system.edgeNodes.hwCapability')"
                  width="150"
                >
                  <template slot-scope="scope">
                    <span
                      v-for="(item,index) in scope.row.hwcapabilities"
                      :key="index"
                    >
                      {{ item.hwType }}
                    </span>
                  </template>
                </el-table-column>
                <el-table-column
                  :label="$t('resource.res')"
                  width="150"
                >
                  <template slot-scope="scope">
                    <el-row
                      id="resourcestatus"
                      v-if="resourceData[scope.row.mechostIp]"
                    >
                      <el-col
                        :span="8"
                      >
                        <el-popover
                          trigger="hover"
                          id="resStatics"
                        >
                          <div>
                            {{ $t('resource.total') }}: {{ getResourceInfo(scope.row.mechostIp, 'disk').total }}
                          </div>
                          <div>
                            {{ $t('resource.used') }}: {{ getResourceInfo(scope.row.mechostIp, 'disk').used }}
                          </div>
                          <div>
                            {{ $t('resource.required') }}: {{ getResourceInfo(scope.row.mechostIp, 'disk').required }}
                          </div>
                          <el-progress
                            type="circle"
                            :percentage="getResourceAvailability(scope.row.mechostIp, 'disk')"
                            :color="getColor(scope.row.mechostIp, 'disk')"
                            :width="30"
                            :stroke-width="4"
                            :text-inside="false"
                            style="margin-top:5px; cursor: pointer"
                            slot="reference"
                          />
                          <div
                            style="margin-top:-10px"
                            slot="reference"
                          >
                            Disk
                          </div>
                        </el-popover>
                      </el-col>
                      <el-col
                        :span="8"
                      >
                        <el-popover
                          trigger="hover"
                        >
                          <div>
                            {{ $t('resource.total') }}: {{ getResourceInfo(scope.row.mechostIp, 'mem').total }}
                          </div>
                          <div>
                            {{ $t('resource.used') }}: {{ getResourceInfo(scope.row.mechostIp, 'mem').used }}
                          </div>
                          <div>
                            {{ $t('resource.required') }}: {{ getResourceInfo(scope.row.mechostIp, 'mem').required }}
                          </div>
                          <el-progress
                            type="circle"
                            :percentage="getResourceAvailability(scope.row.mechostIp, 'mem')"
                            :color="getColor(scope.row.mechostIp, 'mem')"
                            :width="30"
                            :stroke-width="4"
                            :text-inside="false"
                            style="margin-top:5px; cursor: pointer"
                            slot="reference"
                          />
                          <div
                            style="margin-top:-10px"
                            slot="reference"
                          >
                            MEM
                          </div>
                        </el-popover>
                      </el-col>
                      <el-col
                        :span="8"
                      >
                        <el-popover
                          trigger="hover"
                        >
                          <div>
                            {{ $t('resource.total') }}: {{ getResourceInfo(scope.row.mechostIp, 'cpu').total }}
                          </div>
                          <div>
                            {{ $t('resource.used') }}: {{ getResourceInfo(scope.row.mechostIp, 'cpu').used }}
                          </div>
                          <div>
                            {{ $t('resource.required') }}: {{ getResourceInfo(scope.row.mechostIp, 'cpu').required }}
                          </div>
                          <el-progress
                            type="circle"
                            :percentage="getResourceAvailability(scope.row.mechostIp, 'cpu')"
                            :color="getColor(scope.row.mechostIp, 'cpu')"
                            :width="30"
                            :stroke-width="4"
                            :text-inside="false"
                            style="margin-top:5px; cursor: pointer"
                            slot="reference"
                          />
                          <div
                            style="margin-top:-10px"
                            slot="reference"
                          >
                            CPU
                          </div>
                        </el-popover>
                      </el-col>
                    </el-row>
                  </template>
                </el-table-column>
              </el-table>
            </el-col>
          </el-row>
          <el-row>
            <el-pagination
              background
              class="pagination rt"
              @size-change="handleEdgeNodePageSizeChange"
              @current-change="handleEdgeNodeCurrentPageChange"
              :current-page="edgeNodeCurrentPage"
              :page-sizes="[5, 10, 15, 20]"
              :page-size="edgeNodePageSize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="edgeNodeTotalNum"
            />
          </el-row>
        </el-row>
        <el-row
          v-else
        >
          <Intent
            :cancel="cancel"
            :confirm-intent="confirmIntent"
          />
        </el-row>
        <div
          slot="footer"
          class="dialog-footer"
          v-if="knowEdge"
        >
          <el-button
            id="confirmBtn"
            type="primary"
            size="small"
            @click="confirm()"
            :loading="loading"
          >
            {{ $t('common.confirm') }}
          </el-button>
          <el-button
            id="cancelBtn"
            size="small"
            @click="cancel()"
          >
            {{ $t('common.cancel') }}
          </el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import Search from '../../components/common/Search.vue'
import Pagination from '../../components/common/Pagination.vue'
import { appo, apm, inventory, resource } from '../../tools/request.js'
import CustomResource from './CustomResource'
import Intent from './Intent.vue'
export default {
  name: 'EdgeList',
  components: {
    Search, Pagination, CustomResource, Intent
  },
  data () {
    return {
      loading: false,
      currPageTableData: [],
      paginationData: [],
      searchVal: '',
      selectData: null,
      selectedData: [],
      appPackageId: '',
      appVersion: '',
      appPackageName: '',
      appAffinity: '',
      provider: '',
      dialogVisible: false,
      customDialogVisible: false,
      configForm: {
        status: '',
        appPackageId: '',
        appName: '',
        appInstanceDescription: '',
        appId: this.appId,
        hwCapabilities: [],
        selectedNetworkPlane: 'default' // [新增] 2026-01-05 选中的网络平面，默认为 default
      },
      dataLoading: true,
      tableData: [],
      packageData: [],
      interval: null,
      instanceId: '',
      timer: null,
      distributionStatus: ['Distributed', 'Error'],
      serchData: null,
      hostList: [],
      // [新增] 2026-01-05 网络平面动态选择功能
      availableNetworkPlanes: {}, // 当前主机可用的网络平面 {"eth1": "n6-net-1", "eth2": "n6-net-2"}
      networkPlaneOptions: [], // 网络平面下拉选项
      isLoadingNetworkPlanes: false, // 加载网络平面数据的状态
      templateInputs: [],
      capabilities: ['GPU', 'NPU'],
      VmDataGroup: [],
      netWorkSetData: [],
      activeNames: '1',
      otherData: [],
      language: localStorage.getItem('language'),
      // 分发
      appId: sessionStorage.getItem('appId'),
      edgeNodesData: [],
      distributionDialogVisible: false,
      packageSearchInput: '',
      edgeNodeSearchInput: '',
      edgeNodeCurrentPage: 1,
      edgeNodePageSize: 5,
      rowSelection: [],
      nodeSelection: [],
      selectedNodeNum: 0,
      currentRowData: '',
      dialogLoading: false,
      knowEdge: true,
      intentObj: {},
      customResourceData: {},
      resourceData: {}
    }
  },
  computed: {
    edgeNodeTotalNum: function () {
      return this.edgeNodesData.length
    },
    totalNum: function () {
      return this.tableData.length
    },
    currPageEdgeNodeTableData: function () {
      return this.edgeNodesData.filter(data => !this.edgeNodeSearchInput || data.mechostName.toLowerCase().includes(this.edgeNodeSearchInput.toLowerCase()))
    },
    rules () {
      return {
        appName: [
          { required: true, message: this.$t('verify.appNameVerify'), trigger: 'blur' },
          { pattern: /^[a-zA-Z0-9]{4,16}$/, message: this.$t('verify.hostNameVerify') }
        ],
        appInstanceDescription: [
          { required: true, message: this.$t('verify.descVerify'), trigger: 'blur' }
        ],
        selectedNetworkPlane: [
          { required: true, message: '请选择网络平面', trigger: 'change' }
        ]
      }
    }
  },
  mounted () {
    this.initList()
    this.interval = setInterval(() => {
      this.initList()
    }, 15000)
  },
  watch: {
    '$i18n.locale': function () {
      let language = localStorage.getItem('language')
      this.language = language
    }
  },
  beforeDestroy () {
    this.clearInterval()
  },
  methods: {
    // [新增] 2026-01-05 测试方法：直接打开部署对话框
    testDeployDialog () {
      this.configForm = {
        status: '',
        appPackageId: 'test-package-id',
        appName: '测试应用',
        appInstanceDescription: '这是一个测试部署',
        appId: 'test-app-id',
        hwCapabilities: [],
        selectedNetworkPlane: 'default'
      }
      this.dialogVisible = true
      // 【选择1】测试真实API调用：取消下面这行的注释，并填入真实的主机IP
      // this.fetchHostNetworkPlanes('192.168.1.100')

      // 【选择2】使用模拟数据演示（当前启用）
      this.availableNetworkPlanes = {
        'eth1': 'n6-net-1',
        'eth2': 'n6-net-2',
        'eht3': 'n6-net-3'
      }
      this.buildNetworkPlaneOptions(this.availableNetworkPlanes)
    },
    async getResourceList () {
      this.customDialogVisible = false
      let param = this.getDistributeParam()
      await resource.getResourceTemplate(param).then(response => {
        let resData = response.data.data
        if (resData && resData.inputs && resData.inputs.length > 0) {
          this.customResourceData = response.data.data
          this.customDialogVisible = true
          this.loading = false
        } else {
          this.$message.warning(this.$t('resource.resDataNotAvailable'))
        }
      }).catch((error) => {
        console.log(error)
      })
    },
    confirmIntent (data) {
      console.log(JSON.stringify(data))
      this.intentObj = data
      this.confirm()
    },
    getColor (hostIp, dt) {
      let data = this.resourceData[hostIp]
      if (data) {
        let newData = data[dt]
        if (newData.remain < newData.required) {
          return '#f56c6c'
        } else if (newData.remain === newData.required) {
          return '#f79468'
        } else if ((((newData.used + newData.required) / newData.total) * 100) > 75) {
          return '#f79468'
        } else {
          return '#5cb87a'
        }
      }
    },
    getResourceAvailability (hostIp, dt) {
      let data = this.resourceData[hostIp]
      if (data) {
        let newData = data[dt]
        return (newData.used / newData.total) * 100
      }
    },
    getResourceInfo (hostIp, dt) {
      let data = this.resourceData[hostIp]
      if (data) {
        return data[dt]
      }
    },
    getDistributeParam () {
      let address = window.location.protocol === 'https:' ? 'https://' : 'http://'
      return {
        appPkgId: this.currentRowData.packageId,
        appId: this.currentRowData.appId,
        appPkgName: this.currentRowData.name,
        appPkgVersion: this.currentRowData.version,
        appPkgDesc: this.currentRowData.shortDesc ? this.currentRowData.shortDesc : 'none',
        appPkgAffinity: this.currentRowData.affinity,
        appPkgPath: address + this.currentRowData.appstoreEndpoint + '/mec/appstore/v1/apps/' + this.currentRowData.appId + '/packages/' + this.currentRowData.packageId + '/action/download',
        appProvider: this.currentRowData.provider,
        mecHostInfo: null,
        createdTime: new Date().toString(),
        modifiedTime: new Date().toString()
      }
    },
    async getResources () {
      let param = this.getDistributeParam()
      await resource.getResources(param).then(response => {
        let resData = []
        if (response.data.data) {
          resData = response.data.data
        }
        this.makeResourceData(resData)
      }).catch((error) => {
        console.log(error)
      })
    },
    cancelCustomDialog () {
      this.customDialogVisible = false
    },
    makeResourceData (data) {
      this.resourceData = {}
      if (data) {
        for (const val of data) {
          this.resourceData[val.edge] = val.resource
        }
      }
    },
    async checkAvailability (data) {
      console.log(JSON.stringify(data))
      await resource.updateResourceTemplate(data).then(response => {
        this.customDialogVisible = false
        console.log(JSON.stringify(response.data.data))
        let resData = []
        if (response.data.data) {
          resData = response.data.data
        }
        this.makeResourceData(resData)
      }).catch((error) => {
        console.log(error)
      })
    },
    clearInterval () {
      clearTimeout(this.interval)
      this.interval = null
      clearTimeout(this.timer)
      this.timer = null
    },
    // 对app表格进行筛选 val：需要查询的值  key: 数据对应的字段
    filterTableData (val) {
      this.paginationData = this.paginationData.filter(item => {
        return Object.keys(item).some(key => {
          return String(item[key]).toLowerCase().indexOf(val) > -1
        })
      })
    },
    // 根据搜索组件进行筛选
    getSearchData (data) {
      this.serchData = data
      this.paginationData = this.tableData
      if (data) {
        this.filterTableData(data.toLowerCase())
      } else {
        this.initList()
      }
    },
    getCurrentPageData (data) {
      this.currPageTableData = data
    },
    multipleDeploy () {
      this.configForm = {
        status: '',
        appPackageId: '',
        appName: '',
        appInstanceDescription: '',
        appId: this.appId,
        hwCapabilities: []
      }
      if (this.selectData !== null && this.selectData.length > 0) {
        let allStatus = []
        this.selectData.forEach(item => {
          allStatus.push(item.status)
        })
        if (!allStatus.includes('Error')) {
          this.deploy(this.selectData, 2)
        } else {
          this.$message.error(this.$t('app.distriList.deleteError'))
        }
      } else {
        this.$message.warning(this.$t('tip.onePackageAtLeast'))
      }
    },
    beforeDelete (rows) {
      this.$confirm(this.$t('tip.beforeDeleteFromMechost'), this.$t('common.warning'), {
        confirmButtonText: this.$t('common.confirm'),
        cancelButtonText: this.$t('common.cancel'),
        closeOnClickModal: false,
        type: 'warning'
      }).then(() => {
        let hostIp = rows.hostIp
        let type = 1
        apm.deleteDistributionApp(type, hostIp, this.appPackageId).then(() => {
          this.showMessage('success', this.$t('tip.deletePacFrmoHost'), 1500)
          this.initList()
        })
      })
    },
    initList () {
      let _status = ['Processing', 'uploading', 'uploaded', 'Distributing']
      apm.getDistributionList().then(res => {
        this.paginationData = []
        res.data.forEach(item => {
          if (item.appId === this.appId) {
            this.appPackageId = item.appPkgId
            this.appPackageName = item.appPkgName
            this.appVersion = item.appPkgVersion
            this.appAffinity = item.appPkgAffinity
            this.provider = item.appProvider
            this.paginationData = item.mecHostInfo
          }
        })
        this.tableData = this.paginationData
        let _num = 0
        this.tableData.forEach(item => {
          if (!_status.includes(item.status)) {
            _num++
            if (_num === this.tableData.length) {
              this.clearInterval()
            }
          }
        })
        if (this.serchData) {
          this.getSearchData(this.serchData)
        }
        this.dataLoading = false
      }).catch((error) => {
        console.log(error)
        this.dataLoading = false
        this.tableData = this.paginationData = []
      })
    },
    deploy (row, type) {
      apm.getApptemplateApi(this.appPackageId).then(res => {
        this.templateInputs = []
        if (res.data.deployType !== 'container') {
          let inputs = res.data.inputs
          inputs.forEach(ele => {
            let obj = {
              label: '',
              value: ''
            }
            obj.label = ele.name
            obj.value = ele.defaultValue
            this.templateInputs.push(obj)
          })
        }
        this.handleInputsData()
        // [修改] 2026-01-05 添加 selectedNetworkPlane 字段
        this.configForm = {
          status: '',
          appPackageId: '',
          appName: '',
          appInstanceDescription: '',
          appId: this.appId,
          hwCapabilities: [],
          selectedNetworkPlane: 'default' // 重置为默认值
        }
        this.hostList = []
        this.configForm.appPackageId = this.appPackageId
        this.configForm.appId = this.appId
        this.dialogVisible = true
        this.$nextTick(() => {
          this.$refs.configForm.resetFields()
        })
        if (type === 2) {
          let array = []
          row.forEach(item => {
            array.push(item.hostIp)
          })
          this.configForm.mecHost = array
          this.hostList = row
          // [新增] 2026-01-05 批量部署时，查询第一个主机的网络平面
          this.fetchHostNetworkPlanes(row[0].hostIp)
        } else {
          this.configForm.mecHost = row.hostIp
          this.hostList.push(row)
          // [新增] 2026-01-05 单主机部署时，查询该主机的网络平面
          this.fetchHostNetworkPlanes(row.hostIp)
        }
      }).catch(() => {
        this.$message({
          showClose: true,
          type: 'warning',
          message: this.$t('tip.getTemplateListFail'),
          duration: 2000
        })
      })
    },
    /**
     * [新增] 2026-01-05 查询主机的网络平面配置
     * @param {String} hostIp - 主机IP地址
     */
    fetchHostNetworkPlanes (hostIp) {
      this.isLoadingNetworkPlanes = true
      this.availableNetworkPlanes = {}
      this.networkPlaneOptions = []

      inventory.getMecHostRecord(hostIp).then(res => {
        const networkPlanes = res.data.networkPlanes || {}
        this.availableNetworkPlanes = networkPlanes
        this.buildNetworkPlaneOptions(networkPlanes)
        this.isLoadingNetworkPlanes = false
      }).catch(error => {
        console.error('Failed to fetch network planes:', error)
        this.isLoadingNetworkPlanes = false
        // 查询失败时，只显示默认选项
        this.buildNetworkPlaneOptions({})
      })
    },
    /**
     * [新增] 2026-01-05 构建网络平面下拉选项
     * @param {Object} networkPlanes - 网络平面对象 {"eth1": "n6-net-1", "eth2": "n6-net-2"}
     */
    buildNetworkPlaneOptions (networkPlanes) {
      // 始终添加默认选项
      const options = [{
        label: this.$t('app.distriList.networkPlaneDefault'),
        value: 'default'
      }]

      // 遍历 networkPlanes 添加物理直通平面
      if (networkPlanes && typeof networkPlanes === 'object') {
        Object.keys(networkPlanes).forEach(eth => {
          const nad = networkPlanes[eth]
          options.push({
            label: `${this.$t('app.distriList.networkPlanePhysical')}(${nad})`,
            value: nad
          })
        })
      }

      this.networkPlaneOptions = options

      // 如果只有默认选项，自动选中
      if (options.length === 1) {
        this.configForm.selectedNetworkPlane = 'default'
      }
    },
    handleInputsData () {
      let VmData = []
      let NetData = []
      this.otherData = []
      this.templateInputs.forEach(item => {
        if (item.label.indexOf('VDU') === 0) {
          VmData.push(item)
        } else if (item.label.indexOf('APP') === 0) {
          NetData.push(item)
        } else {
          this.otherData.push(item)
        }
      })
      this.handleVmData(VmData)
      this.handleNetData(NetData)
    },
    handleVmData (VmData) {
      let map = {}
      this.VmDataGroup = []
      VmData.forEach(item => {
        let name = item.label.substring(0, 4)
        if (!map[name]) {
          this.VmDataGroup.push({
            label: item.label.substring(0, 4),
            VmResourceData: [],
            data: [item]
          })
          map[name] = item
        } else {
          for (const val of this.VmDataGroup) {
            if (val.label.substring(0, 4) === item.label.substring(0, 4)) {
              val.data.push(item)
              break
            }
          }
        }
      })
      this.VmDataGroup.forEach(item => {
        let NetDataGroup = []
        item.data.forEach(netItem => {
          if (netItem.label.length > 16) {
            let name = netItem.label.substring(0, 16)
            if (!map[name]) {
              NetDataGroup.push({
                label: netItem.label.substring(5, 16),
                data: [netItem]
              })
              map[name] = netItem
            } else {
              this.handleNetDataGroup(NetDataGroup, netItem)
            }
          } else {
            item.VmResourceData.push(netItem)
          }
        })
        item.data = NetDataGroup
      })
      this.VmDataGroup.forEach(item => {
        item.VmResourceData.forEach(resourceData => {
          if (resourceData.label.substring(5) === 'MEM') {
            resourceData.value = resourceData.value / 1024
          }
        })
      })
    },
    handleNetDataGroup (data, item) {
      for (const val of data) {
        if (val.label === item.label.substring(5, 16)) {
          val.data.push(item)
          break
        }
      }
    },
    handleNetData (NetData) {
      let map = {}
      this.netWorkSetData = []
      NetData.forEach(item => {
        let name = item.label.substring(0, 11)
        if (!map[name]) {
          this.netWorkSetData.push({
            label: item.label.substring(0, 11),
            data: [item]
          })
          map[name] = item
        } else {
          for (const val of this.netWorkSetData) {
            if (val.label.substring(0, 11) === item.label.substring(0, 11)) {
              val.data.push(item)
              break
            }
          }
        }
      })
    },
    handleNetName (label) {
      let labelListEn = ['Network', 'Physnet']
      let labelListCn = ['网络名称', '物理网络名称']
      if (labelListEn.includes(label.substring(12))) {
        let index = labelListEn.indexOf(label.substring(12))
        return this.language === 'cn' ? labelListCn[index] : labelListEn[index]
      } else {
        return label.substring(12)
      }
    },
    confirmToDeploy (configForm) {
      this.$refs[configForm].validate(valid => {
        if (valid) {
          // [修改] 2026-01-05 添加 networkPlane 参数
          let params = {
            appId: this.configForm.appId,
            appPackageId: this.configForm.appPackageId,
            appName: this.configForm.appName,
            appInstanceDescription: this.configForm.appInstanceDescription,
            mecHost: this.configForm.mecHost,
            hwCapabilities: this.configForm.hwCapabilities,
            // 携带选择的网络平面（默认选项时为 null）
            networkPlane: this.configForm.selectedNetworkPlane !== 'default'
              ? this.configForm.selectedNetworkPlane
              : null
          }
          this.loading = true
          if (typeof (params.mecHost) === 'string') {
            appo.confirmToDeploy(params).then(res => {
              let instanceId = res.data.response.app_instance_id
              this.timer = setTimeout(() => { this.queryInstanceStatus(instanceId) }, 1000)
            }).catch(() => {
              this.$message.error(this.$t('tip.deployFailed'))
              this.dialogVisible = false
            })
          } else {
            appo.confirmToBatchDeploy(params).then(res => {
              let instanceIds = res.data.response
              let len = instanceIds.length
              this.timer = setTimeout(() => { this.batchInstaniateApp(instanceIds) }, 6000 * len)
            }).catch(() => {
              this.$message.error(this.$t('tip.deployFailed'))
              this.dialogVisible = false
            })
          }
        }
      })
    },
    queryInstanceStatus (instanceids) {
      appo.getInstanceInfo(instanceids).then(res1 => {
        let status = res1.data.response.operationalStatus
        if (status === 'Created') {
          this.instaniateApp(instanceids)
        } else if (status === 'Create failed') {
          this.$message.error(res1.data.response.operationInfo)
          this.dialogVisible = false
          this.loading = false
        } else {
          this.timer = setTimeout(() => { this.queryInstanceStatus(instanceids) }, 1000)
        }
      }).catch(err => {
        if (err.name === 'Error' && err.message === 'Request failed with status code 404') {
          setTimeout(() => { this.queryInstanceStatus() }, 1000)
        } else {
          throw err
        }
      })
    },
    instaniateApp (instanceId) {
      if (this.templateInputs.length > 0) {
        let params = {
          parameters: {}
        }
        this.VmDataGroup.forEach(item => {
          item.VmResourceData.forEach(resourceItem => {
            if (resourceItem.label.substring(5) === 'MEM') {
              resourceItem.value = resourceItem.value * 1024
            }
            let key = resourceItem.label
            params.parameters[key] = resourceItem.value
          })
          item.data.forEach(data => {
            data.data.forEach(netData => {
              let key = netData.label
              params.parameters[key] = netData.value
            })
          })
        })
        this.netWorkSetData.forEach(item => {
          item.data.forEach(data => {
            let key = data.label
            params.parameters[key] = data.value
          })
        })
        this.otherData.forEach(item => {
          let key = item.label
          params.parameters[key] = item.value
        })
        appo.instantiateApp(instanceId, params).then(() => {
          this.afterInstantiateApp(instanceId)
        }).catch(() => {
          this.catchInstantiateApp()
        })
      } else {
        appo.instantiateApp(instanceId).then(() => {
          this.afterInstantiateApp(instanceId)
        }).catch(() => {
          this.catchInstantiateApp()
        })
      }
    },
    afterInstantiateApp (_instanceId) {
      this.loading = false
      this.dialogVisible = false
      this.$router.push('/mecm/app/instance')
    },
    catchInstantiateApp () {
      this.$message.error(this.$t('tip.deployFailed'))
      this.dialogVisible = false
      this.loading = false
    },
    batchInstaniateApp (instanceId) {
      let obj = {
        instantiationParameters: []
      }
      instanceId.forEach(item => {
        let paramObj = {
          parameters: {}
        }
        paramObj.appInstanceId = item.appInstanceId
        this.templateInputs.forEach(val => {
          let key = val.label
          paramObj.parameters[key] = val.value
        })
        obj.instantiationParameters.push(paramObj)
      })
      appo.batchInstantiateApp(obj).then(() => {
        this.afterInstantiateApp(instanceId)
      }).catch(() => {
        this.catchInstantiateApp()
      })
    },
    handleSelectionChange (selection) {
      this.selectData = selection
    },

    // 分发
    distribute () {
      let row = JSON.parse(sessionStorage.getItem('appRow'))
      console.log(row)
      this.currentRowData = row
      try {
        this.getResources()
      } catch (e) {
        console.log('Error in getting resources')
      }
      this.distributionDialogVisible = true
      this.getNodeList(row)
    },
    handleEdgeNodeSelectionChange (val) {
      this.nodeSelection = val
      this.selectedNodeNum = val.length
    },
    async getNodeList (row) {
      sessionStorage.setItem('appId', row.appId)
      await inventory.getList(2).then(response => {
        this.edgeNodesData = response.data
      }).catch((error) => {
        console.log(error)
      })
    },
    handleEdgeNodePageSizeChange (edgeNodePageSize) {
      this.edgeNodePageSize = edgeNodePageSize
    },
    handleEdgeNodeCurrentPageChange (edgeNodeCurrentPage) {
      this.edgeNodeCurrentPage = edgeNodeCurrentPage
    },
    cancel () {
      this.distributionDialogVisible = false
      this.knowEdge = true
      this.$refs.multipleEdgeNodeTable.clearSelection()
    },
    async confirm () {
      this.interval = setInterval(() => {
        this.initList()
      }, 15000)
      this.disLoading = true
      let selectedMecHost = []
      if (this.knowEdge) {
        this.nodeSelection.forEach(data => {
          let obj = {}
          obj.hostIp = data.mechostIp
          selectedMecHost.push(obj)
        })
        this.$refs.multipleEdgeNodeTable.clearSelection()
      }

      let address = window.location.protocol === 'https:' ? 'https://' : 'http://'
      let params = {
        appPkgId: this.currentRowData.packageId,
        appId: this.currentRowData.appId,
        appPkgName: this.currentRowData.name,
        appPkgVersion: this.currentRowData.version,
        appPkgDesc: this.currentRowData.shortDesc ? this.currentRowData.shortDesc : 'none',
        appPkgAffinity: this.currentRowData.affinity,
        appPkgPath: address + this.currentRowData.appstoreEndpoint + '/mec/appstore/v1/apps/' + this.currentRowData.appId + '/packages/' + this.currentRowData.packageId + '/action/download',
        appProvider: this.currentRowData.provider,
        mecHostInfo: selectedMecHost,
        createdTime: new Date().toString(),
        modifiedTime: new Date().toString()
      }
      if (!this.knowEdge) {
        params.intent = this.intentObj
      }
      console.log(JSON.stringify(params))
      if (params.appPkgVersion && (params.mecHostInfo.length > 0 || !this.knowEdge)) {
        apm.confirmToDistribute(params, this.knowEdge).then(response => {
          sessionStorage.setItem('appId', params.appId)
          if (!this.knowEdge) {
            this.knowEdge = !this.knowEdge
            this.filterNodeList(response.data.data)
          } else {
            this.distributionDialogVisible = false
          }
          this.$nextTick(
            this.initList()
          )
        }).catch(() => {
          this.disLoading = false
          this.$message.error(this.$t('tip.failedToDownload'), 3000)
        })
      } else {
        this.disLoading = false
        if (params.mecHostInfo.length === 0) {
          this.$message.warning(this.$t('tip.mecHost'))
        } else {
          this.$message.warning(this.$t('tip.version'))
        }
      }
    },
    filterNodeList (data) {
      this.edgeNodesData = this.edgeNodesData.filter((val) => {
        if (data.indexOf(val.mechostIp) > -1) {
          return val
        }
      })
    }
  }
}
</script>

<style lang='less'>
.configForm{
  padding: 0 30px;
  .title{
    margin-bottom: 12px;
  }
  .title::before{
    content:'';
    display:inline-block;
    width:3px;
    height:15px;
    margin-right:3px;
    background: #409EFF;
    position: relative;
    top:3px;
  }
  .el-form-item{
    margin-bottom: 20px !important;
  }
  .hostinfo{
    line-height: 40px;
  }
  .hostip{
    margin-right:10px;
  }
  .apptemplate{
    .vmcofig{
      margin-bottom: 35px;
    }
    .el-collapse{
      border: none;
      background-color: transparent !important;
      padding: 0 15px;
      .el-collapse-item__header{
        background-color: transparent;
        font-size: 18px;
        padding: 5px 0;
        color: #fff;
        border: none;
        font-weight: normal;
      }
      .el-collapse-item__wrap{
        background-color: transparent !important;
      }
      .el-collapse-item__header:hover {
        background-color: #5f499d;
        border-radius: 20px;
      }
    }
    .first-title{
      font-size: 18px;
      padding: 8px 10px;
      color: #fff;
    }
    .second-title{
      font-size: 16px;
      padding: 0 20px;
      color: #fff;
      line-height: 36px;
    }
    .apptemplate-form.el-form-item{
      margin-bottom: 5px !important;
      .el-form-item__label{
        font-size: 14px !important;
      }
    }
    .unit{
      position: relative;
      top: 5px;
    }
  }
}
.addButton {
  background: rgb(88, 68, 190);
  color: rgb(255, 255, 255);
  padding: 5px 15px;
  height: 30px;
}
</style>
