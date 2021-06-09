<template>
  <div class="app-container">
    <div class="list_search">
      <label for="module_search">模块名</label>
      <el-input
        id="module_search"
        v-model="module_search"
        style="width: 140px"
        clearable
        prefix-icon="el-icon-search"
      />
      <label for="branch_search">分支名</label>
      <el-input
        id="branch_search"
        v-model="branch_search"
        style="width: 250px"
        clearable
        prefix-icon="el-icon-search"
      />
      <label for="revision_search">版本号</label>
      <el-input
        id="revision_search"
        v-model="revision_search"
        style="width: 250px"
        clearable
        prefix-icon="el-icon-search"
      />
    </div>
    <el-table
      :data="
        list.data
          .filter(
            (data) =>
              (!module_search && !branch_search && !revision_search) ||
              (data.module
                .toLowerCase()
                .includes(module_search.toLowerCase()) &&
                data.branch
                  .toLowerCase()
                  .includes(branch_search.toLowerCase()) &&
                data.revision
                  .toLowerCase()
                  .includes(revision_search.toLowerCase()))
          )
          .slice((currentPage - 1) * pagesize, currentPage * pagesize)
      "
      style="width: 100%"
      stripe
    >
      <el-table-column prop="id" label="构建号" width="80"> </el-table-column>
      <el-table-column prop="module" label="模块名" width="140">
      </el-table-column>
      <el-table-column prop="branch" label="分支名"></el-table-column>
      <el-table-column prop="revision" label="版本号"> </el-table-column>
      <el-table-column prop="date" label="提交时间" width="140">
        <template slot-scope="scope">
          <i class="el-icon-time"></i>
          <span style="margin-left: 4px">{{ scope.row.date }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="test.begin" label="开始测试" width="80">
        <template slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-caret-right"
            circle
            size="mini"
            @click="startTest(scope.row)"
          ></el-button>
        </template>
      </el-table-column>
      <el-table-column prop="test.details" label="测试详情" width="80">
        <template slot-scope="scope"
          ><el-button
            type="info"
            icon="el-icon-s-order"
            circle
            size="mini"
            @click="showTest(scope.row)"
          ></el-button>
        </template>
      </el-table-column>
      <el-table-column prop="result" label="评估结果" width="80">
        <template slot-scope="scope">
          <el-button
            type="success"
            icon="el-icon-tickets"
            circle
            size="mini"
            @click="showresult(scope.row)"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
    <div style="text-align: center; margin-top: 30px">
      <el-pagination
        layout="prev, pager, next"
        :total="total"
        @current-change="current_change"
      >
      </el-pagination>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      id: '45180',
      gitversion: '',
      branchname: '',
      module_search: '',
      branch_search: '',
      revision_search: '',
      total: 0,
      currentPage: 1,
      pagesize: 10,
      list: {
        data: [
          { id: '45180', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-9 10:44', test: { begin: '45180', details: '45180' }, result: '45180' },
          // { id: '', module: '', branch: '', revison: '', date: '', test: { begin: '', details: '' }, result: '' },
          { id: '38593', module: 'tracking-service', branch: 'feature/20210520-android_channel', revision: 'a943aa3c6da7370b2bd4561e51c277f45f568163', date: '2021-6-9 10:42', test: { begin: '38593', details: '38593' }, result: '38593' },
          { id: '00005', module: 'action-service', branch: 'feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00004', module: 'action-service', branch: 'feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00003', module: 'action-service', branch: 'feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00002', module: 'action-service', branch: 'feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00005', module: 'action-service', branch: 'feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00004', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00003', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00002', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00005', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00004', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00003', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00002', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00005', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00004', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00003', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00002', module: 'action-service', branch: 'release/java.gdt.action_meta_service/feature/leflow_auto_create/202102021115', revision: '2d2efc060c1d69d179b20ae9b83b600c22b525f6', date: '2021-6-3 10:42', test: { begin: '45180', details: '45180' }, result: '45180' },
          { id: '00001', module: 'tracking-service', branch: 'feature/20210520-android_channel', revision: 'a943aa3c6da7370b2bd4561e51c277f45f568163', date: '2021-6-1 10:44', test: { begin: '38593', details: '38593' }, result: '38593' }
        ]
      }
    }
  },
  mounted() {
    this.total = this.list.data.length
  },
  methods: {
    showresult(row) {
      this.$router.push({
        path: '/result/show',
        query: {
          id: row.id,
          gitversion: row.gitversion,
          branchname: row.branchname
        }
      })
    },
    startTest(row) {
      console.log('start test :' + row.test.begin)
    },
    showTest(row) {
      console.log('show test :' + row.test.details)
    },
    empty() {
      this.module_search = ''
      this.branch_search = ''
      this.revision_search = ''
    },
    current_change: function (currentPage) {
      this.currentPage = currentPage;
    },
  }
}
</script>

<style>
.list_search {
  font-size: 14px;
  padding-bottom: 15px;
}
label {
  color: #606266;
  font-weight: normal;
  padding: 10px;
}
.el-table th {
  background-color: #f6f7fb;
  color: #666;
}
</style>
