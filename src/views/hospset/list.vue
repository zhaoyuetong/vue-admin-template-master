<template>
  <div class="app-container">
    <el-form :inline="true" class="demo-form-inline">
      <el-form-item label="医院名称">
        <el-input v-model="searchObj.hosname" placeholder="医院名称"></el-input>
      </el-form-item>
      <el-form-item label="医院编号">
        <el-input v-model="searchObj.hoscode" placeholder="医院编号">
        </el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="getList()">查询</el-button>
      </el-form-item>
    </el-form>
    <!-- banner列表 -->
    <div>
      <el-button type="danger" size="mini" @click="removeRows()">批量删除</el-button>
    </div>
    <el-table :data="list" stripe style="width: 100%" ref="multipleTable" tooltip-effect="dark"
      @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55">
      </el-table-column>
      <el-table-column type="index" width="50" />
      <el-table-column prop="hosname" label="医院名称" />
      <el-table-column prop="hoscode" label="医院编号" />
      <el-table-column prop="apiUrl" label="api基础路径" width="200" />
      <el-table-column prop="contactsName" label="联系人姓名" />
      <el-table-column prop="contactsPhone" label="联系人手机" />


      <el-table-column label="状态" align="center" width="180">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.status" active-color="#00A854" active-text="可用" :active-value="1"
            inactive-color="#F04134" inactive-text="不可用" :inactive-value="0"
            @change="lockHostSet(scope.row.id, scope.row.status)">
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column label="删除" width="180">
        <template slot-scope="scope">
          <el-button type="danger" icon="el-icon-delete" circle @click="deleteDataById(scope.row.id)"></el-button>
          <router-link :to="'/hospSet/edit/' + scope.row.id">
            <el-button type="primary" size="mini" icon="el-icon-edit"></el-button>
          </router-link>
        </template>

      </el-table-column>
    </el-table>
    <el-pagination background :page-size="limit" :total="total" :current-page="current"
      style="padding: 30px 0; text-align: center;" layout="total, prev, pager, next, jumper"
      @current-change="getList" />
  </div>
</template>
<script>
import hospset from '@/api/hospset'

export default {
  data() {
    return {
      current: 1,//当前页
      limit: 3,//每页限制数
      searchObj: {},//查询条件封装对象
      list: [],//每页数据集合
      total: 0,
      multipleSelection: [],//批量删除变量
    }
  },
  created() {
    this.getList()
  },
  methods: {
    deleteDataById1() {
      hospset.downland()
    },
    handleSelectionChange(selection) {
      this.multipleSelection = selection
    },
    getList(page = 1) {
      this.current = page
      hospset.getHospSetList(this.current, this.limit, this.searchObj)
        .then((response) => {
          this.list = response.data.records;
          this.total = response.data.total;
          console.log(response)
        })
        .catch(((error) => {
          console.log(error)
        }))
    },
    deleteDataById() {
      this.$confirm('此操作将永久删除医院设置信息, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        // .then((response) => {

        // })
        // .catch(((error) => {

        // }))
        this.$message({
          type: 'success',
          message: '删除成功!'
        });
        this.getList(1);
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    removeRows() {
      if (this.multipleSelection.length === 0) {
        this.$message({
          message: '警告，请选择一条要删除的数据',
          type: 'warning'
        });
        return;
      }
      this.$confirm('此操作将永久删除医院设置信息, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        var idList = [];
        for (var i of this.multipleSelection) {
          idList.push(i.id);
        }
        hospset.batchDeleteHospSet(idList);
        this.$message({
          type: 'success',
          message: '删除成功!'
        });
        this.getList(1);
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    //锁定和取消锁定
    lockHostSet(id, status) {
      hospset.lockHospSet(id, status)
        .then(response => {
          //刷新
          // this.getList()
        })
    },
    handleClick(row) {
      console.log(row);
    }
  }
}
</script>