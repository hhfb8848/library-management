<template>
  <div
    v-loading="loading"
    element-loading-text="生成随机推荐图书中"
    element-loading-spinner="el-icon-loading"
  >
    <!--    搜索表单-->

    <div style="margin-bottom: 20px">
      <span style="margin-right: 10px">推荐方式：</span>
      <el-switch
        v-model="recommendType"
        active-text="管理员推荐"
        inactive-text="随机推荐"
        @change="changeRecommendType"
      >
      </el-switch>
      <el-button
        type="success"
        slot="reference"
        style="margin-left: 50px"
        @click="open"
        v-show="disButton"
        >添加推荐</el-button
      >
    </div>

    <el-table :data="tableData" stripe row-key="id" default-expand-all>
      <el-table-column prop="id" label="编号" width="80"></el-table-column>
      <el-table-column prop="name" label="图书名称"></el-table-column>
      <el-table-column prop="bookNo" label="标准码"></el-table-column>
      <el-table-column
        prop="description"
        width="200"
        label="描述"
      ></el-table-column>
      <el-table-column prop="publishDate" label="出版日期"></el-table-column>
      <el-table-column prop="author" label="作者"></el-table-column>
      <el-table-column prop="publisher" label="出版社"></el-table-column>
      <el-table-column prop="category" label="分类"></el-table-column>
      <el-table-column prop="score" label="借书积分"></el-table-column>
      <el-table-column prop="nums" label="数量"></el-table-column>
      <el-table-column prop="cover" label="封面">
        <template v-slot="scope">
          <el-image
            :src="scope.row.cover"
            :preview-src-list="[scope.row.cover]"
          ></el-image>
        </template>
      </el-table-column>
      <el-table-column prop="createtime" label="创建时间"></el-table-column>
      <el-table-column prop="updatetime" label="更新时间"></el-table-column>
      <el-table-column label="操作" width="140">
        <template v-slot="scope">
          <!--          scope.row 就是当前行数据-->
          <el-popconfirm
            style="margin-left: 5px"
            title="您确定移除该图书的推荐吗？"
            @confirm="cancelRecommend(scope.row.id,true)"
          >
            <el-button type="danger" slot="reference">取消推荐</el-button>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>

    <!--    分页-->
    <div style="margin-top: 20px">
      <el-pagination
        background
        :current-page="params.pageNum"
        :page-size="params.pageSize"
        layout="prev, pager, next"
        @current-change="handleCurrentChange"
        :total="total"
      >
      </el-pagination>
    </div>

    <el-dialog title="图书列表" :visible.sync="dialogTableVisible" width="75%">
      <el-table
        :data="gridData"
        stripe
        row-key="id"
        default-expand-all
        @selection-change="handleSelectionChange"
        ref="multipleTable"
      >
        <el-table-column type="selection" width="55"> </el-table-column>
        <el-table-column prop="id" label="编号" width="80"></el-table-column>
        <el-table-column prop="name" label="图书名称"></el-table-column>
        <el-table-column prop="bookNo" label="标准码"></el-table-column>

        <el-table-column prop="publishDate" label="出版日期"></el-table-column>
        <el-table-column prop="author" label="作者"></el-table-column>
        <el-table-column prop="publisher" label="出版社"></el-table-column>
        <el-table-column prop="category" label="分类"></el-table-column>
        <el-table-column prop="nums" label="数量"></el-table-column>
      </el-table>
      <el-pagination
        background
        :current-page="params.pageNum"
        :page-size="params.pageSize"
        layout="prev, pager, next"
        @current-change="handleCurrentChange"
        :total="total"
      >
      </el-pagination>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogTableVisible = false">取 消</el-button>
        <el-button type="primary" @click="comfirm">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import request from "@/utils/request";
import Cookies from "js-cookie";

export default {
  name: "BookList",
  data() {
    return {
      admin: Cookies.get("admin") ? JSON.parse(Cookies.get("admin")) : {},
      dialogTableVisible: false,
      gridData: [],
      tableData: [],
      recommendType: false,
      gridTotal: 0,
      total: 0,
      disButton: false,
      loading: false,
      multipleSelection: [],
      params: {
        pageNum: 1,
        pageSize: 10,
        isRecommend: 1,
        name: "",
        bookNo: "",
      },
    };
  },
    watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    recommendType: function (newQuestion, oldQuestion) {
      console.log("fdf");
      if (newQuestion) {
        this.disButton = true;
      }else{
        this.disButton = false
      }
    }
  },
  created() {
    this.load();
  },
  methods: {
    changeRecommendType() {
      if (!this.recommendType) {
        this.disButton = false;
        //随机
        if(this.tableData.length === 0){
          return
        }
        this.tableData.forEach((item)=>{
          this.cancelRecommend(item.id,false)
        })
      } else {
        this.disButton = true;
      }
    },
    comfirm() {
      console.log(this.multipleSelection);
      if (this.multipleSelection.length === 0) {
        this.$message.warning("请选择要推荐的图书");
        return;
      }
      let ids = [];
      this.multipleSelection.forEach((item) => {
        ids.push(item.id);
      });
      request.put("/book/recommend", ids).then((res) => {
        if (res.code === "200") {
          this.$notify.success("推荐成功");
          this.dialogTableVisible = false;
          this.load();
        }
      });
    },
    toggleSelection(rows) {
      if (rows) {
        rows.forEach((row) => {
          this.$refs.multipleTable.toggleRowSelection(row);
        });
      } else {
        this.$refs.multipleTable.clearSelection();
      }
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    open() {
      this.dialogTableVisible = true;
      this.params.isRecommend = 0;
      request
        .get("/book/page", {
          params: this.params,
        })
        .then((res) => {
          if (res.code === "200") {
            this.gridData = res.data.list;
            this.gridTotal = res.data.total;
          }
        });
    },
    load() {
      this.params.isRecommend = 1;
      request
        .get("/book/page", {
          params: this.params,
        })
        .then((res) => {
          if (res.code === "200") {
            this.tableData = res.data.list;
            this.total = res.data.total;
            if (this.tableData.length > 0) {
               this.recommendType=true
            }
           
          }
        });
    },
    reset() {
      this.params = {
        pageNum: 1,
        pageSize: 10,
        name: "",
        bookNo: "",
      };
      this.load();
    },
    handleCurrentChange(pageNum) {
      // 点击分页按钮触发分页
      this.params.pageNum = pageNum;
      this.load();
    },
    cancelRecommend(id,flag) {
      request.put("/book/cancelRecommend/" + id).then((res) => {
        if (res.code === "200") {
          if(flag){
               this.$notify.success("取消推荐成功");
          }
       
          this.load();
        } else {
          this.$notify.error(res.msg);
        }
      });
    },
  },
};
</script>

<style scoped></style>
