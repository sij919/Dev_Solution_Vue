<template>
  <div class="list-container">
    <h2 class="title">Group List</h2>
    <!-- 버튼 영역 -->
    <div class="button-container">
      <ev-button
        class="btn"
        type="ghost"
        size="small"
        @click="clickAddBtn"
      >
        <ev-icon cls="ei-plus"/>
        Add
      </ev-button>
      <ev-button
        class="btn"
        type="ghost"
        size="small"
        @click="clickModifyBtn"
      >
        <ev-icon cls="ei-user-check"/>
        Modify
      </ev-button>
      <ev-button
        class="btn"
        type="ghost"
        size="small"
        @click="clickRemoveBtn"
      >
        <ev-icon cls="ei-minus"/>
        Remove
      </ev-button>
    </div>
    <!-- 그리드 영역 -->
    <div class="grid-container">
      <ev-grid
        :columns="columns"
        :rows="rows"
        :option="option"
        :selected.sync="selectedRow"
      />
    </div>
    <!-- 팝업 영역 -->
    <ev-window
      :width="400"
      :height="180"
      :modal="true"
      :maximizable="false"
      :is-show.sync="showServerGroupManage"
      title="ServerGroup Manage"
      close-type="destroy"
    >
      <!-- 서버 그룹 관리 컴퍼넌트 -->
      <server-group-manage
        :type="type"
        :selected="selectedObj"
        @on-save="onSave"
        @on-cancel="showServerGroupManage=false"
      />
    </ev-window>
  </div>
</template>
<script>
  /**
   * @file : servergroup.list.vue(서버그룹 리스트 뷰)
   *
   * @structure :
   * servergroup.list.vue - servergroup.list.manage.vue
   *
   * @logic :
   * 1. 그리드를 선택했을때 메인컴퍼넌트를 거쳐 호스트 리스트 컴퍼넌트로 데이터를 전달한다.
   * 2. 선택된 그리드에서 하위 컴퍼넌트로 데이터를 전달한다.
   * 3. 팝업을 호출한다.
   *
   * @method
   * 1. setRowObj : 선택된 그리드의 데이터 저장
   * 2. selectLastRow : 그리드의 마지막 데이터 저장
   * 3. onSave : 세이브 이벤트 전달
   * 4. checkSelectedRow : 체크여부 확인
   * 5. reqRemove : 삭제 api 호출
   * 6. clickAddBtn : 등록 팝업
   * 7. clickModifyBtn : 수정 팝업
   * 8. clickRemoveBtn : 삭제 확인
   */
  import { makeRows } from 'main/utils/helpers';
  import { API } from 'main/utils/define';
  import { request } from 'main/utils/api';
  import ServerGroupManage from './servergroup.list.manage';

  export default {
    name: 'ServerGroupList',
    components: {
      ServerGroupManage,
    },
    props: {
      data: {
        type: Array,
        default: () => [],
      },
    },
    data() {
      return {
        columns: [
          { caption: 'Group Name', field: 'group_name', type: 'string' },
          { caption: 'Group ID', field: 'group_id', type: 'string', hide: true },
          { caption: 'Description', field: 'group_descript', type: 'string' },
        ],
        option: {
          adjust: true,
        },
        showServerGroupManage: false,
        selectedRow: [],
        type: '',
      };
    },
    computed: {
      rows() {
        return makeRows(this.data, this.columns);
      },
      selectedObj() {
        return this.setRowObj(this.selectedRow);
      },
    },
    watch: {
      selectedRow(val) {
        this.$emit('select-row', this.setRowObj(val));
      },
    },
    methods: {
      setRowObj(arr) {
        return {
          group_name: arr[0],
          group_id: arr[1],
          group_descript: arr[2],
        };
      },
      selectLastRow() {
        this.selectedRow = this.rows[this.rows.length - 1];
      },
      onSave() {
        this.$emit('on-save', this.type);
        this.showServerGroupManage = false;
      },
      checkSelectedRow() {
        if (!this.selectedRow.length) {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Please select the row.',
            type: 'info',
            showCancelButton: false,
          });
          return false;
        }
        return true;
      },
      reqRemove() {
        request({
          method: 'post',
          url: API.SETTINGS.SERVERGROUP.DELETE_SERVER_GROUP,
          headers: {
            'Content-Type': 'application/json',
          },
          params: {
            group_id: this.setRowObj(this.selectedRow).group_id,
          },
        }).then(() => {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Deleted',
            type: 'success',
            showCancelButton: false,
          });
          this.selectedRow = [];
          this.$emit('on-save');
        }).catch(({ message }) => {
          this.showMsgBox({
            title: 'Error',
            message,
            type: 'error',
            showCancelButton: false,
          });
        });
      },
      clickAddBtn() {
        this.type = 'Add';
        this.showServerGroupManage = true;
      },
      clickModifyBtn() {
        if (this.checkSelectedRow()) {
          this.type = 'Modify';
          this.showServerGroupManage = true;
        }
      },
      clickRemoveBtn() {
        if (this.checkSelectedRow()) {
          this.showMsgBox({
            title: 'Remove',
            message: 'Do you want to delete selected row?',
            type: 'warning',
            okFn: this.reqRemove,
          });
        }
      },
    },
  };
</script>
<style lang="scss" scoped>
  $title-button-height: 85px;

  .list-container {
    flex: 1;
    height: 100%;
    margin-right: 10px;
    .title {
      padding: 10px 15px 9px 0;
      font-size: 15px;
    }
    .button-container {
      display: flex;
      height: 40px;
      align-items: center;
      justify-content: flex-end;
    }
    .grid-container {
      height: calc(100% - #{$title-button-height});
    }
  }
  .btn {
    margin-right: 5px;
    .ei {
      margin-right: 3px;
    }
    &:last-child {
      margin-right: 0;
    }
  }
</style>
