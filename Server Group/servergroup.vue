<template>
  <div class="container">
    <!-- 서버 그룹 리스트 컴퍼넌트 영역 -->
    <div style="width: 40%;">
      <server-group-list
        ref="servergroupList"
        :data="groupList"
        @on-save="onSaveServerGroup"
        @select-row="selectServerGroup"
      />
    </div>
    <!-- 서버 호스트 리스트 컴퍼넌트 영역 -->
    <server-host-list
      :data="hostList"
      :group-id="groupId"
      @on-save-refresh="handleSaveRefresh"
    />
  </div>
</template>

<script>
  /**
   * @file : servergroup.vue(메인)
   *
   * @structure :
   * servergroup.vue - servergroup.list.vue - servergroup.list.manage.vue
   *                 - servergroup.host.vue
   *
   * @logic :
   * 1. 서버 그룹 리스트를 조회한다.
   * 2. 호스트 리스트를 조회한다.
   *
   * @method
   * 1. getGroupList : 서버그룹 리스트를 조회한다.
   * 2. getHostList : 호스트 리스트를 조회한다.
   * 3. onSaveServerGroup : 서버그룹 리스트 갱신
   * 3. selectServerGroup : 서버그룹 선택시 하위 컴퍼넌트로 데이터 전달
   * 3. handleSaveRefresh : 세이브 후 호스트 컴퍼넌트 리프레시
   */
  import { request } from 'main/utils/api';
  import { API } from 'main/utils/define';
  import ServerGroupList from './servergroup.list';
  import ServerHostList from './servergroup.host';

  export default {
    name: 'ServerGroup',
    components: {
      ServerGroupList,
      ServerHostList,
    },
    data() {
      return {
        groupList: [],
        hostList: [],
        groupId: 0,
        checked: [],
      };
    },
    mounted() {
      this.getGroupList();
      this.getHostList();
    },
    methods: {
      getGroupList(type) {
        request({
          method: 'get',
          url: API.SETTINGS.SERVERGROUP.GET_SERVER_GROUP_LIST,
          params: {
            page: 1,
            start: 0,
            limit: 25,
          },
          headers: {
            'Content-Type': 'application/json',
          },
        }).then((data) => {
          const res = data.map.result;
          if (res.length !== 0) {
            for (let i = 0; i < res.length; i++) {
              this.groupList.push(res[i]);
            }
          }
          if (type === 'Add') {
            this.$nextTick(() => {
              this.$refs.servergroupList.selectLastRow();
            });
          } else if (type === 'Modify') {
            this.$nextTick(() => {
              this.getHostList();
            });
          }
        }).catch(({ message }) => {
          this.showMsgBox({
            title: 'Error',
            message,
            type: 'error',
            showCancelButton: false,
          });
        });
        this.groupList = [];
      },
      getHostList() {
        const params = {};
        request({
          method: 'post',
          url: API.SETTINGS.SERVERGROUP.GET_SERVER_GROUP_HOST,
          params,
        }).then((data) => {
          const res = data.map.result;
          this.hostList = [];

          if (res.length !== 0) {
            for (let i = 0; i < res.length; i++) {
              this.hostList.push(res[i]);
            }
          }
        }).catch(({ message }) => {
          this.showMsgBox({
            title: 'Error',
            message,
            type: 'error',
            showCancelButton: false,
          });
        });
      },
      onSaveServerGroup(type) {
        this.getGroupList(type);
      },
      selectServerGroup(val) {
        this.groupId = val.group_id;
      },
      handleSaveRefresh() {
        this.getHostList();
      },
    },
  };
</script>

<style lang="scss" scoped>
  .container {
    display: flex;
    width: 100%;
    height: 100%;
    flex-direction: row;
  }

</style>
