<template>
  <div v-loading="loading">
    <div v-if="showLockedAlert" class="locked-projects-alert">
      <el-alert
        :title="lockedProjects.join('，') + '已锁定'"
        type="warning"
        show-icon
        :closable="true"
        @close="handleLockedAlertClose"
      ></el-alert>
    </div>

    <div>
      <div class="search-button">
        <el-button
          :class="searchIconChange ? 'search-icon opened' : 'search-icon'"
          @click="toggleFiltering"
        ></el-button>
      </div>
      <!-- <div>
        <has-permission pname="downloadProductAccess">
          <el-button class="icon-downloadProduct" @click="exportData"></el-button>
        </has-permission>
      </div> -->
    </div>

    <el-tabs v-model="productAccessTabs" @tab-click="tabChange">
      <el-tab-pane label="产品准入" name="productAccess">
        <div v-show="productSearchVisible" class="filtering-section">
          <!-- 产品准入搜索 -->
          <div v-show="productAccessTabs == 'productAccess'">
            <el-form
              ref="searchForm"
              :model="productSearchOption"
              label-position="top"
            >
              <el-row class="custom-flex-row">
                <el-col class="grid-content">
                  <el-form-item label="关注等级" prop="attentionLevel">
                    <el-select
                      v-model="productSearchOption.attentionLevelList"
                      clearable
                      multiple
                    >
                      <el-option label="重点关注" value="2">
                        <span class="option-dot red"></span>
                        <span>重点关注</span>
                      </el-option>
                      <el-option label="需要关注" value="1">
                        <span class="option-dot yellow"></span>
                        <span>需要关注</span>
                      </el-option>
                      <el-option label="无需关注" value="0">
                        <span class="option-dot gray"></span>
                        <span>无需关注</span>
                      </el-option>
                    </el-select>
                  </el-form-item>
                </el-col>
                <el-col class="grid-content">
                  <el-form-item label="项目名称" prop="projectName">
                    <el-select
                      v-model="productSearchOption.projectIdList"
                      clearable
                      filterable
                      multiple
                    >
                      <el-option
                        v-for="[name, id] of projectMap"
                        :key="id"
                        :label="name"
                        :value="id"
                      >
                        {{ name }}
                      </el-option>
                    </el-select>
                  </el-form-item>
                </el-col>
<!--                <el-col :span="6">
                  <el-form-item label="产品名称" prop="productName">
                    <el-select
                      clearable
                      filterable
                      multiple
                      v-model="productSearchOption.productCodeList"
                    >
                      <el-option
                        v-for="[name, code] of productMap"
                        :key="code"
                        :label="name"
                        :value="code"
                      />
                    </el-select>
                  </el-form-item>
                </el-col>-->
                <el-col class="grid-content">
                  <el-form-item label="BU" prop="bu">
                    <el-select
                      clearable
                      multiple
                      filterable
                      v-model="productSearchOption.buList"
                    >
                      <el-option
                        v-for="(bu, index) in [...new Set(buList)]"
                        :key="index"
                        :label="bu"
                        :value="bu"
                      />
                    </el-select>
                  </el-form-item>
                </el-col>
                <el-col class="grid-content">
                  <el-form-item label="终端名称" prop="terminalName">
                    <el-select
                      v-model="productSearchOption.terminalNameList"
                      clearable
                      filterable
                      multiple
                    >
                      <el-option
                        v-for="[name, code] of terminalMap"
                        :key="code"
                        :label="name"
                        :value="code"
                      />
                    </el-select>
                  </el-form-item>
                </el-col>
                <el-col class="grid-content">
                  <el-form-item label="终端省份" prop="terminalProvince">
                    <el-select
                      v-model="productSearchOption.terminalProvinceList"
                      @change="selectProvince"
                      clearable
                      filterable
                      multiple
                    >
                      <el-option
                        v-for="(province, index) in provinceNameList"
                        :key="index"
                        :label="province"
                        :value="province"
                      />
                    </el-select>
                  </el-form-item>
                </el-col>
              </el-row>
              <el-row class="custom-flex-row">
                <el-col class="grid-content">
                  <el-form-item label="准入状态" prop="type">
                    <el-select
                      clearable
                      multiple
                      v-model="productSearchOption.actualAccessMethodList"
                    >
                      <el-option
                        v-for="[label, value] of admissionStatus"
                        :key="label"
                        :label="label"
                        :value="value"
                      />
                    </el-select>
                  </el-form-item>
                </el-col>
                <el-col class="grid-content">
                  <el-form-item label="锁定与认定" prop="locked">
                    <el-select
                      multiple
                      clearable
                      v-model="productSearchOption.isLock"
                    >
                      <el-option
                        v-for="[label, value] of isLockMap"
                        :key="label"
                        :label="label"
                        :value="value"
                      />
                    </el-select>
                  </el-form-item>
                </el-col>
                <el-col class="grid-content">
                  <el-form-item label="变更起始日期（年/月/日）" prop="startDate">
                    <el-date-picker
                      style="width: 100%"
                      v-model="productSearchOption.startTime"
                      type="date"
                      value-format="yyyy/MM/dd"
                      placeholder="选择日期"
                      :picker-options="startPickerOptions"
                    ></el-date-picker>
                  </el-form-item>
                </el-col>
                <el-col class="grid-content">
                  <el-form-item label="变更截止日期（年/月/日）" prop="endDate">
                    <el-date-picker
                      style="width: 100%"
                      v-model="productSearchOption.endTime"
                      type="date"
                      value-format="yyyy/MM/dd"
                      placeholder="选择日期"
                      :picker-options="endPickerOptions"
                    ></el-date-picker>
                  </el-form-item>
                </el-col>
              </el-row>
            </el-form>
          </div>
          <!-- 搜索按钮 -->
          <div
            style="
              display: flex;
              justify-content: end;
              border-bottom: 1px solid #dcdfe6;
              height: 40px;
            "
          >
            <el-button class="cancel" style="height: 25px" @click="clearSearch">
              清空
            </el-button>
            <el-button
              class="confirm"
              style="height: 25px"
              @click="submitSearch"
            >
              提交
            </el-button>
          </div>
        </div>
        <div class="button-bar">
          <!--          <has-permission pname="windowsProductAccess">-->
          <!--            <el-button class="bayer-button" @click="loadData()">-->
          <!--              设置窗口期-->
          <!--            </el-button>-->
          <!--          </has-permission>-->
          <el-tooltip
            content="变更记录导出"
            placement="top"
          >
            <el-button
              class="icon-importAdmission"
              @click="openImportDialog"
              v-if="
                role === '1' ||
                role === '5'
              "
            ></el-button>
          </el-tooltip>
          <el-button
            class="icon-downloadProduct"
            @click="exportData"
            v-if="
              role === '1' ||
              role === '2' ||
              role === '3' ||
              role === '4' ||
              role === '5'
            "
          ></el-button>
        </div>
        <el-table
          ref="multipleTable"
          v-loading="tableLoding"
          :data="productAccessList"
          :row-class-name="tableRowClassName"
          :header-cell-class-name="handleHeaderClass"
          style="width: 100%"
          :height="448"
          @sort-change="sortChange"
        >
          <el-table-column prop="lockAndConfirmed" label="" width="40">
            <template #default="{ row }">
              <span
                v-if="row.actualAccessMethod !== '正式准入' && !row.isConfirmed"
                style="visibility: hidden"
              >
                <img
                  src="@/assets/icon/lock2.png"
                  style="height: 18px; vertical-align: middle"
                />
              </span>
              <el-tooltip
                class="item"
                effect="dark"
                v-else-if="
                  row.actualAccessMethod !== '正式准入' && row.isConfirmed
                "
                content="非正式准入已认定"
                placement="top"
              >
                <img
                  src="@/assets/icon/lock2.png"
                  style="height: 18px; vertical-align: middle"
                />
              </el-tooltip>
              <el-tooltip
                class="item"
                effect="dark"
                v-else-if="
                  row.actualAccessMethod === '正式准入' &&
                  !row.isConfirmed &&
                  row.isLock
                "
                content="正式准入已锁定"
                placement="top"
              >
                <img
                  src="@/assets/icon/lock.png"
                  style="height: 18px; vertical-align: middle"
                />
              </el-tooltip>
              <el-tooltip
                class="item"
                effect="dark"
                v-else-if="
                  row.actualAccessMethod === '正式准入' && row.isConfirmed
                "
                content="正式准入已认定"
                placement="top"
              >
                <img
                  src="@/assets/icon/lock2.png"
                  style="height: 18px; vertical-align: middle"
                />
              </el-tooltip>
              <span
                v-else-if="
                  row.actualAccessMethod === '正式准入' && !row.isConfirmed
                "
                style="visibility: hidden"
              >
                <img
                  src="@/assets/icon/lock2.png"
                  style="height: 18px; vertical-align: middle"
                />
              </span>
            </template>
          </el-table-column>
          <!--          <el-table-column-->
          <!--            prop="projectId"-->
          <!--            sortable="custom"-->
          <!--            label="项目编号"-->
          <!--            min-width="100"-->
          <!--          >-->
          <!--            <template #default="{ row }">-->
          <!--              <el-button-->
          <!--                type="text"-->
          <!--                @click="openProductAccessDetail(row, 'productAccessDetail')"-->
          <!--              >-->
          <!--                {{ row.projectId }}-->
          <!--              </el-button>-->
          <!--            </template>-->
          <!--          </el-table-column>-->
          <el-table-column
            prop="attentionLevel"
            sortable="custom"
            label="关注等级"
            min-width="100"
          >
            <template #default="{ row }">
              <el-dropdown
                trigger="click"
                @command="val => handleAttentionCommand(row, val)"
              >
                <span
                  class="attention-tag-wrapper"
                  :class="{
                    empty: !row.attentionLevel || row.attentionLevel === '0',
                    high: row.attentionLevel === '2',
                    need: row.attentionLevel === '1'
                  }"
                >
                  <span class="attention-text">
                    {{ row.attentionLevel === '2'
                    ? '重点关注'
                    : row.attentionLevel === '1'
                      ? '需要关注'
                      : '' }}
                  </span>

                  <el-icon class="dropdown-icon">
                    <ArrowDown />
                  </el-icon>
                </span>

                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item command="2" class="item-red">
                      <span class="dot red"></span> 重点关注
                    </el-dropdown-item>
                    <el-dropdown-item command="1" class="item-yellow">
                      <span class="dot yellow"></span> 需要关注
                    </el-dropdown-item>
                    <el-dropdown-item command="0" class="item-gray">
                      <span class="dot gray"></span> 无需关注
                    </el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
            </template>

          </el-table-column>

          <el-table-column
            prop="projectName"
            sortable="custom"
            min-width="100"
            label="项目名称"
          >
            <template #default="{ row }">
              {{ row.projectName }}
            </template>
          </el-table-column>
          <el-table-column
            prop="productName"
            sortable="custom"
            label="产品名称"
            min-width="100"
          >
            <template #default="{ row }">
              {{ row.productName }}
            </template>
          </el-table-column>
          <el-table-column
            prop="bu"
            sortable="custom"
            label="BU"
            min-width="100"
          >
            <template #default="{ row }">
              {{ row.bu }}
            </template>
          </el-table-column>
          <el-table-column
            prop="terminalName"
            sortable="custom"
            :width="flexColumnWidth(productAccessList, 'terminalName', 300)"
            label="终端名称"
            min-width="100"
          >
            <template #default="{ row }">
              {{ row.terminalName }}
            </template>
          </el-table-column>
          <el-table-column
            prop="terminalProvince"
            sortable="custom"
            min-width="100"
            label="终端省份"
          >
            <template #default="{ row }">
              {{ row.terminalProvince }}
            </template>
          </el-table-column>
          <el-table-column
            prop="actualAccessMethod"
            sortable="custom"
            label="准入状态"
            min-width="100"
          >
            <template #default="{ row }">
              {{ row.actualAccessMethod }}
            </template>
          </el-table-column>
          <el-table-column
            prop="lockAndConfirmed"
            sortable="custom"
            label="锁定与认定"
            width="120"
          >
            <template #default="{ row }">
              {{ row.lockAndConfirmed }}
            </template>
          </el-table-column>
          <el-table-column label="操作" fixed="right" width="80">
            <template #default="{ row }">
              <el-button
                type="text"
                icon="el-icon-view"
                class="custom-icon-btn"
                @click="openProductAccessDetail(row, 'productAccessDetail')"
              ></el-button>
              <el-button
                type="text"
                icon="el-icon-edit"
                class="custom-icon-btn"
                style="padding-left: 10px"
                @click="openProductAccessDetail(row, 'productAccessInfo')"
                :disabled="disabledOption(row)"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-dialog
          top="8vh"
          width="90%"
          :title="dialogTitle"
          ref="productDialog"
          :visible.sync="dialogProductVisible"
          v-if="dialogProductVisible"
          @close="handleDialogProductClose"
          :before-close="handleBeforeClose"
          :close-on-click-modal="true"
          :close-on-press-escape="false"
        >
          <template #title>
            <div style="display: inline-flex; align-items: center">
              <span style="font-size: 20px">{{ project.projectName }} - {{ project.terminalName }} | {{ project.productName }}</span>
              &nbsp;&nbsp;&nbsp;
              <el-dropdown
                class="attention-dropdown"
                trigger="click"
                :disabled="showEdit"
                @command="val => handleDetailAttention(project, val)"
              >
                <span
                  v-if="!project.attentionLevel || project.attentionLevel === '0'"
                  class="attention-empty"
                >无需关注
                </span>
                <span
                  v-else
                  class="attention-tag"
                  :class="project.attentionLevel === '2' ? 'high' : 'need'"
                >
                  {{ project.attentionLevel === '2' ? '重点关注' : '需要关注' }}
                </span>

                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item command="2" class="item-red">
                      <span class="dot red"></span> 重点关注
                    </el-dropdown-item>
                    <el-dropdown-item command="1" class="item-yellow">
                      <span class="dot yellow"></span> 需要关注
                    </el-dropdown-item>
                    <el-dropdown-item command="0" class="item-gray">
                      <span class="dot gray"></span> 无需关注
                    </el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
              &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
              <span v-if="project.KAHospitalOne !== ''" class="ka-tag blue">{{ project.KAHospitalOne }}</span>
              &nbsp;&nbsp;
              <span v-if="project.KAHospitalTwo !== ''" class="ka-tag yellow">{{ project.KAHospitalTwo }}</span>
            </div>
          </template>
          <el-scrollbar style="height: 72vh">
            <ProductAccessInfo
              :current-title="dialogTitle"
              :select-data="selectData"
              :show-edit="showEdit"
              :admission-status-map="admissionStatusMap"
              :recent-so-status-map="recentSoStatusMap"
              :recent-so-type-map="recentSoTypeMap"
              :reverse-entry-reason-map="reverseEntryReasonMap"
              :so-validated-map="soValidatedMap"
              :commercial-territory="commercialTerritory"
              :commercial-territory2="commercialTerritory2"
              :distributor-infor="distributorInfor"
              :attention-level="parantAttentionLevel"
              @close-dialog="closeDialogHandler"
            />
          </el-scrollbar>
        </el-dialog>
        <el-pagination
          background
          :current-page="currentPage"
          :page-sizes="[20, 50, 100]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="totalOrder"
          @size-change="handleSizeChange"
          @current-change="handlePageChange"
        />
      </el-tab-pane>
      <el-dialog
        title="导入"
        :visible.sync="dialogVisibleImportConfirmedResult"
        width="60%"
        :close-on-click-modal="false"
        custom-class="dialogstyle"
        @closed="closeImportDialog()"
      >
        <el-form
          ref="batchUploadForm"
          label-position="top"
          label-width="100px"
          size="small"
        >
          <el-form-item label-width="0" prop="upload">
            <el-upload
              ref="upload"
              class="uploadsection"
              drag
              show-file-list
              :auto-upload="false"
              :file-list="fileList"
              :limit="1"
              :on-exceed="handleExceed"
              :on-change="handleFileChange"
              :on-remove="handleRemove"
            >
              <i class="el-icon-upload"></i>
              <div class="el-upload__text">
                将文件拖到此处，或
                <em>点击上传</em>
              </div>
              <div slot="tip" class="el-upload__tip">
                文件大小不超过20M
              </div>
            </el-upload>
          </el-form-item>
        </el-form>
        <div class="downloadTemplateBtn" @click="downloadTemplate">
          下载导入模版
        </div>
        <div slot="footer">
          <el-button class="cancel" @click="dialogVisibleImportConfirmedResult = false">
            取消
          </el-button>
          <el-button
            class="confirm"
            :loading="saveLoading"
            @click="batchUpload()"
          >
            提交
          </el-button>
        </div>
      </el-dialog>
    </el-tabs>
  </div>
</template>

<script>
  import ProductAccessInfo from '@/views/ProductAccess/ProductAccessInfo.vue'
  import dayjs from "dayjs";
  import axios from 'axios'

  export default {
    name: 'ProductAccessTable',
    components: {
      ProductAccessInfo,
    },
    data() {
      return {
        startPickerOptions: {
          disabledDate: (time) => {
            if (this.productSearchOption.endTime) {
              return time.getTime() > new Date(this.productSearchOption.endTime).getTime()
            }
            return false
          },
        },
        endPickerOptions: {
          disabledDate: (time) => {
            if (this.productSearchOption.startTime) {
              return time.getTime() < new Date(this.productSearchOption.startTime).getTime()
            }
            return false
          },
        },
        currentWindowNumber: '',
        currentUserRoles: [],
        terminalMap: new Map(),
        admissionStatus: new Map(),
        productAccessTabs: 'productAccess',
        productMap: new Map(),
        provinceNameList: [],
        projectMap: new Map(),
        isLockMap: new Map(),
        buList: [],
        dialogTitle: '',
        saveLoading: false,
        dialogWindowsVisible: false,
        searchIconChange: true,
        productSearchVisible: true,
        productSearchOption: {
          projectIdList: [],
          productCodeList: [],
          buList: [],
          terminalNameList: [],
          terminalProvinceList: [],
          actualAccessMethodList: [],
          isLock: [],
          startTime: "",
          endTime: "",
          attentionLevelList:[],
        },
        countryCityList: [],
        totalOrder: 0,
        dialogProductVisible: false,
        selectData: {}, //详情
        showEdit: false,
        selectLoading: false,
        productAccessList: [],
        lockedProjects: [],
        showLockedAlert: false,
        currentPage: 1,
        pageSize: 20,
        loading: true,
        exportLoading: false,
        sortRules: [],
        isEdit: 0,
        role: '',
        userRoles: [],
        admissionStatusMap: new Map(),
        recentSoStatusMap: new Map(),
        recentSoTypeMap: new Map(),
        reverseEntryReasonMap: new Map(),
        soValidatedMap: new Map(),
        commercialTerritory: [],
        commercialTerritory2: [],
        distributorInfor: [],
        currentDistributorInfor: [],
        projectSupportDealer: [],
        tableLoding: false,
        hasUnsavedChanges: false, // 标记是否有未保存的修改
        isClosing: false, // 防止重复触发
        currentQueryParams: {},
        project: {
          projectName: '',
          terminalName: '',
          productName: '',
          attentionLevel: '',
          KAHospitalOne: '',
          KAHospitalTwo: '',
        },
        parantAttentionLevel: '',//传给子页面的值
        dialogVisibleImportConfirmedResult: false,
        fileList: [],
        file: null,
        userEmail: '',
      }
    },
    mounted() {
      if (this.$route.query.productAccessTabs != null) {
        this.productAccessTabs = this.$route.query.productAccessTabs
      }
      this.getProductAccessDropdown()
      this.getProductAdmissionListFilterOptions()
      this.fetchData()
    },
    created() {
      /**
       * 一线商务/大区经理/固定负责人/商务总监/管理员
       */
      let { userRoles } = JSON.parse(sessionStorage.getItem('userInfo'))
      this.userRoles = userRoles
      if (
        this.userRoles.filter((item) => {
          return item.roleName.indexOf('管理员') > -1
        }).length > 0
      ) {
        this.role = '1'
      } else if (
        this.userRoles.filter((item) => {
          return item.roleName == '商务一线'
        }).length > 0
      ) {
        this.role = '2'
      } else if (
        this.userRoles.filter((item) => {
          return item.roleName == '固定负责人'
        }).length > 0
      ) {
        this.role = '3'
      } else if (
        this.userRoles.filter((item) => {
          return item.roleName == '商务经理'
        }).length > 0
      ) {
        this.role = '4'
      } else if (
        this.userRoles.filter((item) => {
          return item.roleName == '商务总监'
        }).length > 0
      ) {
        this.role = '5'
      }
    },

      methods: {
        handleLockedAlertClose() {
          this.showLockedAlert = false;
        },
        disabledOption(row) {
          if (this.userRoles) {
            if (row.onlyCentral) {
              if (
              this.userRoles.filter((item) => {
                return (
                  item.roleName.indexOf('商务运营') > -1
                )
              }).length > 0
            ) {
              return false
            } else {
              return true
            }
          } else {
            if (row.isFreeze) {
              return true
            }
            if (row.actualAccessMethod === '正式准入') {
              if (row.isConfirmed) {
                if (
                  this.userRoles.filter((item) => {
                    return (
                      item.roleName.indexOf('商务经理') > -1 ||
                      item.roleName.indexOf('商务总监') > -1 ||
                      item.roleName.indexOf('商务一线') > -1 ||
                      item.roleName.indexOf('固定负责人') > -1
                    )
                  }).length > 0
                ) {
                  return true
                }
              }
              if (row.isLock) {
                if (row.charge) {
                  return false
                }
                if (
                  this.userRoles.filter((item) => {
                    return (
                      item.roleName.indexOf('商务经理') > -1 ||
                      item.roleName.indexOf('商务总监') > -1 ||
                      item.roleName.indexOf('商务一线') > -1
                    )
                  }).length > 0
                ) {
                  return true
                }
              }
            }
          }
          return false
        } else {
          return true
        }
      },
      canShowByRoleName(roleName) {
        if (roleName == null || roleName == undefined) {
          return false
        } else if (roleName == 'RKA') {
          return (
            this.currentUserRoles.filter((item) => {
              return item.roleName == 'KA管理员'
            }).length > 0
          )
        } else if (roleName == 'KA经理') {
          return (
            this.currentUserRoles.filter((item) => {
              return item.roleName == 'RKA' || item.roleName == 'KA管理员'
            }).length > 0
          )
        } else {
          return false
        }
      },
      tableRowClassName({ row, rowIndex }) {
        if (
          (row.actualAccessMethod === '正式准入' &&
            row.isLock &&
            !row.isConfirmed) ||
          (row.actualAccessMethod === '正式准入' && row.isConfirmed)
        ) {
          return 'locked-row' // 当 locked 为 true 时返回自定义类名
        }
        return '' // 否则返回空字符串
      },
      // 保持表头排序图标状态
      handleHeaderClass({ column }) {
        const rule = this.sortRules.find(
          (rule) => rule.prop === column.property
        )
        if (rule) {
          // 将当前列的排序状态设置为自定义的状态
          column.order = rule.order
        }
      },
      getProductAccessDropdown() {
        this.$apiFetch
          .post(`/acc/productAdmission/getListFilterOptions`, {})
          .then((res) => {
            if (res && res.code == 200) {

              // 使用后端返回的锁定项目列表
              console.log('getListFilterOptions 返回数据:', res.data);
              if (res.data.lockedProjects && res.data.lockedProjects.length > 0) {
                console.log('锁定项目:', res.data.lockedProjects);
                this.lockedProjects = res.data.lockedProjects;
                this.showLockedAlert = true;
              }

              this.projectMap = new Map(Object.entries(res.data.project || {}))
              this.productMap = new Map(Object.entries(res.data.product || {}))
              this.terminalMap = new Map(
                Object.entries(res.data.terminal || {})
              )
              this.isLockMap = new Map(Object.entries(res.data.isLock || {}))
              this.buList = res.data.bu || []
              this.provinceNameList = res.data.province || []
              this.admissionStatus = new Map(
                Object.entries(res.data.admissionStatus || {})
              )
            }
          })
          .catch((error) => {})
      },
      // getKAUsers() {
      //   this.$apiFetch
      //     .get(`/dfs/ProductAccess/getUserDataByRole?roleName=KA`)
      //     .then((res) => {
      //       if (res && res.code == 200) {
      //         this.KAUsers = res.data.users
      //       }
      //     })
      // },
      // getRKAUsers() {
      //   this.$apiFetch
      //     .get(`/dfs/ProductAccess/getUserDataByRole?roleName=RKA`)
      //     .then((res) => {
      //       if (res && res.code == 200) {
      //         this.RKAUsers = res.data.users
      //       }
      //     })
      // },
      tabChange(tab, event) {
        if (tab.index == 0) {
          this.productAccessTabs = 'productAccess'
        } else if (tab.index == 1) {
          this.$apiFetch.get(`/user/user/getUserInfo`).then((res) => {
            if (res && res.code == 200) {
              this.currentUserRoles = res.data.userRoles
            }
          })
          this.productAccessTabs = 'performance'
        }
      },
      getYearMonthWeek() {
        //获取最近三年
        let nowDate = new Date()
        let date = {
          year: nowDate.getFullYear(),
          month: nowDate.getMonth() + 1,
          date: nowDate.getDate(),
        }

        let yearCount = 8
        let year = date.year - 3
        let yearList = []
        let monthList = [
          '01',
          '02',
          '03',
          '04',
          '05',
          '06',
          '07',
          '08',
          '09',
          '10',
          '11',
          '12',
        ]
        let weekList = ['第1周', '第2周', '第3周', '第4周']
        while (yearCount > 1) {
          yearList.push(year)
          year = year + 1
          yearCount = yearCount - 1
        }
        yearList.sort(function (a, b) {
          return b - a
        })

        let count = 1
        for (var i = 0; i < yearList.length; i++) {
          var param = []
          for (var j = 0; j < monthList.length; j++) {
            var week = []
            for (var k = 0; k < weekList.length + 1; k++) {
              const dateRange = this.getDateRangeForWeek(
                yearList[i],
                j + 1,
                k + 1
              )
              let weekDate =
                weekList[k] +
                '(' +
                dateRange.startDate.toLocaleDateString() +
                '-' +
                dateRange.endDate.toLocaleDateString() +
                ')'
              if (k === 4) {
                let month = 0
                if (j === 11) {
                  month = 1
                } else {
                  month = j + 2
                }
                let nextMonth = yearList[i] + '-' + month + '-1 00:00:00'
                let nextMonthDate = new Date(nextMonth)

                if (this.compareTime(nextMonthDate, dateRange.endDate)) {
                  weekDate =
                    '第5周(' +
                    dateRange.startDate.toLocaleDateString() +
                    '-' +
                    dateRange.endDate.toLocaleDateString() +
                    ')'
                  week.push({ label: weekDate, value: weekDate })
                  count = count + 1
                }
              } else {
                week.push({ label: weekDate, value: weekDate })
                count = count + 1
              }
            }
            param.push({
              label: monthList[j],
              value: monthList[j],
              children: week,
            })
          }
          this.monthYearWeekOptions.push({
            label: yearList[i],
            value: yearList[i],
            children: param,
          })
        }
        // console.log('this.monthYearWeekOptions')
        // console.log(this.monthYearWeekOptions)
      },
      getDateRangeForWeek(year, month, week) {
        // 创建一个日期对象，设置为指定年份和月份的第一天
        const firstDayOfMonth = new Date(year, month - 1, 1)

        // 获取第一天是星期几（0表示星期日，1表示星期一，以此类推）
        const firstDayOfWeek = firstDayOfMonth.getDay()

        // 计算第一周的日期范围
        let startDateFirstWeek = new Date(
          year,
          month - 1,
          1 - firstDayOfWeek + 1
        )
        let endDateFirstWeek = new Date(year, month - 1, 7 - firstDayOfWeek + 1)

        if (firstDayOfWeek === 0) {
          // 计算第一周的日期范围
          startDateFirstWeek = new Date(year, month - 1, 1 - firstDayOfWeek - 6)
          endDateFirstWeek = new Date(year, month - 1, 7 - firstDayOfWeek - 6)
        }

        // 计算给定周的日期范围
        const startDateGivenWeek = new Date(startDateFirstWeek)
        startDateGivenWeek.setDate(
          startDateFirstWeek.getDate() + (week - 1) * 7
        )
        const endDateGivenWeek = new Date(startDateGivenWeek)
        endDateGivenWeek.setDate(startDateGivenWeek.getDate() + 6)

        // 返回日期范围
        return {
          startDate: startDateGivenWeek,
          endDate: endDateGivenWeek,
        }
      },

      closeDialogNewWindows() {
        this.dialognewWindowsForm.name = ''
        this.$refs.dialognewWindowsForm.resetFields()
      },
      loadData() {
        this.dialogWindowsVisible = true
      },
      addNewWindows() {},
      compareTime(time, compareTime) {
        // console.log(time - compareTime); // 这里计算两个时间之间的毫秒差
        return time - compareTime > 0
      },
      deleteProductAccess(row) {
        let userInfo = JSON.parse(sessionStorage.getItem('userInfo'))
        let params = {
          userId: userInfo.userInfo.id,
          action: 'delete',
          id: row.id,
        }
        // this.$apiFetch
        //   .post(`/dfs/ProductAccess/saveProductAccess`, params)
        //   .then((res) => {
        //     if (res && res.code == 200) {
        //       this.$message.success('删除成功')
        //       this.fetchData()
        //     }
        //   })
        //   .catch((error) => {
        //     console.log(error)
        //   })
      },
      getBUList() {
        let userInfo = JSON.parse(sessionStorage.getItem('userInfo'))
        const queryParams = new URLSearchParams({
          currentUser: userInfo.userInfo.id,
        })
        // this.$apiFetch
        //   .post(`/dfs/ProductAccess/getProductOption?${queryParams}`)
        //   .then((res) => {
        //     if (res && res.code == 200) {
        //       this.buList = res.data.productOption
        //     }
        //   })
        //   .catch((error) => {
        //     console.log(error)
        //   })
      },
      getProjectList() {
        let userInfo = JSON.parse(sessionStorage.getItem('userInfo'))
        const queryParams = new URLSearchParams({
          currentUser: userInfo.userInfo.id,
        })
        // this.$apiFetch
        //   .post(`/dfs/ProductAccess/getProductOption?${queryParams}`)
        //   .then((res) => {
        //     if (res && res.code == 200) {
        //       this.projectNameList = res.data.productOption
        //     }
        //   })
        //   .catch((error) => {
        //     console.log(error)
        //   })
      },
      getProductDropdownList() {
        let userInfo = JSON.parse(sessionStorage.getItem('userInfo'))
        const queryParams = new URLSearchParams({
          currentUser: userInfo.userInfo.id,
        })
        // this.$apiFetch
        //   .post(`/dfs/ProductAccess/getProductOption?${queryParams}`)
        //   .then((res) => {
        //     if (res && res.code == 200) {
        //       this.productDropdownList = res.data.productOption
        //     }
        //   })
        //   .catch((error) => {
        //     console.log(error)
        //   })
      },
      getProvinceDropdown() {
        let dropDown = JSON.parse(sessionStorage.getItem('dropDown'))
        let provinceList = dropDown.countryCity
        provinceList.forEach((item) => {
          if (!item.parentId) {
            this.provinceDropdown.push(item)
            this.provinceNameList.push(item)
          }
        })
        this.initDropdownData = JSON.parse(sessionStorage.getItem('dropDown'))
      },
      filterCityNameCN(search) {
        let val = search.trim()
        if (val) {
          this.CityNameList = this.cityDropdown.filter((item) => {
            if (
              item.dropdownValue.indexOf(val) >= 0 ||
              item.dropdownValue.toUpperCase().indexOf(val.toUpperCase()) >= 0
            )
              return true
          })
        } else {
          this.CityNameList = [...this.cityDropdown]
        }
      },
      filterProvinceNameCN(search) {
        let val = search.trim()
        if (val) {
          this.provinceNameList = this.provinceDropdown.filter((item) => {
            if (
              item.dropdownValue.indexOf(val) >= 0 ||
              item.dropdownValue.toUpperCase().indexOf(val.toUpperCase()) >= 0
            )
              return true
          })
        } else {
          this.provinceNameList = [...this.provinceDropdown]
        }
      },
      selectProvince(value) {
        let dropDown = JSON.parse(sessionStorage.getItem('dropDown'))
        let provinceList = dropDown.countryCity
        this.cityDropdown = []
        provinceList.forEach((item) => {
          if (item.parentId === value) {
            this.cityDropdown.push(item)
          }
        })
        // this.productSearchOption.city = ''
        this.CityNameList = [...this.cityDropdown]
      },
      toggleFiltering() {
        this.searchIconChange = !this.searchIconChange
        this.productSearchVisible = !this.productSearchVisible
      },
      clearSearch() {
        this.productSearchOption = {
          accessState: '',
          KaRegion: '',
          province: '',
          terminalName: '',
          product: '',
          drugCouncilTime: [],
          hospitalLevel: '',
          productAccessGrade: '',
          number: '',
          windowNumber: '',
          RKA: '',
          KA: '',
          status: '',
        }
        if (this.productAccessTabs == 'productAccess') this.fetchData()
        else if (this.productAccessTabs == 'performance')
          this.$refs.performance.performanceDataInit()
      },
      submitSearch() {
        this.currentPage = 1
        this.fetchData()
      },
      exportData() {
        if (this.exportLoading) {
          this.$message.warning('正在下载，请稍后')
          return
        }
        if (this.productAccessTabs == 'productAccess') {
          this.exportLoading = true
          let userInfo = JSON.parse(sessionStorage.getItem('userInfo'))
          const queryParams = {
            userId: userInfo.userInfo.id,
          }

          const addListIfNotEmpty = (paramName, list) => {
            if (list && Array.isArray(list) && list.length > 0) {
              // queryParams[paramName] = JSON.stringify(list)
              queryParams[paramName] = list
            }
            // 如果为空，则跳过，params对象中将不会有这个属性
          }

          addListIfNotEmpty(
            'projectIdList',
            this.productSearchOption.projectIdList
          )
          addListIfNotEmpty(
            'productCodeList',
            this.productSearchOption.productCodeList
          )
          addListIfNotEmpty('buList', this.productSearchOption.buList)
          addListIfNotEmpty(
            'terminalCodeList',
            this.productSearchOption.terminalNameList
          )
          addListIfNotEmpty(
            'terminalProvinceList',
            this.productSearchOption.terminalProvinceList
          )
          addListIfNotEmpty(
            'actualAccessMethodList',
            this.productSearchOption.actualAccessMethodList
          )
          addListIfNotEmpty('isLock', this.productSearchOption.isLock)
          addListIfNotEmpty(
            'attentionLevelList',
            this.productSearchOption.attentionLevelList
          )
          queryParams.startTime = this.productSearchOption.startTime
          queryParams.endTime = this.productSearchOption.endTime
          this.$apiFetch
            .PostFile(
              `/acc/productAdmission/downloadExcelTemplate`,
              queryParams
            )
            .then((response) => {
              console.log(response)
              if (response && response.status === 200) {
                let fileName = ''
                const contentDisposition =
                  response.headers['content-disposition']
                if (contentDisposition) {
                  const filenamePart = contentDisposition
                    .split(';')
                    .find((part) => part.trim().startsWith('filename'))
                  if (filenamePart) {
                    fileName = decodeURIComponent(
                      filenamePart.split('=')[1].trim().replace(/"/g, '')
                    )
                  }
                }
                this.downloadStreamFileToLocal(response.data, fileName)
                this.$message.success('导出成功!')
              }
              this.exportLoading = false
            })
            .catch((error) => {
              this.exportLoading = false
              console.log(error.message)
              this.$message.error('失败')
              // 处理错误
            })
        } else if (this.productAccessTabs == 'performance')
          this.$refs.performance.exportPerformance()
      },
      downloadStreamFileToLocal(buffers, fileName = '') {
        const blob = new Blob([buffers])
        if (window.navigator && window.navigator.msSaveOrOpenBlob) {
          window.navigator.msSaveOrOpenBlob(blob, fileName)
        } else {
          const downloadElement = document.createElement('a')
          const href = window.URL.createObjectURL(blob)
          downloadElement.href = href
          downloadElement.download = decodeURIComponent(fileName)
          downloadElement.style.display = 'none'
          document.body.appendChild(downloadElement)
          downloadElement.click()
          document.body.removeChild(downloadElement)
          window.URL.revokeObjectURL(href)
        }
      },
      getProvinceName(code) {
        const province = this.countryCityList.find(
          (item) => item.id === parseInt(code)
        )
        return province ? province.dropdownValue : ''
      },
      openFileUpload() {
        this.$refs.fileInput.value = null
        this.$refs.fileInput.click()
      },
      dateFormatOnlyDate(row,column, cellValue){
        return dayjs(cellValue).format('YYYY-MM-DD');
      },
      formatToDate(d) {
        var date = new Date(d)
        var YY = date.getFullYear() + '-'
        var MM =
          (date.getMonth() + 1 < 10
            ? '0' + (date.getMonth() + 1)
            : date.getMonth() + 1) + '-'
        var DD = date.getDate() < 10 ? '0' + date.getDate() : date.getDate()
        var hh =
          (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':'
        var mm =
          (date.getMinutes() < 10
            ? '0' + date.getMinutes()
            : date.getMinutes()) + ':'
        var ss =
          date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds()
        return YY + MM + DD
      },
      fetchData(flag) {
        let userInfo = JSON.parse(sessionStorage.getItem('userInfo'))
        this.loading = !this.tableLoding

        this.$apiFetch
          .post(`/acc/productAdmission/getRegionManagerEntryId`, {
            userId: userInfo.userInfo.id,
          })
          .then((res) => {
            if (res && res.code == 200) {
              const ids = res.data
              let queryParams = {}

              if (flag) {
                queryParams = this.currentQueryParams
              } else {
                const addListIfNotEmpty = (paramName, list) => {
                  if (list && Array.isArray(list) && list.length > 0) {
                    // queryParams[paramName] = JSON.stringify(list)
                    queryParams[paramName] = list
                  }
                  // 如果为空，则跳过，params对象中将不会有这个属性
                }

                addListIfNotEmpty(
                  'projectIdList',
                  this.productSearchOption.projectIdList
                )
                addListIfNotEmpty(
                  'productCodeList',
                  this.productSearchOption.productCodeList
                )
                addListIfNotEmpty('buList', this.productSearchOption.buList)
                addListIfNotEmpty(
                  'terminalCodeList',
                  this.productSearchOption.terminalNameList
                )
                addListIfNotEmpty(
                  'terminalProvinceList',
                  this.productSearchOption.terminalProvinceList
                )
                addListIfNotEmpty(
                  'actualAccessMethodList',
                  this.productSearchOption.actualAccessMethodList
                )
                addListIfNotEmpty('isLock', this.productSearchOption.isLock)
                addListIfNotEmpty(
                  'attentionLevelList',
                  this.productSearchOption.attentionLevelList
                )

                this.sortRules.forEach((rule) => {
                  queryParams[rule.prop + 'OrderBy'] =
                    rule.order === 'ascending' ? 'ASC' : 'DESC'
                })
              }
              queryParams.startTime = this.productSearchOption.startTime
              queryParams.endTime = this.productSearchOption.endTime
              queryParams.currentUser = userInfo.userInfo.id
              queryParams.currentPage = this.currentPage
              queryParams.pageSize = this.pageSize
              this.$apiFetch
                .post(
                  `/acc/productAdmission/getProductAdmissionList`,
                  queryParams
                )
                .then((res) => {
                  if (res && res.code == 200) {

                    // 处理数据：根据ids设置charge字段
                    this.productAccessList = res.data.list.map((item) => {
                      // 如果item的id在ids数组中，设置charge为true，否则为false
                      return {
                        ...item,
                        charge: ids.includes(item.id), // 假设item的id字段与ids中的值对应
                      }
                    })
                    this.totalOrder = res.data.total

                  }
                  this.loading = false
                  this.tableLoding = false
                  this.currentQueryParams = queryParams
                })
                .catch((error) => {
                  this.loading = false
                  this.tableLoding = false
                  this.currentQueryParams = queryParams
                  console.log(error)
                })
            }
          })
          .catch((error) => {})
      },
      handleSizeChange(val) {
        this.pageSize = val
        this.currentPage = 1
        this.fetchData(true)
      },
      handlePageChange(newPage) {
        this.currentPage = newPage
        this.fetchData(true)
      },
      formatDate(dateString) {
        if (dateString) {
          const date = new Date(dateString)
          const year = date.getFullYear()
          const month = String(date.getMonth() + 1).padStart(2, '0')
          const day = String(date.getDate()).padStart(2, '0')
          return `${year}-${month}-${day}`
        }
        return ''
      },
      formatDateTimeAll(originalDateString) {
        const date = new Date(originalDateString)
        const year = date.getFullYear()
        const month = String(date.getMonth() + 1).padStart(2, '0')
        const day = String(date.getDate()).padStart(2, '0')
        const hours = String(date.getHours()).padStart(2, '0')
        const minutes = String(date.getMinutes()).padStart(2, '0')
        const seconds = String(date.getSeconds()).padStart(2, '0')
        return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`
      },
      sortChange({ column, prop, order }) {
        this.tableLoding = true
        // this.sortRules = this.sortRules.filter((rule) => rule.prop !== prop)
        this.sortRules = []
        if (order) {
          this.sortRules.push({ prop, order })
        }
        this.currentPage = 1
        this.fetchData()
      },
      openProductAccessDetail(row, index) {
        this.dialogTitle =
          row.projectName + ' - ' + row.terminalName + ' | ' + row.productName
        this.project.projectName = row.projectName
        this.project.terminalName = row.terminalName
        this.project.productName = row.productName
        this.project.attentionLevel = row.attentionLevel
        this.project.KAHospitalOne = ''
        this.project.KAHospitalTwo = ''
        /* 取近两年的KA医院 */
        if (row.kahospital !== '无') {
          const hospitals = (row.kahospital || '').split(',').filter(Boolean)
          const [one = '', two = ''] = hospitals.slice(-2)
          this.project.KAHospitalOne = one
          this.project.KAHospitalTwo = two
        }
        if (row.terminalProvince) {
          this.$apiFetch.get(`/acc/productAdmission/getadmissionDealerByProvince`, {province: row.terminalProvince, type:'notAll'}).then((res) => {
            if (res && res.code == 200) {
              this.commercialTerritory2 = res.data
              this.dialogProductVisible = true
              this.selectData = row
              if (index === 'productAccessDetail') {
                this.showEdit = true
              } else if (index === 'productAccessInfo') {
                this.showEdit = false
              }
            }
          }).catch((error) => {
            console.log(error)
          })
        } else {
          this.dialogProductVisible = true
          this.selectData = row
          if (index === 'productAccessDetail') {
            this.showEdit = true
          } else if (index === 'productAccessInfo') {
            this.showEdit = false
          }
        }
        // this.$nextTick(() => {
        //   this.$refs.productDialog.$el.scrollTop = 0 // 滚动到顶部
        // })
      },
      closeDialogHandler(shouldClose, shouldReload) {
        if (shouldClose) {
          this.dialogProductVisible = false
          this.hasUnsavedChanges = false
        }
        if (shouldReload) {
          this.fetchData()
        }
      },
      handleDialogProductClose() {
        this.dialogProductVisible = false
        this.isClosing = false
        this.dialogProductVisible = false
        this.fetchData(true)
      },
      handleUnsavedChange(hasChanges) {
        this.hasUnsavedChanges = hasChanges
      },
      flexColumnWidth(data, prop, maxWidth) {
        console.log(prop, maxWidth)
        if (!data || data.length === 0) return 'auto'

        // 1. 获取该列所有单元格的内容（包括表头）
        const contents = data.map((item) =>
          item[prop] ? String(item[prop]) : ''
        )

        // 2. 计算最宽内容的宽度
        let maxContentWidth = 0
        // 创建一个临时的span元素来测量文本宽度
        const tempSpan = document.createElement('span')
        tempSpan.style.cssText =
          'position: absolute; visibility: hidden; white-space: nowrap;'
        document.body.appendChild(tempSpan)

        contents.forEach((content) => {
          tempSpan.textContent = content
          maxContentWidth = Math.max(maxContentWidth, tempSpan.offsetWidth)
        })

        document.body.removeChild(tempSpan)

        // 3. 加上一些内边距，并应用最大宽度限制
        const calculatedWidth = Math.min(maxContentWidth + 20, maxWidth) // 20px为预留padding

        return `${calculatedWidth}px`
      },
      getProductAdmissionListFilterOptions() {
        this.$apiFetch
          .post(
            `/acc/productAdmission/getProductAdmissionListFilterOptions`,
            {}
          )
          .then((res) => {
            if (res && res.code == 200) {

              /*this.admissionStatusMap = new Map(
                Object.entries(res.data.admissionStatusMap || {})
              )*/
              this.admissionStatusMap = res.data.admissionStatusMap
              this.recentSoStatusMap = new Map(
                Object.entries(res.data.recentSoStatusMap || {})
              )
              this.recentSoTypeMap = new Map(
                Object.entries(res.data.recentSoTypeMap || {})
              )
              this.reverseEntryReasonMap = new Map(
                Object.entries(res.data.reverseEntryReasonMap || {})
              )
              this.soValidatedMap = new Map(
                Object.entries(res.data.soValidatedMap || {})
              )
              this.commercialTerritory = res.data.commercialTerritory
              this.distributorInfor = res.data.distributorInfor
            }
          })
          .catch((error) => {
            this.filterLoding = false
            console.log(error)
          })
      },
      handleBeforeClose(done) {
        if (!this.showEdit && !this.isClosing) {
          this.$confirm('是否取消编辑？', '提示', {
            confirmButtonText: '是',
            cancelButtonText: '否',
            type: 'warning',
            distinguishCancelAndClose: true,
          })
            .then(() => {
              // 用户点击确定，放弃修改
              this.isClosing = true
              this.hasUnsavedChanges = false
              done() // 执行关闭
            })
            .catch((action) => {
              // 用户点击取消，不关闭弹框
              if (action === 'cancel') {
                // 取消关闭，什么都不做
                return
              }
            })
        } else {
          // 没有未保存的修改，直接关闭
          this.isClosing = true
          done()
        }
      },
      handleAttentionCommand (row, command) {
        row.attentionLevel = command
        let param = {
          id: row.id,
          attentionLevel: command
        }
        this.$apiFetch
          .post(`/acc/productAdmission/updateProductAdmissionAttentionLevel`, param)
          .then((res) => {
            if (res && res.code == 200) {
              this.$message.success('更新成功')
            }
          })
      },
      handleDetailAttention(project, command) {
        project.attentionLevel = command
        this.parantAttentionLevel = command
      },
      openImportDialog() {
        this.dialogVisibleImportConfirmedResult = true
      },
      closeImportDialog() {
        this.dialogVisibleImportConfirmedResult = false
      },
      downloadTemplate() {
        this.$apiFetch
          .PostFile(
            `/acc/productAdmission/downloadAdmissionTemplate`)
          .then((response) => {
            if (response && response.status === 200) {
              this.downloadStreamFileToLocal(response.data, '准入认定模版.xlsx')
              this.$message.success('下载模版成功!')
            }
            this.exportLoading = false
          })
          .catch((error) => {
            this.exportLoading = false
            console.log(error.message)
            this.$message.error('失败')
            // 处理错误
          })
      },
      handleFileChange(file, fileList) {
        this.fileList = fileList
        this.file = file.raw
      },
      handleRemove() {
        this.file = null
      },
      handleExceed() {
        this.$message.warning('只能上传一个文件')
      },
      batchUpload() {
        if (!this.file) {
          this.$message.warning('请先选择文件')
          return
        }

        const formData = new FormData()
        formData.append('file', this.file)
        formData.append('userEmail', JSON.parse(sessionStorage.getItem('userInfo')).userInfo.userAccount)

        this.saveLoading = true

        this.$apiFetch.postUploadExcel('/acc/productAdmission/uploadFile', formData)
          .then((response) => {
            if (response && response.status === 200) {
              this.$message.success('上传成功')
              this.file = null
              this.fileList = []
              this.$refs.upload.clearFiles()
              this.dialogVisibleImportConfirmedResult = false
            }
          }).catch(() => {
            this.$message.error('上传失败')
          }).finally(() => {
            this.saveLoading = false
          })
      },
    },
  }
</script>

<style scoped lang="scss">
  .locked-projects-alert {
    margin-bottom: 20px;
  }

  /*.icon-downloadProduct {
background-repeat: no-repeat;
height: 26px;
width: 26px;
background-position: 50%;
background-size: 20px;
border: none;
position: relative;
right: 0;
top: 0;
z-index: 0;
float: right;
margin-bottom: 10px;
}*/

  .button-bar {
    position: relative;
    padding-bottom: 25px;
    margin-bottom: 30px;

    .bayer-button {
      margin-bottom: 20px;
    }

    .icon-downloadProduct {
      right: 10px;
      top: 10px;
    }

    .icon-importAdmission {
      right: 50px;
      top: 10px;
    }
  }

  .search-button {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    margin-right: 10px;
    position: absolute;
    right: 10px;
    z-index: 9;

    .search-icon {
      width: 30px;
      grid-column: 3/4;
      grid-row: 3/3;
      background-color: #ff3162;
      background-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 10 10.1'%3E%3Cpath d='M9.2 1.7C7.9-.1 5.4-.5 3.5.8c-1.8 1.3-2.2 3.9-.9 5.7L0 9.1l1 1 2.6-2.6c1.4 1 3.3 1 4.7-.1 1.8-1.3 2.3-3.9.9-5.7zM5.9 7C4.3 7 3 5.7 3 4.1c0-1.6 1.3-2.9 2.9-2.9 1.6 0 2.9 1.3 2.9 2.9C8.8 5.7 7.5 7 5.9 7z' fill='%23fff'/%3E%3C/svg%3E");
      background-repeat: no-repeat;
      background-position: 63%;
      background-size: 13px;
      height: 26px;
      align-self: center;
      clip-path: polygon(25% 0, 100% 0, 100% 100%, 0 100%);
      cursor: pointer;
      border-radius: 0;
    }

    .search-icon.opened {
      background-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 17.1 17.1'%3E%3Cpath fill='%23fff' d='M17.1 1.1L16 0 8.5 7.5 1.1 0 0 1.1l7.5 7.4L0 16l1.1 1.1 7.4-7.5 7.5 7.5 1.1-1.1-7.5-7.5z'/%3E%3C/svg%3E");
    }
  }

  ::v-deep .el-table .locked-row td {
    background-color: #e6e6ea52 !important; /* 置灰背景色 */
    color: #c0c4cc !important; /* 置灰文字颜色 */
  }

  .custom-icon-btn {
    font-size: 18px;
  }

  .attention-empty {
    display: inline-flex;
    align-items: center;
    padding: 4px 14px;
    border-radius: 4px;
    font-size: 12px;
    color: #909399;
    background-color: #f5f7fa;
    cursor: pointer;
  }

  .attention-tag {
    display: inline-flex;
    align-items: center;
    padding: 4px 14px;
    border-radius: 4px;
    font-size: 12px;
    cursor: pointer;
  }

  .attention-tag.high {
    background-color: #fde2e2;
    color: #f56c6c;
  }

  .attention-tag.need {
    background-color: #fdf6ec;
    color: #e6a23c;
  }

  .dropdown-icon {
    margin-left: 4px;
    font-size: 12px;
  }

  /* 下拉里的小圆点 */
  .dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    margin-right: 6px;
  }

  .dot.red {
    background-color: #f56c6c;
  }

  .dot.yellow {
    background-color: #e6a23c;
  }

  .dot.gray {
    background-color: gray;
  }

  :deep(.el-dropdown-menu__item) {
    transition: background-color 0.2s, color 0.2s;
  }

  :deep(.el-dropdown-menu__item.item-gray:hover),
  :deep(.el-dropdown-menu__item.item-gray.is-active) {
    background-color: #f4f4f5 !important;
    color: #909399 !important;
  }

  :deep(.el-dropdown-menu__item.item-red:hover),
  :deep(.el-dropdown-menu__item.item-red.is-active) {
    background-color: #fef0f0 !important;
    color: #f56c6c !important;
  }

  :deep(.el-dropdown-menu__item.item-yellow:hover),
  :deep(.el-dropdown-menu__item.item-yellow.is-active) {
    background-color: #fdf6ec !important;
    color: #e6a23c !important;
  }


  .option-dot {
    display: inline-block;
    width: 8px;
    height: 8px;
    border-radius: 50%;
    margin-right: 6px;
    vertical-align: middle;
  }

  .option-dot.red {
    background-color: #ff430d;
  }

  .option-dot.yellow {
    background-color: #e6a23c;
  }

  .option-dot.gray {
    background-color: #909399;
  }

  .attention-dropdown[aria-disabled="true"] :deep(.el-dropdown-link),
  .attention-dropdown[aria-disabled="true"] :deep(span) {
    cursor: default !important;
  }

  .attention-dropdown[aria-disabled="true"] {
    cursor: default !important;
  }

  .attention-dropdown[aria-disabled="true"] :deep(.el-dropdown-link),
  .attention-dropdown[aria-disabled="true"] :deep(.el-dropdown-selfdefine) {
    pointer-events: none;
    cursor: default !important;
  }

  .el-table__fixed-right {
    background: #fff;
    z-index: 3;
  }

  .el-table__fixed-right::before {
    content: '';
    position: absolute;
    left: -1px;
    top: 0;
    width: 1px;
    height: 100%;
    background-color: #ebeef5;
  }

  .attention-tag-wrapper {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-width: 58px;
    height: 4vh;
    padding: 0 4px;
    border-radius: 4px;
    cursor: pointer;
  }

  .attention-tag-wrapper.high {
    background-color: #fde2e2;
    color: #f56c6c;
  }

  .attention-tag-wrapper.need {
    background-color: #fff7e6;
    color: #e6a23c;
  }

  .attention-tag-wrapper.empty {
    background-color: transparent;
    color: transparent;
  }

  .attention-text {
    white-space: nowrap;
    font-size: 10px;
    line-height: 1;
    margin-left: 4px;
  }

  .attention-tag-wrapper.empty .attention-text {
    visibility: hidden;
  }

  .dropdown-icon {
    margin-left: 4px;
    font-size: 12px;
    color: #909399;
  }

  .custom-flex-row {
    display: flex;
    gap: 10px;
    margin-bottom: 10px;
  }

  .custom-flex-row > .el-col {
    flex: 0 0 calc((100% - 16px * 4) / 5);
  }

  .ka-tag{
    display: inline-block;
    padding: 4px 10px;
    border-radius: 2px;
    font-size: 12px;
    color: #fff;
    margin-right: 8px;
    line-height: 10px;
  }

  .ka-tag.blue{
    background-color: #2b7bbb;
  }

  .ka-tag.yellow{
    background-color: #f5b400;
  }

  .uploadsection {
    width: 100%;
  }

  .uploadsection ::v-deep(.el-upload) {
    width: 100%;
  }

  .uploadsection ::v-deep(.el-upload-dragger) {
    width: 100%;
  }

  .el-form--label-top {
    padding-right: 0 !important;
  }

  .downloadTemplateBtn {
    color: #00a6ff;
    cursor: pointer;
    font-weight: bold;
  }
</style>
