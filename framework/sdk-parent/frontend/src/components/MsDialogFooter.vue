<template>

  <div class="dialog-footer">
    <el-button @click="cancel" :size="btnSize">{{ $t('commons.cancel') }}</el-button>
    <el-button v-prevent-re-click type="primary" @click="confirm" @keydown.enter.native.prevent :size="btnSize" :disabled="disableSaveBtn">{{ $t('commons.confirm') }}
    </el-button>
    <el-button v-prevent-re-click type="primary" v-if="isShow" @click="saveAsEdit" @keydown.enter.native.prevent>{{ title }}</el-button>
  </div>

</template>

<script>
export default {
  name: "MsDialogFooter",
  props: {
    isShow: {
      type: Boolean,
      default: false,
    },
    title: String,
    btnSize: {
      type: String,
      default() {
        return '';
      }
    }
  },
  data() {
    return {
      disableSaveBtn: false,
    };
  },
  methods: {
    cancel() {
      this.$emit("cancel");
    },
    confirm() {
      this.$emit("confirm");
    },
    saveAsEdit() {
      this.$emit("saveAsEdit");
    }
  }
}
</script>

<style scoped>

</style>
