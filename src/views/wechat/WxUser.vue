<template>
  <el-card class="main-card">
    <div class="title">{{ this.$route.name }}</div>
    <!-- 表格操作 -->
    <div class="operation-container">
      <el-button
        type="success"
        size="small"
        icon="el-icon-s-promotion"
        @click="sendMessage"
      >
        发送通知
      </el-button>
      <!-- 条件筛选 -->
      <div style="margin-left: auto">
        <el-input
          v-model="keywords"
          prefix-icon="el-icon-search"
          size="small"
          placeholder="请输入昵称"
          style="width: 200px"
          @keyup.enter.native="searchUsers"
        />
        <el-button
          type="primary"
          size="small"
          icon="el-icon-search"
          style="margin-left: 1rem"
          @click="searchUsers"
        >
          搜索
        </el-button>
      </div>
    </div>
    <!-- 表格展示 -->
    <el-table border :data="wxUserList" v-loading="loading">
      <!-- 表格列 -->
      <el-table-column prop="id" label="ID" align="center" width="100">
      </el-table-column>
      <el-table-column
        prop="tempId"
        label="模板ID"
        align="center"
        width="200"
      />
      <el-table-column prop="openId" label="微信ID" align="center">
      </el-table-column>

      <el-table-column prop="tempId" label="模板ID" align="center" />
      <el-table-column
        prop="myBirthday"
        label="自己生日"
        align="center"
      />
      <el-table-column
        prop="babyBirthday"
        label="情侣生日"
        align="center"
      />

      <el-table-column
        prop="loveDay"
        label="相恋时间"
        align="center"
      />
      <el-table-column
        prop="area"
        label="城市天气"
        align="center"
      />

      <el-table-column prop="isDeleted" label="禁用" align="center" width="100">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.isDeleted"
            active-color="#13ce66"
            inactive-color="#F4F4F5"
            :active-value="1"
            :inactive-value="0"
            @change="changeDisable(scope.row)"
          />
        </template>
      </el-table-column>

      <el-table-column
        prop="createTime"
        label="创建时间"
        width="130"
        align="center"
      >
        <template slot-scope="scope">
          <i class="el-icon-time" style="margin-right: 5px" />
          {{ scope.row.createTime | date }}
        </template>
      </el-table-column>
      <el-table-column
        prop="lastLoginTime"
        label="修改时间"
        width="130"
        align="center"
      >
        <template slot-scope="scope">
          <i class="el-icon-time" style="margin-right: 5px" />
          {{ scope.row.lastLoginTime | date }}
        </template>
      </el-table-column>
      <!-- 列操作 -->
      <el-table-column label="操作" align="center" width="200" >
        <template slot-scope="scope">
          <el-button
            type="success"
            size="mini"
            @click="sendMessage(scope.row)"
          >
            发送通知
          </el-button>
          <el-button
            type="primary"
            size="mini"
            @click="openEditModel(scope.row)"
          >
            编辑
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      class="pagination-container"
      background
      @size-change="sizeChange"
      @current-change="currentChange"
      :current-page="current"
      :page-size="size"
      :total="count"
      :page-sizes="[10, 20]"
      layout="total, sizes, prev, pager, next, jumper"
    />
    <!-- 修改对话框 -->
    <el-dialog :visible.sync="isEdit" width="30%">
      <div class="dialog-title-container" slot="title">修改用户</div>
      <el-form label-width="60px" size="medium" :model="wxUserForm">
        <el-form-item label="模板ID">
          <el-input v-model="wxUserForm.tempId" style="width: 220px" />
        </el-form-item>
        <el-form-item label="自己生日">
          <el-input v-model="wxUserForm.myBirthday" style="width: 220px" />
        </el-form-item>
        <el-form-item label="情侣生日">
          <el-input v-model="wxUserForm.babyBirthday" style="width: 220px" />
        </el-form-item>
        <el-form-item label="相恋时间">
          <el-input v-model="wxUserForm.loveDay" style="width: 220px" />
        </el-form-item>
        <el-form-item label="城市天气">
          <el-input v-model="wxUserForm.area" style="width: 220px" />
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="isEdit = false">取 消</el-button>
        <el-button type="primary" @click="editUserRole"> 确 定 </el-button>
      </div>
    </el-dialog>
  </el-card>
</template>
  
  <script>
export default {
  created() {
    this.listUsers();
  },
  data: function () {
    return {
      loading: true,
      isEdit: false,
      wxUserForm: {},
      userIdWx: null,
      wxUserList: [],
      keywords: null,
      current: 1,
      size: 10,
      count: 0,
    };
  },
  methods: {
    searchUsers() {
      this.current = 1;
      this.listUsers();
    },
    sizeChange(size) {
      this.size = size;
      this.listUsers();
    },
    currentChange(current) {
      this.current = current;
      this.listUsers();
    },
    changeDisable(wxUser) {
      this.axios.post("/api/admin/wxUser/update", {
        id: wxUser.id,
        isDeleted: wxUser.isDeleted,
      });
    },
    openEditModel(wxUser) {
      this.wxUserForm = JSON.parse(JSON.stringify(wxUser));
      this.isEdit = true;
    },
    editUserRole() {
      this.wxUserForm.roleIdList = this.roleIdList;
      this.axios
        .post("/api/admin/wxUser/update", this.wxUserForm)
        .then(({ data }) => {
          if (data.flag) {
            this.$notify.success({
              title: "成功",
              message: data.message,
            });
            this.listUsers();
          } else {
            this.$notify.error({
              title: "失败",
              message: data.message,
            });
          }
          this.isEdit = false;
        });
    },
    listUsers() {
      this.axios
        .get("/api/admin/wxUser", {
          params: {
            current: this.current,
            size: this.size,
            id: " ",
            openId: " ",
            userIdWx: " ",
          },
        })
        .then(({ data }) => {
          this.wxUserList = data.data.recordList;
          this.count = data.data.count;
          this.loading = false;
        });
    },
    sendMessage(wxUser) {
      this.axios
        .get("/api/wechat/sendMsg", {
          params: {
            openId: wxUser.openId
          },
        })
        .then(({ data }) => {
          if (data.flag) {
          this.$notify.success({
            title: "成功",
            message: data.message
          });
          this.listRoles();
        } else {
          this.$notify.error({
            title: "失败",
            message: data.message
          });
        }
        });
    },
    sendMessageOne(wxUser) {
      this.axios.get("/api/wechat/sendMsg", {
        openId: wxUser.openId
      });
    },
  },
  watch: {
    loginType() {
      this.current = 1;
      this.listUsers();
    },
  },
};
</script>
  