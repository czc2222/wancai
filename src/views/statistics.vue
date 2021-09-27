<template>
  <Layout>
    <Tabs class-prefix="type" :data-source="recordTypeList" :value.sync="type"/>
    <!--    <Tabs class-prefix="interval" :data-source="intervalList" :value.sync="interval"/>-->

    <div class="chart-Wrapper" ref="chartWrapper">
      <Charts class="chart" :options="chartList"/>
    </div>
    <div>
      <ol v-if="groupList.length>0">
        <li v-for="(group,index ) in groupList" :key="index">
          <h3 class="title"><span>{{ beautify(group.title) }}</span> <span>￥{{ group.total }}</span></h3>
          <ol>
            <li class="record" v-for="item in group.items" :key="item.id">
              <span>{{ tagString(item.tags) }}</span>
              <span class="notes">{{ item.notes }}</span>
              <span>￥{{ item.amount }}</span>
            </li>
          </ol>
        </li>
      </ol>
      <div v-else class="noResult">
        目前无相关记录
      </div>
    </div>


  </Layout>
</template>


<script lang="ts">
import {Component, Vue} from 'vue-property-decorator';
import Tabs from '@/components/Tabs.vue';
// import intervalList from '@/constants/intervalList';
import recordTypeList from '@/constants/recordTypeList';
import dayjs from 'dayjs';
import clone from '@/lib/clone';
import Charts from '@/components/Money/Charts.vue';
import _ from 'lodash'


@Component({
  components: {Tabs, Charts}

})
export default class Statistics extends Vue {
  get chartValues(){
    const today=new Date()
    const array=[]
    for (let i=0;i<=29;i++){
      const date=dayjs(today).subtract(i,'day').format('YYYY-MM-DD')
      const found=_.find(this.groupList,{title:date})
      array.push({day:date,value:found && found.total})

    }
    array.sort((a,b)=>{
      if(a.day > b.day){
        return 1
      }else if(a.day === b.day ){
        return 0
      }else {
        return -1
      }
    })
   return array

  }
  get chartList() {

    const days=this.chartValues.map(item=>item.day)
    const totalAmount=this.chartValues.map(item=>item.value)

    return {
      grid:{
        left:0,
        right:0
      },
      xAxis: {
        type: 'category',
        axisTick:{alignWithLabel:true},
        axisLine:{lineStyle:{color:'#666'}},
        data: days
      },
      yAxis: {
        type: 'value',
        show:false,
      },
      series: [{
        symbol:'circle',
        symbolSize:12,
        itemStyle:{borderWidth:1,color:'#666'},
        data: totalAmount,
        type: 'line'
      }],
      tooltip: {show: true,triggerOn:'click',position:'top',formatter:'{c}'}
    };

  }
mounted(){
    const div=this.$refs.chartWrapper
    div.scrollLeft=div.scrollWidth
}
  beautify(string: string) {
    const day = dayjs(string);
    const now = dayjs();
    if (day.isSame(now, 'day')) {
      return '今天';
    } else if (day.isSame(now.subtract(1, 'day'), 'day')) {
      return '昨天';
    } else if (day.isSame(now.subtract(2, 'day'), 'day')) {
      return '前天';
    } else if (day.isSame(now, 'year')) {
      return day.format('M月D日');
    } else {
      return day.format('YYYY年M月D日');
    }
  }

  tagString(tags: Tag[]) {
    return tags.length === 0 ? '无' : tags.map(tags => tags.name).join(' ，');
  }

  get recordList() {
    return (this.$store.state as RootState).recordList;
  }

  get groupList() {
    const {recordList} = this;

    type Result = { title: dayjs.Dayjs; total: number; items: RecordItem[] }[]
    const newList = clone(recordList).filter(record => record.type === this.type)
        .sort((a, b) => dayjs(b.createdAt).valueOf() - dayjs(a.createdAt).valueOf());
    if (newList.length === 0) {return [];}
    const result: Result = [{title: dayjs(newList[0].createdAt).format('YYYY-MM-DD'), total: 0, items: [newList[0]]}];

    for (let i = 1; i < newList.length; i++) {
      const currentGroup = newList[i];
      const lastGroup = result[result.length - 1];
      if (dayjs(lastGroup.title).isSame(dayjs(currentGroup.createdAt), 'day')) {
        lastGroup.items.push(currentGroup);
      } else {
        result.push({title: dayjs(currentGroup.createdAt).format('YYYY-MM-DD')
          , total: 0, items: [currentGroup]});
      }
    }
    result.map(group => {
      group.total = group.items.reduce((sum, item) => sum + item.amount, 0);
    });
    return result;
  }


  beforeCreate() {
    this.$store.commit('fetchRecords');
  }

  type = '-';
  interval = 'day';
  // intervalList = intervalList;
  recordTypeList = recordTypeList;
}
</script>


<style lang="scss" scoped>
.noResult {
  padding: 16px;
  text-align: center;
}

::v-deep .type-tabs-item {
  background: #c4c4c4;

  &.selected {

    background: white;

    &::after {
      display: none;
    }
  }
}

::v-deep .interval-tabs-item {
  height: 48px;
}

%item {
  padding: 0 16px;
  min-height: 40px;
  line-height: 24px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.title {
  @extend %item;
  display: flex;
  justify-content: space-between;
}

.record {
  background: white;
  @extend %item;

  > .notes {

    margin-right: auto;
    margin-left: 16px;
    padding: 0;
    color: #999;
    white-space: normal;
    word-wrap: break-word;
    word-break: break-all;
  }
}

.chart{
  width:430%;
  &-Wrapper{
    overflow:auto;
  }
}
</style>

