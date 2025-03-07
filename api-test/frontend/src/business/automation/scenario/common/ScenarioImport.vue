<template>
  <el-dialog
    :close-on-click-modal="false"
    :title="$t('api_test.automation.scenario_import')"
    width="30%"
    :visible.sync="visible"
    class="api-import"
    v-loading="result"
    @close="close">
    <div class="header-bar">
      <div>{{ $t('api_test.api_import.data_format') }}</div>
      <el-radio-group v-model="selectedPlatformValue">
        <el-radio v-for="(item, index) in platforms" :key="index" :label="item.value">{{ item.name }}</el-radio>
      </el-radio-group>

      <div class="operate-button">
        <el-button class="save-button" type="primary" plain @click="save">
          {{ $t('commons.save') }}
        </el-button>
        <el-button class="cancel-button" type="warning" plain @click="visible = false">
          {{ $t('commons.cancel') }}
        </el-button>
      </div>
    </div>

    <el-form :model="formData" :rules="rules" label-width="110px" v-loading="result" ref="form">
      <el-row>
        <el-col :span="11">
          <el-form-item :label="$t('commons.import_module')">
            <ms-select-tree
              size="small"
              :data="moduleOptions"
              :defaultKey="formData.moduleId"
              @getValue="setModule"
              :obj="moduleObj"
              clearable
              checkStrictly />
          </el-form-item>
          <el-form-item :label="$t('commons.import_mode')" prop="modeId">
            <el-select size="small" v-model="formData.modeId" class="project-select" clearable style="width: 100%">
              <el-option v-for="item in modeOptions" :key="item.id" :label="item.name" :value="item.id"/>
            </el-select>
            <el-checkbox size="mini" v-if="formData.modeId === 'fullCoverage'" v-model="formData.coverModule">
              {{ this.$t('commons.cover_scenario') }}
            </el-checkbox>
          </el-form-item>
          <el-form-item
            v-xpack
            v-if="projectVersionEnable && formData.modeId === 'incrementalMerge'"
            :label="$t('api_test.api_import.import_version')"
            prop="versionId">
            <el-select size="small" v-model="formData.versionId" clearable style="width: 100%">
              <el-option v-for="item in versionOptions" :key="item.id" :label="item.name" :value="item.id" />
            </el-select>
          </el-form-item>
          <el-form-item
            v-xpack
            v-if="projectVersionEnable && formData.modeId === 'fullCoverage'"
            :label="$t('api_test.scenario_import.data_update_version')"
            prop="versionId">
            <el-select size="small" v-model="formData.updateVersionId" clearable style="width: 100%">
              <el-option v-for="item in versionOptions" :key="item.id" :label="item.name" :value="item.id" />
            </el-select>
          </el-form-item>
          <el-form-item
            v-xpack
            v-if="projectVersionEnable && formData.modeId === 'fullCoverage'"
            :label="$t('api_test.scenario_import.data_new_version')"
            prop="versionId">
            <el-select size="small" v-model="formData.versionId" clearable style="width: 100%">
              <el-option v-for="item in versionOptions" :key="item.id" :label="item.name" :value="item.id" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="1">
          <el-divider direction="vertical" />
        </el-col>
        <el-col :span="12">
          <el-upload
            class="api-upload"
            drag
            action=""
            :http-request="upload"
            :limit="1"
            :beforeUpload="uploadValidate"
            :on-remove="handleRemove"
            :file-list="fileList"
            :on-exceed="handleExceed"
            multiple>
            <i class="el-icon-upload"></i>
            <div class="el-upload__text" v-html="$t('load_test.upload_tips')"></div>
            <div class="el-upload__tip" slot="tip">
              {{ $t('api_test.api_import.file_size_limit') }}
            </div>
          </el-upload>
        </el-col>
      </el-row>
    </el-form>

    <div class="format-tip">
      <div>
        <span>{{ $t('api_test.api_import.tip') }}：{{ selectedPlatform.tip }}</span>
      </div>
      <div>
        <span>{{ $t('api_test.api_import.export_tip') }}：{{ selectedPlatform.exportTip }}</span>
      </div>
      <div>
        <span>
          {{ $t('api_test.api_import.cover_tip') }} :<br />
          {{ $t('api_test.api_import.cover_tip_scenario_1') }}<br />
          {{ $t('api_test.api_import.cover_tip_scenario_2') }}<br />
          {{ $t('api_test.api_import.cover_tip_scenario_3') }}<br />
          {{ $t('api_test.api_import.cover_tip_scenario_4') }}<br />
          {{ $t('api_test.api_import.no_cover_tip') }} :<br />
          {{ $t('api_test.api_import.no_cover_tip_scenario_1') }}<br />
          {{ $t('api_test.api_import.no_cover_tip_scenario_2') }}
        </span>
      </div>
    </div>
  </el-dialog>
</template>

<script>
import { getProjectVersions, versionEnableByProjectId } from '@/api/xpack';
import { importScenario } from '@/api/scenario';
import MsDialogFooter from 'metersphere-frontend/src/components/MsDialogFooter';
import { getCurrentProjectID } from 'metersphere-frontend/src/utils/token';
import { hasLicense } from 'metersphere-frontend/src/utils/permission';
import { listenGoBack, removeGoBackListener } from 'metersphere-frontend/src/utils';
import MsSelectTree from 'metersphere-frontend/src/components/select-tree/SelectTree';

export default {
  name: 'ScenarioImport',
  components: { MsDialogFooter, MsSelectTree },
  props: {
    saved: {
      type: Boolean,
      default: true,
    },
    moduleOptions: Array,
  },
  data() {
    return {
      visible: false,
      swaggerUrlEable: false,
      swaggerSynchronization: false,
      showEnvironmentSelect: true,
      modeOptions: [
        {
          id: 'fullCoverage',
          name: this.$t('commons.cover'),
        },
        {
          id: 'incrementalMerge',
          name: this.$t('commons.not_cover'),
        },
      ],
      protocol: '',
      platforms: [
        {
          name: 'MeterSphere',
          value: 'Metersphere',
          tip: this.$t('api_test.api_import.ms_tip'),
          exportTip: this.$t('api_test.api_import.ms_export_tip'),
          suffixes: new Set(['json']),
        },
        {
          name: 'Postman',
          value: 'Postman',
          tip: this.$t('api_test.api_import.postman_tip'),
          exportTip: this.$t('api_test.api_import.post_export_tip'),
          suffixes: new Set(['json']),
        },
        {
          name: 'JMeter',
          value: 'Jmeter',
          tip: this.$t('api_test.api_import.jmeter_tip'),
          exportTip: this.$t('api_test.api_import.jmeter_export_tip'),
          suffixes: new Set(['jmx']),
        },
        {
          name: 'HAR',
          value: 'Har',
          tip: this.$t('api_test.api_import.har_tip'),
          exportTip: this.$t('api_test.api_import.har_export_tip'),
          suffixes: new Set(['har']),
        },
      ],
      selectedPlatform: {},
      selectedPlatformValue: 'Metersphere',
      result: false,
      projects: [],
      environments: [],
      useEnvironment: false,
      formData: {
        file: undefined,
        swaggerUrl: '',
        modeId: 'incrementalMerge',
        moduleId: '',
        coverModule: false,
      },
      rules: {
        modeId: [
          {
            required: true,
            message: this.$t('commons.please_select_import_mode'),
            trigger: 'change',
          },
        ],
      },
      currentModule: {},
      fileList: [],
      moduleObj: {
        id: 'id',
        label: 'name',
      },
      versionOptions: [],
      projectVersionEnable: false,
    };
  },
  created() {
    this.getVersionOptions();
    this.checkVersionEnable();
    this.selectedPlatform = this.platforms[0];
  },
  watch: {
    selectedPlatformValue() {
      for (let i in this.platforms) {
        if (this.platforms[i].value === this.selectedPlatformValue) {
          this.selectedPlatform = this.platforms[i];
          break;
        }
      }
    },
  },
  computed: {
    projectId() {
      return getCurrentProjectID();
    },
  },
  methods: {
    scheduleEdit() {
      if (!this.formData.swaggerUrl) {
        this.$warning(this.$t('commons.please_fill_path'));
        this.swaggerSynchronization = !this.swaggerSynchronization;
      } else {
        if (this.swaggerSynchronization) {
          this.$refs.scheduleEdit.open(this.buildParam());
        }
      }
    },
    scheduleEditByText() {
      this.$refs.scheduleEdit.open(this.buildParam());
    },
    open(module) {
      this.currentModule = module;
      if (localStorage.getItem('scenarioModule')) {
        this.formData.moduleId = localStorage.getItem('scenarioModule');
      }
      if (module) {
        this.formData.moduleId = module.id;
      }
      if (!module && !localStorage.getItem('scenarioModule')) {
        this.formData.moduleId = this.moduleOptions[0].id;
      }
      this.visible = true;
      listenGoBack(this.close);
    },
    upload(file) {
      this.formData.file = file.file;
    },
    handleExceed(files, fileList) {
      this.$warning(this.$t('test_track.case.import.upload_limit_count'));
    },
    handleRemove(file, fileList) {
      this.formData.file = undefined;
    },
    uploadValidate(file, fileList) {
      let suffix = file.name.substring(file.name.lastIndexOf('.') + 1);
      if (this.selectedPlatform.suffixes && !this.selectedPlatform.suffixes.has(suffix)) {
        this.$warning(this.$t('api_test.api_import.suffixFormatErr'));
        return false;
      }
      if (file.size / 1024 / 1024 > 100) {
        this.$warning(this.$t('test_track.case.import.upload_limit_size'));
        return false;
      }
      return true;
    },
    save() {
      localStorage.setItem('scenarioModule', this.formData.moduleId);
      if (!this.formData.file) {
        this.$warning('请添加一个文件');
        return;
      }
      let suffix = this.formData.file.name.substring(this.formData.file.name.lastIndexOf('.') + 1);
      if (this.selectedPlatform.suffixes && !this.selectedPlatform.suffixes.has(suffix)) {
        this.$warning(this.$t('api_test.api_import.suffixFormatErr'));
        return false;
      }
      this.$refs.form.validate((valid) => {
        if (valid) {
          let param = this.buildParam();
          this.result = importScenario('/api/automation/import', param.file, null, this.buildParam()).then(
            (response) => {
              let res = response.data;
              this.$success(this.$t('test_track.case.import.success'));
              this.visible = false;
              this.$emit('refreshAll', res);
            }
          );
        } else {
          return false;
        }
      });
    },
    buildParam() {
      let param = {};
      Object.assign(param, this.formData);
      param.platform = this.selectedPlatformValue;
      param.saved = this.saved;
      if (this.currentModule) {
        param.moduleId = this.formData.moduleId;
        this.moduleOptions.filter((item) => {
          if (item.id === this.formData.moduleId) {
            param.modulePath = item.path;
          }
        });
        param.modeId = this.formData.modeId;
      }
      if (this.formData.moduleId.length === 0) {
        param.moduleId = '';
      }
      param.projectId = this.projectId;
      if (!this.swaggerUrlEable) {
        param.swaggerUrl = undefined;
      }
      return param;
    },
    close() {
      localStorage.setItem('scenarioModule', this.formData.moduleId);
      this.formData = {
        file: undefined,
        swaggerUrl: '',
        modeId: this.formData.modeId,
        moduleId: '',
        coverModule: false,
      };
      this.fileList = [];
      removeGoBackListener(this.close);
      this.visible = false;
    },
    setModule(id, data) {
      this.formData.moduleId = id;
      this.formData.modulePath = data.path;
    },
    getVersionOptions() {
      if (hasLicense()) {
        getProjectVersions(getCurrentProjectID()).then((response) => {
          this.versionOptions = response.data.filter((v) => v.status === 'open');
          this.versionOptions.forEach((v) => {
            if (v.latest) {
              v.name = v.name + ' ' + this.$t('api_test.api_import.latest_version');
            }
          });
        });
      }
    },
    checkVersionEnable() {
      if (!this.projectId) {
        return;
      }
      if (hasLicense()) {
        versionEnableByProjectId(this.projectId).then((response) => {
          this.projectVersionEnable = response.data;
        });
      }
    },
  },
};
</script>

<style scoped>
.api-import :deep(.el-dialog) {
  min-width: 700px;
}

.format-tip {
  background: #ededed;
}

.api-upload {
  text-align: center;
  margin: auto 0;
}

.api-upload :deep(.el-upload) {
  width: 100%;
  max-width: 350px;
}

.api-upload :deep(.el-upload-dragger) {
  width: 100%;
}

.el-radio-group {
  margin: 10px 0;
}

.header-bar,
.format-tip,
.el-form {
  border: solid #e1e1e1 1px;
  margin: 10px 0;
  padding: 10px;
  border-radius: 3px;
}

.header-bar {
  padding: 10px 30px;
}

.api-import :deep(.el-dialog__body) {
  padding: 15px 25px;
}

.operate-button {
  float: right;
}

.save-button {
  margin-left: 10px;
}


.dialog-footer {
  float: right;
}

.swagger-url-disable {
  margin-top: 10px;

  margin-left: 80px;
}

.el-divider {
  height: 200px;
}
</style>
