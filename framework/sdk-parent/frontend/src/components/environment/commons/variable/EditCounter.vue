<template>
  <el-form
    :model="editData"
    label-position="right"
    label-width="80px"
    size="small"
    ref="form2"
    :rules="rules"
  >
    <el-form-item :label="$t('api_test.variable_name')" prop="name">
      <el-input
        v-model="editData.name"
        :placeholder="$t('api_test.variable_name')"
      ></el-input>
    </el-form-item>

    <el-form-item :label="$t('commons.description')" prop="description">
      <el-input
        class="ms-http-textarea"
        v-model="editData.description"
        type="textarea"
        :autosize="{ minRows: 2, maxRows: 10 }"
        :rows="2"
        size="small"
        :disabled="disabled"
      />
    </el-form-item>

    <el-form-item :label="$t('variables.start')" prop="startNumber">
      <el-input
        :disabled="disabled"
        size="small"
        v-model="editData.startNumber"
        placeholder="0"
        style="width: 100%"
      />
    </el-form-item>
    <el-form-item :label="$t('variables.end')" prop="endNumber">
      <el-input
        :disabled="disabled"
        size="small"
        v-model="editData.endNumber"
        placeholder="10"
        style="width: 100%"
      />
    </el-form-item>
    <el-form-item :label="$t('variables.increment')" prop="increment">
      <el-input
        :disabled="disabled"
        size="small"
        v-model="editData.increment"
        placeholder="1"
        style="width: 100%"
      />
    </el-form-item>
    <el-form-item :label="$t('variables.format')" prop="value">
      <el-input
        :disabled="disabled"
        v-model="editData.value"
        :placeholder="$t('variables.counter_info')"
      ></el-input>
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  name: "MsEditCounter",
  components: {},
  props: {
    editData: {},
  },
  computed: {
    disabled() {
      return !(this.editData.name && this.editData.name !== "");
    },
  },
  data() {
    return {
      rules: {
        name: [
          {
            required: true,
            message: this.$t("test_track.variable_name"),
            trigger: "blur",
          },
        ],
      },
    };
  },
};
</script>

<style scoped></style>
