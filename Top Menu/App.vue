<template>
  <div
    :class="{
      flamingo: true,
      expand: isExpanded,
      'no-nav': !mainMenu,
    }"
  >
    <Nav
      v-if="mainMenu"
      :selected-menu="currentMenu"
      :main-menu="mainMenu"
      :is-expand="isExpanded"
      class="menu"
    />
    <router-view
      :class="{
        contents: true,
        dashboard: isDashboard,
      }"
    >
      <h2>{{ $route.name }}</h2>
      <div class="header-icon-wrap">
        <ev-icon
          :cls="`header-icon ${isExpanded ? 'ei-s-double-left' : 'ei-s-double-right'}`"
          @click="isExpanded = !isExpanded"
        />
        <img
          :src="require(`./assets/images/topmenu/${licenseIcon}.png`)"
          style="position: relative; margin-right: 13px;"
          @click="onLicenseInfo"
        />
        <span style="position: relative;"
              @click="onThresholdAlert">
          <img
              :src="require(`./assets/images/topmenu/${thresholdAlertIcon}.png`)"
              style="margin-right: 13px;"
          />
          <span v-if="thresholdCnt > 0"
                class="service-alert-count">{{ thresholdCnt }}</span>
        </span>
        <span style="position: relative;"
              @click="onServiceAlert">
          <img
            :src="require(`./assets/images/topmenu/${serviceAlertIcon}.png`)"
            style="margin-right: 13px;"
          />
          <span v-if="serviceCnt > 0"
                class="service-alert-count">{{ serviceCnt }}</span>
        </span>
        <ev-icon
          cls="header-icon ei-logout"
          @click="onLogout"
        />
      </div>
    </router-view>
    <!---------------------------- License Infomation Popup --------------------------->
    <ev-popup
            :width="800"
            :height="400"
            :modal="false"
            :is-show.sync="showLicenseInfo"
            :resizable="false"
            :maximizable="false"
            class="alert-window license"
            title="Infomation"
            close-type="destroy"
    >
      <div class="detail-container">
        <div class="wrap"
             style="padding-bottom: 5px;
                    margin-bottom: 20px;
                    border-bottom: 1px solid;">
          <h2 class="title">
            <div class="icon">
              <ev-icon cls="ei-arrow-right2"/>
            </div>
            Lisense
          </h2>
          <div class="row">
            <ev-label
              class="label"
              value="Expiration Date"
              text-align="left"
              style="width: 150px;"
            />
            <ev-label
              :value="licenseInfo"
              style="width: 100%;"
              text-align="left"
            />
          </div>
          <div class="row">
            <ev-label
              class="label"
              value="Server ID"
              text-align="left"
              style="width: 150px;"
            />
            <textarea
              :value="serverId"
              style="width: 100%; height: 50px;"
              text-align="left"
              disabled
            />
          </div>
        </div>
        <div class="wrap">
          <h2 class="title">
            <div class="icon">
              <ev-icon cls="ei-arrow-right2"/>
            </div>
            Service
          </h2>
          <div class="row">
            <ev-grid
              :columns="serviceInfoColumns"
              :rows="serviceInfoList"
              :option="gridOptions"
              style="margin-bottom: 30px;"
            >
            </ev-grid>
          </div>
          <div style="text-align: right;
                      padding-bottom: 10px;">Open Source License
            <a style="color: #2F77FC; cursor: pointer;" @click="onClick()">Apache License 2.0</a>
          </div>
        </div>
      </div>
    </ev-popup>
    <!---------------------------- License Infomation Popup --------------------------->
    <!------------------------------ Service Alert Popup ------------------------------>
    <ev-popup
      :width="450"
      :height="260"
      :modal="false"
      :is-show.sync="showServiceAlert"
      :resizable="false"
      :maximizable="false"
      class="alert-window service"
      title="Service Alert"
    >
      <div class="detail-container">
        <ev-grid
          :columns="serviceAlertColumns"
          :rows="serviceAlertList"
          :option="gridOptions"
        >
          <template v-slot:status="item">
            <img
              :src="icons[item.value]"
              class="state-icon"
              alt="status"
            />
          </template>
        </ev-grid>
      </div>
    </ev-popup>
    <!-------------------------------- Service Alert Popup -------------------------------->
    <!------------------------------ Service Threshold Popup ------------------------------>
    <ev-popup
        :width="950"
        :height="400"
        :modal="false"
        :is-show.sync="showThresholdAlert"
        :resizable="false"
        :maximizable="false"
        class="alert-window threshold"
        title="Threshold Alert"
    >
      <div class="detail-container">
        <ev-grid
            :columns="thresholdAlertColumns"
            :rows="thresholdAlertList"
            :option="gridOptions"
        >
          <template v-slot:status="item">
            <img
                :src="icons[item.value]"
                class="state-icon"
                alt="status"
            />
          </template>
        </ev-grid>
      </div>
    </ev-popup>
    <!------------------------------ Service Threshold Popup ------------------------------>
  </div>
</template>

<script>
  import { API, BASE_URL } from 'main/utils/define';
  import { request, ws } from 'main/utils/api';
  import manual from 'main/assets/manual/manual.pdf';
  import Nav from './nav';

  export default {
    name: 'App',
    components: {
      Nav,
    },
    data() {
      return {
        eleHeight: 0,
        mainMenu: '',
        currentMenu: '',
        isExpanded: true,
        isLoading: false,
        licenseIcon: 'user',
        thresholdAlertIcon: 'alert',
        serviceAlertIcon: 'bell',
        showLicenseInfo: false,
        showThresholdAlert: false,
        showServiceAlert: false,
        serverId: '',
        licenseInfo: '',
        thresholdCnt: 0,
        thresholdAlertColumns: [
          { caption: 'Status', field: 'status', type: 'string', width: 70, render: { use: true } },
          { caption: 'Host Name', field: 'host_name', type: 'string', width: 200, align: 'center' },
          { caption: 'Alert', field: 'alert_label', type: 'string', width: 130, align: 'center' },
          { caption: 'Text', field: 'alert_text', type: 'string', align: 'left' },
          { caption: 'Time', field: 'collection_time', type: 'string', width: 130, align: 'center' },
        ],
        thresholdAlertList: [],
        serviceInfoColumns: [
          { caption: 'Name', field: 'service_name', type: 'string', width: 100, align: 'center' },
          { caption: 'Version', field: 'service_version', type: 'string', width: 150, align: 'center' },
          { caption: 'Description', field: 'service_desc', type: 'string', align: 'left' },
        ],
        serviceInfoList: [],
        serviceCnt: 0,
        serviceAlertColumns: [
          { caption: 'Id', field: 'id', type: 'string', hide: true },
          { caption: 'Status', field: 'status', type: 'string', width: 70, render: { use: true } },
          { caption: 'Operations', field: 'operations_str', type: 'string', align: 'left' },
          { caption: 'Start Time', field: 'start_time', type: 'string', align: 'center' },
          { caption: 'End Time', field: 'end_time', type: 'string', align: 'center' },
        ],
        serviceAlertList: [],
        alertColumns: [],
        alertList: [],
        gridOptions: {
          adjust: true,
          rowHeight: 35,
          columnWidth: 100,
          scrollWidth: 10,
        },
        icons: {
          /* eslint-disable */
          SUCCESS: require('main/assets/images/service/check.png'),
          RUNNING: require('main/assets/images/service/running.png'),
          FAIL: require('main/assets/images/service/warning.png'),
          CRITICAL: require('main/assets/images/service/warning.png'),
          /* eslint-enable */
        },
        minuteTimer: null,
      };
    },
    computed: {
      isDashboard() {
        const dashboardMenu = [
          'Dashboard',
          'HDFS',
          'YARN',
          'NiFi',
        ];
        return dashboardMenu.includes(this.currentMenu);
      },
    },
    watch: {
      $route(to) {
        if (this.mainMenu !== to.matched[0].name) {
          this.mainMenu = to.matched[0].name;
        }

        if (this.currentMenu !== to.name) {
          this.currentMenu = to.name;
        }
      },
    },
    mounted() {
      this.serviceAlertCount();
      this.thresholdCntCount();
    },
    created() {
      ws.connect(BASE_URL, API.WEBSOCKET.SUBSCRIBES);
      this.minuteTimer = setInterval(this.serviceAlertCount, 10000);
    },
    methods: {
      onLicenseInfo() {
        this.isLoading = true;
        request({
          method: 'get',
          url: API.LICENSE.GET_SERVERID,
        }).then((data) => {
          this.serverId = data.map.serverId;
        });
        request({
          method: 'get',
          url: API.LICENSE.GET_LICENSEINFO,
          headers: {
            'Content-Type': 'application/json',
          },
        }).then((data) => {
          this.licenseInfo = `${data.map.goodBeforeDate} (${data.map.days}일 전)`;
        });
        request({
          method: 'get',
          url: API.SERVICEALERT.GET_SERVICES_INFO,
        }).then((data) => {
          const result = [];
          if (data.map.result.length !== 0) {
            for (let i = 0; i < data.map.result.length; i++) {
              const row = data.map.result[i];
              result.push([
                row.service_name,
                row.service_version,
                row.service_desc,
              ]);
            }
          }
          this.serviceInfoList = result;
        });
        if (!this.showLicenseInfo) {
          this.showLicenseInfo = true;
          this.showThresholdAlert = false;
          this.showServiceAlert = false;
          this.licenseIcon = 'user_on';
          this.thresholdAlertIcon = 'alert';
          this.serviceAlertIcon = 'bell';
        } else {
          this.showLicenseInfo = false;
          this.licenseIcon = 'user';
        }
      },
      serviceAlertCount() {
        request({
          method: 'get',
          url: API.SERVICEALERT.GET_FLAG_FALSE_COUNT,
          headers: {
            'Content-Type': 'application/json',
          },
        }).then((count) => {
          this.serviceCnt = count;
        });
      },
      thresholdCntCount() {
        request({
          method: 'get',
          url: API.SERVICEALERT.GET_THRESHOLD_FALSE_COUNT,
          headers: {
            'Content-Type': 'application/json',
          },
        }).then((count) => {
          this.thresholdCnt = count;
        });
      },
      onThresholdAlert() {
        this.thresholdCntCount();

        request({
          method: 'get',
          url: API.SERVICEALERT.GET_THRESHOLD_ALERT_LIST,
          headers: {
            'Content-Type': 'application/json',
          },
        }).then((data) => {
          const result = [];
          if (data.map.result.length !== 0) {
            for (let i = 0; i < data.map.result.length; i++) {
              const row = data.map.result[i];
              result.push([
                row.status,
                row.host_name,
                row.alert_label,
                row.alert_text,
                row.collection_time,
              ]);
            }
          }
          this.thresholdAlertList = result;
        });

        if (!this.showThresholdAlert) {
          this.showThresholdAlert = true;
          this.showLicenseInfo = false;
          this.showServiceAlert = false;
          this.licenseIcon = 'user';
          this.thresholdAlertIcon = 'alert_on';
          this.serviceAlertIcon = 'bell';
        } else {
          this.showThresholdAlert = false;
          this.thresholdAlertIcon = 'alert';
        }
      },
      onServiceAlert() {
        this.serviceAlertCount();
        request({
          method: 'get',
          url: API.SERVICEALERT.GET_SERVICE_ALERT_LIST,
          headers: {
            'Content-Type': 'application/json',
          },
        }).then((data) => {
          const result = [];
          if (data.map.result.length !== 0) {
            for (let i = 0; i < data.map.result.length; i++) {
              const row = data.map.result[i];
              result.push([
                row.id,
                row.status,
                row.operations_str,
                row.start_time,
                row.end_time,
              ]);
            }
          }
          this.serviceAlertList = result;
        });
        if (!this.showServiceAlert) {
          this.showLicenseInfo = false;
          this.showServiceAlert = true;
          this.showThresholdAlert = false;
          this.licenseIcon = 'user';
          this.thresholdAlertIcon = 'alert';
          this.serviceAlertIcon = 'bell_on';
        } else {
          this.showServiceAlert = false;
          this.serviceAlertIcon = 'bell';
        }
      },
      onFlamingoDown() {
        const iframe = document.createElement('iframe');
        iframe.setAttribute('id', 'agentFrame');
        iframe.setAttribute('style', 'visibility:hidden;width:0px;height:0px;');
        iframe.setAttribute('src', `${API.HDFS.BROWSER_GET_AGENT_DOWNLOAD_PATH}`);
        document.body.appendChild(iframe);
        setTimeout(() => {
          document.getElementById('agentFrame').remove();
        }, 5000, iframe);
      },
      onManual() {
        window.open(manual);
      },
      onLogout() {
        request({
          method: 'POST',
          url: API.LOGOUT,
        }).then(() => {
          location.href = `http://${location.host}/login`;
        });
      },
      onClick() {
        window.open(
          API.LICENSE.GET_PATH,
          '_blank',
          `height=${screen.height + 100}, width=${screen.width + 100}, fullscreen=yes`);
        return false;
      },
    },
  };
</script>

<style lang="scss">
  @import './assets/css/main';
  $btn-field-height: 39px;
  $btn-area-margin-top: 10px;

  .alert-window {
    top: 55px !important;
    left: auto !important;
    right: 115px;
    .ev-window-body-area {
      padding: 20px !important;
    }
    &.service {
      right: 55px !important;
    }
    &.license {
      right: 120px !important;
    }
    &.threshold {
      right: 90px !important;
    }
  }

  .flamingo {
    position: absolute;
    display: flex;
    width: 100%;
    height: 100%;
    padding-left: 80px;
    flex-direction: row;
    color: white;

    &.expand {
      padding-left: 250px;
    }
    &.no-nav {
      padding-left: 0;
    }
  }

  .menu {
    position: absolute;
    left: 0;
    height: 100%;
  }

  .contents {
    width: 100%;
    height: 100%;
    background: #232A32;

    &.dashboard {
      background: #181B20;
    }

    .header-icon-wrap {
      color: #ADB3B8;
      font-size: 18px;
      cursor: pointer;

      .header-icon {
        margin-right: 13px;

        &:last-child {
          margin-right: 0;
        }
      }
    }
  }

  .detail-container {
    width: 100%;
    height: 100%;
    z-index: 10000;

    .row {
      display: flex;
      margin-bottom: 15px;

      .vscroll-spacer {
        height: 0 !important;
      }
      .state-icon {
        margin-right: 15px;
      }
    }
    .icon {
      width: 15px;
      height: 15px;
      background-color: #0091FF;
      border-radius: 50%;
      padding: 2px;
      font-size: 12px;
      margin-right: 8px;
    }
    .title {
      display: flex;
      align-items: center;
      margin-bottom: 20px;
      font-size: 15px;
    }
  }

  .service-alert-count {
    position: absolute;
    top: -5px;
    right: 2px;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background-color: #E02020;
    color: #FFFFFF;
    text-align: center;
    vertical-align: middle;
    font-size: 12px;
    border: 2px solid #181B20;
  }
  .btn-area {
    height: $btn-field-height;
    margin: #{$btn-area-margin-top} 0 0 0;
    display: flex;
    justify-content: flex-end;
    align-items: flex-end;

    .ev-btn {
      width: 100px;
      margin: 0 10px;
    }
  }
</style>
