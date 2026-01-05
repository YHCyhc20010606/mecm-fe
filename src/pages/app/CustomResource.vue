<template>
  <div id="customResourceDialog">
    <div class="secondLabel">
      {{ $t('resource.custRes') }}
    </div>
    <el-form
      label-width="auto"
      class="configForm"
    >
      <el-row>
        <el-col :span="12">
          <div
            v-for="(item, index) in customResourceData.inputs"
            :key="index"
          >
            <el-form-item
              v-if="index%2==0"
              :label="item.name"
              prop="name"
            >
              <el-input
                id="appname"
                maxlength="20"
                width="100"
                :placeholder="item.type"
                v-model="item.defaultValue"
              />
            </el-form-item>
          </div>
        </el-col>
        <el-col :span="12">
          <div
            v-for="(item, index) in customResourceData.inputs"
            :key="index"
          >
            <el-form-item
              v-if="index%2!=0"
              :label="item.name"
              prop="name"
            >
              <el-input
                id="appname"
                maxlength="20"
                width="100"
                :placeholder="item.type"
                v-model="item.defaultValue"
              />
            </el-form-item>
          </div>
        </el-col>
      </el-row>
    </el-form>
    <div
      slot="footer"
      class="dialog-footer"
    >
      <el-button
        id="confirmBtn"
        type="primary"
        size="small"
        @click="confirm(customResourceData)"
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
    </div>
  </div>
</template>
<script>
export default {
  name: 'CustomResource',
  data () {
    return {
    }
  },
  props: {
    cancel: {
      type: Function,
      default: function () {
        return null
      }
    },
    confirm: {
      type: Function,
      default: function () {
        return null
      }
    },
    customResourceData: {
      type: Object,
      default: function () {
        return {}
      }
    }
  }
}
</script>
<style lang='less' scoped>
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
}
#customResourceDialog {
    padding: 0px 0px 20px 0px;
}
</style>
