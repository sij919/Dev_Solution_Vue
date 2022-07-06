<template>
  <div class="manage-container">
    <div class="row">
      <ev-label
        class="label"
        value="Name:"
        text-align="left"
      />
      <ev-text-field
        v-model.trim="serverGroupObj.group_name"
        type="text"
      />
    </div>
    <div class="row">
      <ev-label
        class="label"
        value="Description:"
        text-align="left"
      />
      <ev-text-field
        v-model.trim="serverGroupObj.group_descript"
        type="text"
      />
    </div>
    <div class="btn-container">
      <ev-button
        class="btn"
        size="small"
        type="primary"
        @click="onSave"
      >
        Save
      </ev-button>
      <ev-button
        class="btn"
        size="small"
        type="primary"
        @click="onCancel"
      >
        Cancel
      </ev-button>
    </div>
  </div>
</template>
<script>
  /**
   * @file : servergroup.list.manage.vue(서버그룹 리스트 crud 처리- 팝업)
   *
   * @logic :
   * 1. 서버 그룹의 등록, 수정 삭제 처리
   *
   * @method
   * 1. reqSave : 저장 api 호출
   * 2. checkValidate : 유효성 검증
   * 3. onSave : 저장 확인
   * 4. onCancel : 삭제 이벤트 상위 컴퍼넌트로 전달
   */
  import { request } from 'main/utils/api';
  import { API } from 'main/utils/define';

  export default {
    name: 'ServerGroupListManage',
    props: {
      type: {
        type: String,
        default: '',
      },
      selected: {
        type: Object,
        default() {
          return {};
        },
      },
    },
    data() {
      return {
        serverGroupObj: {
          group_name: '',
          group_descript: '',
        },
      };
    },
    mounted() {
      if (this.type === 'Modify') {
        this.serverGroupObj = Object.assign({
          group_name: '',
          group_id: '',
          group_descript: '',
        }, this.selected);
      }
    },
    methods: {
      reqSave(data) {
        let apiUrl;

        if (this.type === 'Add') {
          apiUrl = API.SETTINGS.SERVERGROUP.INSERT_SERVER_GROUP;
        } else if (this.type === 'Modify') {
          apiUrl = API.SETTINGS.SERVERGROUP.UPDATE_SERVER_GROUP;
        }

        request({
          method: 'post',
          url: apiUrl,
          headers: {
            'Content-Type': 'application/json',
          },
          params: {
            group_id: data.group_id,
            group_name: data.group_name,
            group_descript: data.group_descript,
          },
        }).then(() => {
          this.showMsgBox({
            title: 'Confirm',
            message: 'Saved',
            type: 'success',
            showCancelButton: false,
          });
          this.$emit('on-save');
        }).catch(({ message }) => {
          this.showMsgBox({
            title: 'Error',
            message,
            type: 'error',
            shwCancelButton: false,
          });
        });
        this.serverGroupObj = [];
      },
      checkValidate() {
        let msg;
        if (!this.serverGroupObj.group_name || !this.serverGroupObj.group_descript) {
          msg = 'Please enter all the authority information.';
        }
        return msg;
      },
      onSave() {
        const data = this.serverGroupObj;
        const message = this.checkValidate();
        if (message) {
          this.showMsgBox({
            title: 'Confirm',
            message,
            type: 'info',
            showCancelButton: false,
          });
          return;
        }

        this.reqSave(data);
      },
      onCancel() {
        this.$emit('on-cancel');
      },
    },
  };
</script>
<style lang="scss" scoped>
  .manage-container {
    width: 100%;
    height: 100%;
    .row {
      display: flex;
      height: 30px;
      margin-bottom: 2px;
      align-items: center;

      .label {
        flex: none;
        width: 120px;
      }
    }
    .btn-container {
      display: flex;
      justify-content: flex-end;
      margin-top: 10px;
      .btn {
        margin-right: 5px;
        &:last-child {
          margin-right: 0;
        }
      }
    }
  }
</style>
