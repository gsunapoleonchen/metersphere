<template>
  <header class="report-header">
    <div class="action-row">
      <div>
        <span v-if="!debug">
          <el-input
            v-if="nameIsEdit"
            size="mini"
            @blur="handleSave(report.name)"
            @keyup.enter.native="handleSaveKeyUp"
            style="width: 200px"
            v-model="report.name"
            maxlength="60"
            show-word-limit />
          <span v-else>
            <el-link v-if="!isShare && isSingleScenario" type="primary" class="report-name" @click="redirect">
              {{ report.name }}
            </el-link>
            <span v-else>
              {{ report.name }}
            </span>
            <i
              v-if="showCancelButton"
              class="el-icon-edit"
              style="cursor: pointer"
              @click="nameIsEdit = true"
              @click.stop />
          </span>
        </span>
        <span v-if="report.endTime || report.createTime">
          <span style="margin-left: 10px">{{ $t('report.test_start_time') }}：</span>
          <span class="time"> {{ report.createTime | datetimeFormat }}</span>
          <span style="margin-left: 10px">{{ $t('report.test_end_time') }}：</span>
          <span class="time"> {{ report.endTime | datetimeFormat }}</span>
        </span>
      </div>
      <div>
        <el-popover
          v-if="!isPlan && (!debug || exportFlag) && !isTemplate"
          v-permission="['PROJECT_PERFORMANCE_REPORT:READ+EXPORT']"
          placement="bottom"
          trigger="click"
          popperClass="ms-custom-message-class"
          width="300"
          >
          <p>{{ shareUrl }}</p>
          <span style="color: red" v-if="application.typeValue">{{
            $t('commons.validity_period') + application.typeValue
          }}</span>
          <div>
            <el-button type="primary" size="mini" :disabled="!shareUrl" v-clipboard:copy="shareUrl"
              >{{ $t('commons.copy') }}
            </el-button>
          </div>
          <template v-slot:reference>
            <el-button :disabled="isReadOnly" type="danger" plain size="mini" @click="handleShare(report)">
              {{ $t('test_track.plan_view.share_report') }}
            </el-button>
          </template>
        </el-popover>
        <el-button class="btn-second" v-if="showCancelButton" size="mini" @click="returnView">
          {{ $t('commons.cancel') }}
        </el-button>
        <el-button
          v-if="!isPlan && (!debug || exportFlag) && !isTemplate"
          v-permission="['PROJECT_API_REPORT:READ+EXPORT']"
          :disabled="isReadOnly"
          plain
          type="primary"
          size="mini"
          class="btn-second"
          @click="handleExport(report.name)">
          {{ $t('test_track.plan_view.export_report') }}
        </el-button>
        <el-button class="btn-second" v-if="showRerunButton" plain size="mini" @click="rerun">
          {{ $t('api_test.automation.rerun') }}
        </el-button>
      </div>
    </div>
    <div class="content-row">
      <div v-if="this.mode">
        <span> {{ $t('report.run_model') + ':' }} </span>
          <span style="color: #61c550">
          {{ getModeName(this.mode) }}
        </span>
      </div>
      <div v-if="this.poolName">
        <span> {{ $t('load_test.select_resource_pool') + ':' }} </span>
        <span style="color: #61c550;">
          {{ this.poolName }}
        </span>
      </div>
    </div>
    <div v-if="showProjectEnv" class="content-row">
      <span> {{ $t('commons.environment') + ':' }} </span>
      <span v-for="(values, key) in projectEnvMap" :key="key">
        {{ key + ':' }}
        <ms-tag v-for="(item, index) in values" :key="index" type="success" :content="item" style="margin-left: 2px" />
      </span>
    </div>
  </header>
</template>

<script>
import { generateShareInfoWithExpired, getShareRedirectUrl } from '../../../api/share';
import { getCurrentProjectID } from 'metersphere-frontend/src/utils/token';
import MsTag from 'metersphere-frontend/src/components/MsTag';
import { apiProjectByScenarioId, getProjectApplicationConfig } from '../../../api/project';
import { apiTestReRun } from '../../../api/xpack';
import { getUUID } from 'metersphere-frontend/src/utils';
import { getApiScenarioIdByPlanScenarioId } from '@/api/test-plan';

export default {
  name: 'MsApiReportViewHeader',
  components: { MsTag },
  props: {
    report: {},
    projectEnvMap: {},
    debug: Boolean,
    showCancelButton: {
      type: Boolean,
      default: true,
    },
    showRerunButton: {
      type: Boolean,
      default: false,
    },
    isTemplate: Boolean,
    exportFlag: {
      type: Boolean,
      default: false,
    },
    isPlan: Boolean,
    poolName: {
      type: String,
    },
    mode: {
      type: String,
    },
    isShare: Boolean,
  },
  computed: {
    showProjectEnv() {
      return this.projectEnvMap && JSON.stringify(this.projectEnvMap) !== '{}';
    },
    path() {
      return '/api/test/edit?id=' + this.report.testId;
    },
    scenarioId() {
      if (typeof this.report.scenarioId === 'string') {
        return this.report.scenarioId;
      } else {
        return '';
      }
    },
    isSingleScenario() {
      try {
        JSON.parse(this.report.scenarioId);
        return false;
      } catch (e) {
        return true;
      }
    },
  },
  data() {
    return {
      isReadOnly: false,
      nameIsEdit: false,
      shareUrl: '',
      application: {},
    };
  },
  methods: {
    handleExport(name) {
      this.$emit('reportExport', name);
    },
    handleSave(name) {
      this.nameIsEdit = false;
      this.$emit('reportSave', name);
    },
    handleSaveKeyUp($event) {
      $event.target.blur();
    },
    redirect() {
      let resourceId = this.scenarioId;
      getApiScenarioIdByPlanScenarioId(this.scenarioId).then((res) => {
        resourceId = res.data;
        apiProjectByScenarioId(resourceId).then((response) => {
          if (response.data) {
            let projectId = response.data.id;
            let workspaceId = response.data.workspaceId;
            let projectName = response.data.name;
            let workspaceName = response.data.workspaceName;
            this.showDetails(resourceId, projectId, projectName, workspaceId, workspaceName);
          }
        });
      });
    },
    showDetails(resourceId, projectId, projectName, workspaceId, workspaceName) {
      let uuid = getUUID().substring(1, 5);
      let prefix = '/#';
      if (
        this.$route &&
        this.$route.path.startsWith('/api/automation/report') &&
        this.$route.query &&
        this.$route.query.list
      ) {
        prefix = '';
      }
      let path = `${prefix}/api/automation/?redirectID=${uuid}&dataType=scenario&projectId=${projectId}&workspaceId=${workspaceId}&resourceId=${resourceId}&projectName=${projectName}&workspaceName=${workspaceName}`;
      let data = this.$router.resolve({
        path: path,
      });
      window.open(data.href, '_blank');
    },
    rerun() {
      let type = this.report.reportType;
      let rerunObj = { type: type, reportId: this.report.id };
      apiTestReRun(rerunObj).then((res) => {
        if (res.data !== 'SUCCESS') {
          this.$error(res.data);
        } else {
          this.$success(this.$t('api_test.automation.rerun_success'));
          if (this.$route.query && this.$route.query.list) {
            this.returnView();
          }
        }
      });
    },
    returnView() {
      this.$router.push('/api/automation/report');
    },
    handleShare(report) {
      this.getProjectApplication();
      let pram = {};
      pram.customData = report.id;
      pram.shareType = 'API_REPORT';
      generateShareInfoWithExpired(pram).then((res) => {
        let data = res.data;
        this.shareUrl = getShareRedirectUrl(data);
      });
    },
    getProjectApplication() {
      getProjectApplicationConfig(getCurrentProjectID(), '/API_SHARE_REPORT_TIME').then((res) => {
        if (res.data && res.data.typeValue) {
          let quantity = res.data.typeValue.substring(0, res.data.typeValue.length - 1);
          let unit = res.data.typeValue.substring(res.data.typeValue.length - 1);
          if (unit === 'H') {
            res.data.typeValue = quantity + this.$t('commons.date_unit.hour');
          } else if (unit === 'D') {
            res.data.typeValue = quantity + this.$t('commons.date_unit.day');
          } else if (unit === 'M') {
            res.data.typeValue = quantity + this.$t('commons.workspace_unit') + this.$t('commons.date_unit.month');
          } else if (unit === 'Y') {
            res.data.typeValue = quantity + this.$t('commons.date_unit.year');
          }
          this.application = res.data;
        }
      });
    },
    getModeName(mode) {
      switch (mode) {
        case 'serial':
          return this.$t('run_mode.serial');
        case 'parallel':
          return this.$t('run_mode.parallel');
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.report-header {
  min-width: 1200px;
  .btn-second {
    margin-left: 10px;
  }
  .action-row{
    display: flex;
    justify-content: space-between;
    align-items: center;
    .report-name {
      border-bottom: 1px solid var(--primary_color);
    }
  }
  .content-row{
    margin-top: 5px;
  }
}
</style>
