<template>
  <div class="dashboard-card">
    <el-card shadow="never" class="box-card" style="height: 100%">
      <div slot="header" class="clearfix">
        <span class="dashboard-title">{{ $t('home.dashboard.api.title') }}</span>
      </div>
      <div v-loading="loading" element-loading-background="#FFFFFF">
        <div
          v-show="loadError"
          style="
            width: 100%;
            height: 300px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
          ">
          <img style="height: 100px; width: 100px" src="/assets/module/figma/icon_load_error.svg" />
          <span class="addition-info-title" style="color: #646a73">{{ $t('home.dashboard.public.load_error') }}</span>
        </div>
        <div v-show="!loadError">
          <div class="main-info">
            <count-chart :version-id="versionId" :api-data="apiData" ref="countChart" />
          </div>
          <div class="addition-info">
            <el-row :gutter="16" style="margin: 0">
              <el-col :span="12" style="padding-left: 0">
                <hover-card
                  :title="$t('home.dashboard.api.covered_rate')"
                  :main-info="apiData.apiCoveredRate"
                  :tool-tip="apiCoveredRageToolTip">
                  <!--未覆盖、已覆盖-->
                  <template v-slot:mouseOut>
                    <div style="margin: 16px 0px 0px 16px">
                      <el-row>
                        <el-col :span="12">
                          <span class="addition-info-title">
                            {{ $t('home.dashboard.public.covered') }}
                          </span>
                          <div class="common-amount">
                            <el-popover placement="top-start" width="200" trigger="hover">
                              <div>
                                <el-row class="addition-info-title">
                                  <el-col>
                                    <div
                                      style="
                                        width: 8px;
                                        height: 8px;
                                        background-color: #aa4fbf;
                                        margin: 7px;
                                        float: left;
                                      " />
                                    <span style="line-height: 22px">HTTP :</span>
                                    <div style="float: right">
                                      <el-link
                                        class="coverage-num-link"
                                        @click="redirectPageWithDataType('api', 'api', 'covered', null, 'HTTP')"
                                        >{{ apiData.httpCovered }}</el-link
                                      >
                                    </div>
                                  </el-col>
                                </el-row>
                                <el-row class="addition-info-title">
                                  <div
                                    style="
                                      width: 8px;
                                      height: 8px;
                                      background-color: #fad355;
                                      margin: 7px;
                                      float: left;
                                    " />
                                  <span style="line-height: 22px">DUBBO :</span>
                                  <div style="float: right">
                                    <el-link
                                      class="coverage-num-link"
                                      @click="redirectPageWithDataType('api', 'api', 'covered', null, 'DUBBO')"
                                      >{{ apiData.rpcCovered }}</el-link
                                    >
                                  </div>
                                </el-row>
                                <el-row class="addition-info-title">
                                  <div
                                    style="
                                      width: 8px;
                                      height: 8px;
                                      background-color: #14e1c6;
                                      margin: 7px;
                                      float: left;
                                    " />
                                  <span style="line-height: 22px">TCP :</span>
                                  <div style="float: right">
                                    <el-link
                                      class="coverage-num-link"
                                      @click="redirectPageWithDataType('api', 'api', 'covered', null, 'TCP')"
                                      >{{ apiData.tcpCovered }}
                                    </el-link>
                                  </div>
                                </el-row>
                                <el-row class="addition-info-title">
                                  <div
                                    style="
                                      width: 8px;
                                      height: 8px;
                                      background-color: #4e83fd;
                                      margin: 7px;
                                      float: left;
                                    " />
                                  <span style="line-height: 22px">SQL :</span>
                                  <div style="float: right">
                                    <el-link
                                      class="coverage-num-link"
                                      @click="redirectPageWithDataType('api', 'api', 'covered', null, 'SQL')"
                                      >{{ apiData.sqlCovered }}</el-link
                                    >
                                  </div>
                                </el-row>
                              </div>
                              <el-link slot="reference" class="addition-info-num">
                                {{ formatAmount(apiData.coveredCount) }}
                              </el-link>
                            </el-popover>
                          </div>
                        </el-col>
                        <el-col :span="12">
                          <span class="addition-info-title">
                            {{ $t('home.dashboard.public.not_covered') }}
                          </span>
                          <div class="common-amount">
                            <el-popover placement="top-start" width="200" trigger="hover">
                              <div>
                                <el-row class="addition-info-title">
                                  <div
                                    style="
                                      width: 8px;
                                      height: 8px;
                                      background-color: #aa4fbf;
                                      margin: 7px;
                                      float: left;
                                    " />
                                  <span style="line-height: 22px">HTTP:</span>
                                  <div style="float: right">
                                    <el-link
                                      class="coverage-num-link"
                                      @click="redirectPageWithDataType('api', 'api', 'notCovered', null, 'HTTP')"
                                      >{{ apiData.httpNotCovered }}</el-link
                                    >
                                  </div>
                                </el-row>
                                <el-row class="addition-info-title">
                                  <div
                                    style="
                                      width: 8px;
                                      height: 8px;
                                      background-color: #fad355;
                                      margin: 7px;
                                      float: left;
                                    " />
                                  <span style="line-height: 22px">DUBBO :</span>
                                  <div style="float: right">
                                    <el-link
                                      class="coverage-num-link"
                                      @click="redirectPageWithDataType('api', 'api', 'notCovered', null, 'DUBBO')"
                                      >{{ apiData.rpcNotCovered }}</el-link
                                    >
                                  </div>
                                </el-row>
                                <el-row class="addition-info-title">
                                  <div
                                    style="
                                      width: 8px;
                                      height: 8px;
                                      background-color: #14e1c6;
                                      margin: 7px;
                                      float: left;
                                    " />
                                  <span style="line-height: 22px">TCP :</span>
                                  <div style="float: right">
                                    <el-link
                                      class="coverage-num-link"
                                      @click="redirectPageWithDataType('api', 'api', 'notCovered', null, 'TCP')"
                                      >{{ apiData.tcpNotCovered }}</el-link
                                    >
                                  </div>
                                </el-row>
                                <el-row class="addition-info-title">
                                  <div
                                    style="
                                      width: 8px;
                                      height: 8px;
                                      background-color: #4e83fd;
                                      margin: 7px;
                                      float: left;
                                    " />
                                  <span style="line-height: 22px">SQL :</span>
                                  <div style="float: right">
                                    <el-link
                                      class="coverage-num-link"
                                      @click="redirectPageWithDataType('api', 'api', 'notCovered', null, 'SQL')"
                                      >{{ apiData.sqlNotCovered }}</el-link
                                    >
                                  </div>
                                </el-row>
                              </div>
                              <el-link slot="reference" class="addition-info-num">
                                {{ formatAmount(apiData.notCoveredCount) }}
                              </el-link>
                            </el-popover>
                          </div>
                        </el-col>
                      </el-row>
                    </div>
                  </template>
                </hover-card>
              </el-col>

              <el-col :span="12" style="padding-right: 0">
                <hover-card
                  :title="$t('home.dashboard.api.completed_rate')"
                  :main-info="apiData.completedRate"
                  :tool-tip="completedRageToolTip">
                  <!--已完成、进行中、未开始-->
                  <template v-slot:mouseOut>
                    <div style="margin: 16px 0px 0px 16px">
                      <el-row>
                        <el-col :span="8">
                          <span class="addition-info-title">
                            {{ $t('home.dashboard.public.completed') }}
                          </span>
                          <div class="common-amount">
                            <el-link
                              class="addition-info-num"
                              v-permission-disable="['PROJECT_API_DEFINITION:READ']"
                              @click="redirectPage('api', 'api', 'Completed', null)">
                              {{ formatAmount(apiData.finishedCount) }}
                            </el-link>
                          </div>
                        </el-col>
                        <el-col :span="8">
                          <span class="addition-info-title">
                            {{ $t('home.dashboard.public.underway') }}
                          </span>
                          <div class="common-amount">
                            <el-link
                              class="addition-info-num"
                              v-permission-disable="['PROJECT_API_DEFINITION:READ']"
                              @click="redirectPage('api', 'api', 'Underway', null)">
                              {{ formatAmount(apiData.runningCount) }}
                            </el-link>
                          </div>
                        </el-col>
                        <el-col :span="8">
                          <span class="addition-info-title">
                            {{ $t('home.dashboard.public.prepared') }}
                          </span>
                          <div class="common-amount">
                            <el-link
                              class="addition-info-num"
                              v-permission-disable="['PROJECT_API_DEFINITION:READ']"
                              @click="redirectPage('api', 'api', 'Prepare', null)">
                              {{ formatAmount(apiData.notStartedCount) }}
                            </el-link>
                          </div>
                        </el-col>
                      </el-row>
                    </div>
                  </template>
                </hover-card>
              </el-col>
            </el-row>
          </div>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
import countChart from '@/business/home/components/chart/CountChart';
import hoverCard from '@/business/home/components/card/HoverCard';
import { getCurrentProjectID } from 'metersphere-frontend/src/utils/token';
import { apiCountByProjectId, formatNumber } from '@/api/home';

export default {
  name: 'ApiDashboard',
  components: { countChart, hoverCard },
  data() {
    return {
      loading: false,
      loadError: false,
      apiCoveredRageToolTip: this.$t('api_test.home_page.formula.api_coverage'),
      completedRageToolTip: this.$t('api_test.home_page.formula.completion'),
      versionId: 'default',
      apiData: {
        httpCount: 0,
        tcpCount: 0,
        rpcCount: 0,
        sqlCount: 0,
        createdInWeek: 0,
        apiCoveredRate: '0%',
        completedRate: '0%',
        coveredCount: 0,
        notCoveredCount: 0,
        runningCount: 0,
        finishedCount: 0,
        notRunCount: 0,
        httpCovered: 0,
        rpcCovered: 0,
        tcpCovered: 0,
        sqlCovered: 0,
        httpNotCovered: 0,
        rpcNotCovered: 0,
        tcpNotCovered: 0,
        sqlNotCovered: 0,
      },
    };
  },
  methods: {
    search(versionId) {
      this.loading = true;
      this.loadError = false;
      this.versionId = versionId;
      let selectProjectId = getCurrentProjectID();
      apiCountByProjectId(selectProjectId, versionId)
        .then((response) => {
          this.loading = false;
          this.loadError = false;
          this.apiData = response.data;
          this.$refs.countChart.reload();
        })
        .catch(() => {
          this.loading = false;
          this.loadError = true;
          this.$refs.countChart.reload();
        });
    },
    formatAmount(number) {
      return formatNumber(number);
    },
    redirectPage(redirectPage, dataType, selectRange, selectParam) {
      this.$emit('redirectPage', redirectPage, dataType, selectRange, selectParam);
    },
    redirectPageWithDataType(redirectPage, dataType, selectRange, selectParam, type) {
      this.$emit('redirectPageWithDataType', redirectPage, dataType, selectRange, selectParam, type);
    },
  },
};
</script>

<style scoped>
.coverage-num-link {
  line-height: 22px;
  color: #783887 !important;
  font-size: 14px;
  font-weight: 500;
}
</style>
