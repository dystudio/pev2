<template>
  <div>
    <div class="form-group text-center my-1">
      <div class="btn-group btn-group-xs">
        <button class="btn btn-outline-secondary" :class="{'active': metric === metrics.time}" v-on:click="metric = metrics.time">time</button>
        <button class="btn btn-outline-secondary" :class="{'active': metric === metrics.rows}" v-on:click="metric = metrics.rows">rows</button>
        <button class="btn btn-outline-secondary" :class="{'active': metric === metrics.cost}" v-on:click="metric = metrics.cost">cost</button>
        <button class="btn btn-outline-secondary" :class="{'active': metric === metrics.buffers}" v-on:click="metric = metrics.buffers">buffers</button>
      </div>
    </div>
    <div class="form-group text-center my-1" v-if="metric == metrics.buffers">
      <div class="btn-group btn-group-xs">
        <button class="btn btn-outline-secondary" :class="{'active': buffersMetric === buffersMetrics.shared}" v-on:click="buffersMetric = buffersMetrics.shared">shared</button>
        <button class="btn btn-outline-secondary" :class="{'active': buffersMetric === buffersMetrics.temp}" v-on:click="buffersMetric = buffersMetrics.temp">temp</button>
        <button class="btn btn-outline-secondary" :class="{'active': buffersMetric === buffersMetrics.local}" v-on:click="buffersMetric = buffersMetrics.local">local</button>
      </div>
    </div>
    <div class="legend text-center">
      <ul class="list-unstyled list-inline mb-0" v-if="metric == metrics.buffers">
        <li class="list-inline-item">
          <span class="bg-hit rounded"></span>
          Hit
        </li>
        <li class="list-inline-item">
          <span class="bg-read"></span>
          Read
        </li>
        <li class="list-inline-item">
          <span class="bg-written"></span>
          Written
        </li>
        <li class="list-inline-item">
          <span class="bg-dirtied"></span>
          Dirtied
        </li>
      </ul>
    </div>
    <table class="my-1 table-hover">
      <tbody>
        <tr v-for="row, index in data" :content="tooltip(row[1])" v-tippy="{arrow: true, animation: 'fade', delay: [200, 0]}" @click.prevent="showNode(row[1], false, true)">
          <th class="node-type pr-2">
            <template v-for="i in lodash.range(row[0])">
              <template v-if="lodash.indexOf(row[3], i) != -1">│</template><template v-else-if="i !== 0">⠀</template>
            </template>{{ index !== 0 ? (row[2] ? '└' : '├') : '' }}
            {{ nodeType(row) }}
          </th>
          <td>
            <div class="progress rounded-0 align-items-center bg-transparent" style="height: 5px;" v-if="metric == metrics.time">
              <div class="bg-secondary" role="progressbar" :style="'opacity: 0.2;width: ' + (row[1].node[nodeProps.INCLUSIVE_DURATION] - row[1].node[nodeProps.ACTUAL_DURATION]) / (plan.planStats.executionTime || plan.content.Plan[nodeProps.ACTUAL_TOTAL_TIME]) * 100 + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 1px;"></div>
              <div class="progress-bar border-left bg-secondary" role="progressbar" :style="'width: ' + row[1].node[nodeProps.ACTUAL_DURATION] / (plan.planStats.executionTime || plan.content.Plan[nodeProps.ACTUAL_TOTAL_TIME]) * 100 + '%; height:5px;'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100"></div>
            </div>
            <div class="progress rounded-0 align-items-center bg-transparent" style="height: 5px;" v-else-if="metric == metrics.rows">
              <div class="bg-secondary" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.ACTUAL_ROWS] / plan.planStats.maxRows * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
            </div>
            <div class="progress rounded-0 align-items-center bg-transparent" style="height: 5px;" v-else-if="metric == metrics.cost">
              <div class="bg-secondary" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.ACTUAL_COST] / plan.planStats.maxCost * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
            </div>
            <div class="progress rounded-0 align-items-center bg-transparent" style="height: 5px;" v-else-if="metric == metrics.buffers && buffersMetric == buffersMetrics.shared && plan.planStats.maxSharedBlocks">
              <div class="bg-hit" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.SHARED_HIT_BLOCKS] / plan.planStats.maxSharedBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
              <div class="bg-read" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.SHARED_READ_BLOCKS] / plan.planStats.maxSharedBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
              <div class="bg-written" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.SHARED_WRITTEN_BLOCKS] / plan.planStats.maxSharedBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
              <div class="bg-dirtied" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.SHARED_DIRTIED_BLOCKS] / plan.planStats.maxSharedBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
            </div>
            <div class="progress rounded-0 align-items-center bg-transparent" style="height: 5px;" v-else-if="metric == metrics.buffers && buffersMetric == buffersMetrics.temp && plan.planStats.maxTempBlocks">
              <div class="bg-hit" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.TEMP_HIT_BLOCKS] / plan.planStats.maxTempBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
              <div class="bg-read" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.TEMP_READ_BLOCKS] / plan.planStats.maxTempBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
              <div class="bg-written" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.TEMP_WRITTEN_BLOCKS] / plan.planStats.maxTempBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
              <div class="bg-dirtied" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.TEMP_DIRTIED_BLOCKS] / plan.planStats.maxTempBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
            </div>
            <div class="progress rounded-0 align-items-center bg-transparent" style="height: 5px;" v-else-if="metric == metrics.buffers && buffersMetric == buffersMetrics.local && plan.planStats.maxLocalBlocks">
              <div class="bg-hit" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.LOCAL_HIT_BLOCKS] / plan.planStats.maxLocalBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
              <div class="bg-read" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.LOCAL_READ_BLOCKS] / plan.planStats.maxLocalBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
              <div class="bg-written" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.LOCAL_WRITTEN_BLOCKS] / plan.planStats.maxLocalBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
              <div class="bg-dirtied" role="progressbar" :style="'width: ' + Math.round(row[1].node[nodeProps.LOCAL_DIRTIED_BLOCKS] / plan.planStats.maxLocalBlocks * 100) + '%'" aria-valuenow="15" aria-valuemin="0" aria-valuemax="100" style="height: 5px;"></div>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script lang="ts">
import * as _ from 'lodash';
import { Component, Prop, Vue } from 'vue-property-decorator';
import { duration, durationClass, rows } from '@/filters';
import { BuffersMetric, NodeProp, Metric } from '../enums';
import PlanNode from '@/components/PlanNode.vue';
import { IPlan } from '../iplan';

@Component({
  filters: {
    durationClass,
  },
})
export default class Diagram extends Vue {
  @Prop() private data!: any[];
  @Prop() private plan!: IPlan;
  @Prop() private showNode!: () => void;

  private lodash = _;
  private nodeProps = NodeProp;
  private metrics = Metric;
  private buffersMetrics = BuffersMetric;

  private metric: Metric = Metric.buffers;
  private buffersMetric: BuffersMetric = BuffersMetric.shared;

  private tooltip(cmp: PlanNode): string {
    switch (this.metric) {
      case Metric.time:
        return this.timeTooltip(cmp);
      case Metric.rows:
        return this.rowsTooltip(cmp);
      case Metric.cost:
        return this.costTooltip(cmp);
      case Metric.buffers:
        return this.buffersTooltip(cmp);
    }
    return '';
  }

  private timeTooltip(cmp: PlanNode): string {
    return [
      'Duration: ',
      duration(cmp.node[NodeProp.ACTUAL_DURATION], true),
      ' | ',
      cmp.executionTimePercent,
      '%',
    ].join('');
  }

  private rowsTooltip(cmp: PlanNode): string {
    return [
      'Rows: ',
      rows(cmp.node[NodeProp.ACTUAL_ROWS]),
    ].join('');
  }

  private costTooltip(cmp: PlanNode): string {
    return [
      'Cost: ',
      rows(cmp.node[NodeProp.ACTUAL_COST]),
    ].join('');
  }

  private buffersTooltip(cmp: PlanNode): string {
    let text = '';
    let hit;
    let read;
    let written;
    let dirtied;
    switch (this.buffersMetric) {
      case BuffersMetric.shared:
        hit = cmp.node[NodeProp.SHARED_HIT_BLOCKS];
        read = cmp.node[NodeProp.SHARED_READ_BLOCKS];
        written = cmp.node[NodeProp.SHARED_WRITTEN_BLOCKS];
        dirtied = cmp.node[NodeProp.SHARED_DIRTIED_BLOCKS];
        break;
      case BuffersMetric.temp:
        hit = cmp.node[NodeProp.TEMP_HIT_BLOCKS];
        read = cmp.node[NodeProp.TEMP_READ_BLOCKS];
        written = cmp.node[NodeProp.TEMP_WRITTEN_BLOCKS];
        dirtied = cmp.node[NodeProp.TEMP_DIRTIED_BLOCKS];
        break;
      case BuffersMetric.local:
        hit = cmp.node[NodeProp.LOCAL_HIT_BLOCKS];
        read = cmp.node[NodeProp.LOCAL_READ_BLOCKS];
        written = cmp.node[NodeProp.LOCAL_WRITTEN_BLOCKS];
        dirtied = cmp.node[NodeProp.LOCAL_DIRTIED_BLOCKS];
        break;
    }
    text += hit ? '<br>Hit: ' + rows(hit) : '';
    text += read ? '<br>Read: ' + rows(read) : '';
    text += written ? '<br>Written: ' + rows(written) : '';
    text += dirtied ? '<br>Dirtied: ' + rows(dirtied) : '';
    text = text ? text : ' N/A';
    switch (this.buffersMetric) {
      case BuffersMetric.shared:
        text = 'Shared Blocks:' + text;
        break;
      case BuffersMetric.temp:
        text = 'Temp Blocks:' + text;
        break;
      case BuffersMetric.local:
        text = 'Local Blocks:' + text;
        break;
    }
    return text;
  }

  private nodeType(row: any[]): string {
    return row[1].node[NodeProp.NODE_TYPE];
  }
}
</script>