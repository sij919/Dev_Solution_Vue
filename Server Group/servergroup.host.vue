<template>
  <div class="menu-container">
    <div class="button-container">
      <ev-button
        class="btn"
        type="ghost"
        size="small"
        @click="clickSaveBtn"
      >
        <ev-icon cls="ei-save"/>
        Group Save
      </ev-button>
      <ev-button
          class="btn"
          type="ghost"
          size="small"
          @click="clickDelBtn"
      >
        <ev-icon cls="ei-save"/>
        Group Del
      </ev-button>
      <ev-button
        class="btn"
        type="ghost"
        size="small"
        @click="clickLimitSaveBtn"
      >
        <ev-icon cls="ei-gear"/>
        Limit Save
      </ev-button>
      <ev-button
          class="btn"
          type="ghost"
          size="small"
          @click="clickLimitDelBtn"
      >
        <ev-icon cls="ei-gear"/>
        Limit Del
      </ev-button>
    </div>
    <div class="detail-container">
      <div class="row" style="height: 483px; border: 1px solid #474B57;">
        <ev-grid
          :columns="columns"
          :rows="hostList"
          :option="gridOptions"
          :selected.sync="selectedRow"
        >
          <template v-slot:check="item">
            <ev-checkbox
              v-model="item.row[item.cellIndex]"
              :value="item.row[2]"
              type="square"
              after-type="check"
              size="small"
            />
          </template>
        </ev-grid>
      </div>
    </div>
    <!------------------------ Service / Threshold Alert Popup ------------------------>
    <ev-window
      :width="350"
      :height="290"
      :modal="true"
      :is-show.sync="onOpenLimitSetting"
      :resizable="false"
      :maximizable="false"
      class="setting-window"
      title="Limit Setting"
      close-type="destroy"
    >
      <div class="setting-container">
        <div class="row">
          <ev-label
            value="CPU"
            class="setting-menu"
          />
          <ev-text-field
            v-model="cpuGauge"
            class="setting-gauge"
          />
        </div>
        <div class="row">
          <ev-label
            value="Memory"
            class="setting-menu"
          />
          <ev-text-field
            v-model="memoryGauge"
            class="setting-gauge"
          />
        </div>
        <div class="row">
          <ev-label
            value="Disk"
            class="setting-menu"
          />
          <ev-text-field
            v-model="diskGauge"
            class="setting-gauge"
          />
        </div>
        <div class="btn-container">
          <ev-button
            type="primary"
            class="btn"
            @click="onSaveLimit"
          >
            OK
          </ev-button>
          <ev-button
            type="primary"
            @click="onCancel"
          >
            Cancel
          </ev-button>
        </div>
      </div>
    </ev-window>
    <!------------------------ Service / Threshold Alert Popup ------------------------>
  </div>
</template>
<script>
  import { makeRows } from 'main/utils/helpers';
  import { request } from 'main/utils/api';
  import { API } from 'main/utils/define';

  export default {
    name: 'ServerGroupMenu',
    props: {
      data: {
        type: Array,
        default: () => [],
      },
      groupId: {
        type: Number,
        default: 0,
      },
    },
    data() {
      return {
        columns: [
          { caption: 'Group ID', field: 'group_id', type: 'string', hide: true },
          { caption: '', field: 'check', type: 'boolean', render: { use: true }, width: 50 },
          { caption: 'Host Name', field: 'host_name', type: 'string', hide: true },
          { caption: 'Host Name', field: 'host_name', type: 'string' },
          { caption: 'Group Name', field: 'group_name', type: 'string' },
          { caption: 'Limit', field: 'limit_value', type: 'string', align: 'center' },
        ],
        gridOptions: {
          adjust: true,
          useSelected: true,
          columnWidth: 100,
          scrollWidth: 10,
        },
        selectedRow: [],
        onOpenLimitSetting: false,
        hostName: '',
        cpuGauge: '85',
        memoryGauge: '85',
        diskGauge: '85',
        selectedHost: [],
      };
    },
    computed: {
      hostList() {
        return makeRows(this.data, this.columns);
      },
    },
    methods: {
      clickSaveBtn() {
        const check = '.ev-checkbox-input:checked';
        const selectedEls = document.querySelectorAll(check);

        if (this.groupId === 0) {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Please select the group.',
            type: 'info',
            showCancelButton: false,
          });
          return;
        }

        const list = [];
        selectedEls.forEach((el) => {
          list.push(el.value);
        });

        if (list.length === 0) {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Please select the host.',
            type: 'info',
            showCancelButton: false,
          });
          return;
        }

        const obj = {
          group_id: this.groupId,
          host_list: list,
        };
        this.reqSave(obj);
      },
      reqSave(data) {
        request({
          method: 'post',
          url: API.SETTINGS.SERVERGROUP.SAVE_SERVER_GROUP_HOST,
          data,
        }).then(() => {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Saved',
            type: 'success',
            showCancelButton: false,
          });
          this.$emit('on-save-refresh', data);
        }).catch(({ message }) => {
          this.showMsgBox({
            title: 'Error',
            message,
            type: 'error',
            showCancelButton: false,
          });
        });
      },
      clickDelBtn() {
        const check = '.ev-checkbox-input:checked';
        const selectedEls = document.querySelectorAll(check);

        const list = [];
        selectedEls.forEach((el) => {
          list.push(el.value);
        });

        if (list.length === 0) {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Please select the host.',
            type: 'info',
            showCancelButton: false,
          });
          return;
        }

        const obj = {
          host_list: list,
        };
        this.reqDel(obj);
      },
      reqDel(data) {
        request({
          method: 'post',
          url: API.SETTINGS.SERVERGROUP.DELETE_SERVER_GROUP_HOST,
          data,
        }).then(() => {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Saved',
            type: 'success',
            showCancelButton: false,
          });
          this.$emit('on-save-refresh', data);
        }).catch(({ message }) => {
          this.showMsgBox({
            title: 'Error',
            message,
            type: 'error',
            showCancelButton: false,
          });
        });
      },
      clickLimitSaveBtn() {
        const check = '.ev-checkbox-input:checked';
        const selectedEls = document.querySelectorAll(check);

        const list = [];
        selectedEls.forEach((el) => {
          list.push(el.value);
        });

        this.selectedHost = list;

        if (list.length === 0) {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Please select the host.',
            type: 'info',
            showCancelButton: false,
          });
          return;
        }

        this.clickSettingBtn();
      },
      clickSettingBtn() {
        this.hostName = 'All Host';
        this.cpuGauge = '85';
        this.memoryGauge = '85';
        this.diskGauge = '85';
        this.onOpenLimitSetting = true;
      },
      onCancel() {
        this.onOpenLimitSetting = false;
      },
      onSaveLimit() {
        if (this.cpuGauge === '' || this.memoryGauge === '' || this.diskGauge === '') {
          const message = 'Input the Limit';
          this.showMsgBox({
            title: 'Error',
            message,
            type: 'error',
            showCancelButton: false,
          });
          return;
        }

        const data = {
          host_list: this.selectedHost,
          cpu_limit: this.cpuGauge,
          memory_limit: this.memoryGauge,
          disk_limit: this.diskGauge,
        };

        request({
          method: 'post',
          url: API.SETTINGS.SERVERGROUP.SAVE_SERVER_GROUP_LIMIT,
          data,
        }).then(() => {
          this.showMsgBox({
            title: 'Info',
            message: 'Saved',
            type: 'info',
            showCancelButton: false,
          });
          this.$emit('on-save-refresh', data);
        }).catch(({ message }) => {
          this.showMsgBox({
            title: 'Error',
            message,
            type: 'error',
            showCancelButton: false,
          });
        }).finally(() => {
          this.onOpenLimitSetting = false;
        });
      },
      clickLimitDelBtn() {
        const check = '.ev-checkbox-input:checked';
        const selectedEls = document.querySelectorAll(check);

        const list = [];
        selectedEls.forEach((el) => {
          list.push(el.value);
        });

        if (list.length === 0) {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Please select the host.',
            type: 'info',
            showCancelButton: false,
          });
          return;
        }

        const obj = {
          host_list: list,
        };
        this.reqLimitDel(obj);
      },
      reqLimitDel(data) {
        request({
          method: 'post',
          url: API.SETTINGS.SERVERGROUP.DELETE_SERVER_GROUP_LIMIT,
          data,
        }).then(() => {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Saved',
            type: 'success',
            showCancelButton: false,
          });
          this.$emit('on-save-refresh', data);
        }).catch(({ message }) => {
          this.showMsgBox({
            title: 'Error',
            message,
            type: 'error',
            showCancelButton: false,
          });
        });
      },
    },
  };

</script>
<style lang="scss">
  $title-button-height: 85px;
  $menu-margin-right: 25px;
  $btn-field-height: 39px;

  .menu-container {
    flex: 1;
    height: 100%;
    margin-left: 10px;
    padding-top: 37.4px;
    .button-container {
      display: flex;
      height: 40px;
      align-items: center;
      justify-content: flex-end;
    }
  }

  .btn {
    margin-right: 5px;
    .ei {
      margin-right: 5px;
    }
    &:last-child {
      margin-right: 0;
    }
  }

  .table-body td {
    border: none;
    white-space: pre;
    cursor: pointer;
  }

  .detail-container, .setting-container {
    width: 100%;
    .row {
      display: flex;
      margin-bottom: 2px;
      align-items: center;
    }
  }

  .setting-container {
    .setting-title {
      font-size: 14px;
      margin-bottom: 30px;
    }
    .setting-menu {
      width: 150px;
      color: rgba(255,255,255,0.7);
      text-align: left !important;
    }
    .setting-gauge {
      display: inline-block;
      width: 100%;
      height: 28px !important;
      .ev-input-text {
        text-align: right !important;
      }
    }
    .btn-container {
      display: flex;
      justify-content: flex-end;
      margin-top: 32px;
      .btn {
        margin-right: 5px;
        &:last-child {
          margin-right: 0;
        }
      }
    }
  }
</style>
