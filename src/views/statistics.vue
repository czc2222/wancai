<template>
  <Layout>
    <Tabs class-prefix="type" :data-source="recordTypeList" :value.sync="type"/>
    <!--    <Tabs class-prefix="interval" :data-source="intervalList" :value.sync="interval"/>-->
    <div>
      <ol  v-if="groupList.length>0">
        <li v-for="(group,index ) in groupList" :key="index">
          <h3 class="title"><span>{{ beautify(group.title) }}</span> <span>￥{{group.total}}</span> </h3>
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
        目前相关记录
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

@Component({
  components: {Tabs}

})
export default class Statistics extends Vue {
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
    return tags.length === 0 ? '无' : tags.map(tags=>tags.name).join(' ，');
  }

  get recordList() {
    return (this.$store.state as RootState).recordList;
  }

  get groupList() {
    const {recordList} = this;

    type Result = { title: dayjs.Dayjs; total: number;items: RecordItem[] }[]
    const newList = clone(recordList).filter(record => record.type === this.type)
        .sort((a, b) => dayjs(b.createdAt).valueOf() - dayjs(a.createdAt).valueOf());
    if (newList.length === 0) {return []}
    const result:Result = [{title: dayjs(newList[0].createdAt),total: 0, items: [newList[0]]}];

    for (let i = 1; i < newList.length; i++) {
      const currentGroup = newList[i];
      const lastGroup = result[result.length - 1];
      if (dayjs(lastGroup.title).isSame(dayjs(currentGroup.createdAt), 'day')) {
        lastGroup.items.push(currentGroup);
      } else {
        result.push({title: dayjs(currentGroup.createdAt),total:0, items: [currentGroup]});
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
.noResult{
  padding:16px;
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


</style>

