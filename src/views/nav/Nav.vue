<template>
  <el-card class="main-card">
    <!-- 标题 -->
    <div class="title">{{ this.$route.name }}</div>
    <div class="operation-container">
      <el-button
        type="primary"
        size="small"
        icon="el-icon-plus"
        @click="openModel(null)"
      >
        新增导航栏
      </el-button>
      <!-- 数据筛选 -->
      <div style="margin-left: auto">
        <el-input
          v-model="keywords"
          prefix-icon="el-icon-search"
          size="small"
          placeholder="请输入导航栏名"
          style="width: 200px"
          @keyup.enter.native="listNavs"
        />
        <el-button
          type="primary"
          size="small"
          icon="el-icon-search"
          style="margin-left: 1rem"
          @click="listNavs"
        >
          搜索
        </el-button>
      </div>
    </div>
    <!-- 权限列表 -->
    <el-table
      v-loading="loading"
      :data="navList"
      row-key="id"
      :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
    >
      
      <!-- 导航栏名称 -->
      <el-table-column prop="name" label="导航栏名称" width="140" />
      <!-- 导航栏icon -->
      <el-table-column prop="icon" align="center" label="图标" width="100">
        <template slot-scope="scope">
          <i :class="'iconfont ' + scope.row.icon" />
        </template>
      </el-table-column>
      <!-- 导航栏排序 -->
      <el-table-column
        prop="orderNum"
        align="center"
        label="排序"
        width="100"
      />
      <!-- 访问路径 -->
      <el-table-column prop="url" label="访问路径" />
      <!-- 是否隐藏 -->
      <el-table-column prop="isHidden" label="隐藏" align="center" width="80">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.isHidden"
            active-color="#13ce66"
            inactive-color="#F4F4F5"
            :active-value="1"
            :inactive-value="0"
            @change="changeDisable(scope.row)"
          />
        </template>
      </el-table-column>
      <!-- 创建时间 -->
      <el-table-column
        prop="createTime"
        label="创建时间"
        align="center"
        width="150"
      >
        <template slot-scope="scope">
          <i class="el-icon-time" style="margin-right: 5px" />
          {{ scope.row.createTime | date }}
        </template>
      </el-table-column>
      <!-- 操作 -->
      <el-table-column label="操作" align="center" width="200">
        <template slot-scope="scope">
          <el-button
            type="text"
            size="mini"
            @click="openModel(scope.row, 1)"
            v-if="scope.row.children"
          >
            <i class="el-icon-plus" /> 新增
          </el-button>
          <el-button type="text" size="mini" @click="openModel(scope.row, 2)">
            <i class="el-icon-edit" /> 修改
          </el-button>
          <el-popconfirm
            title="确定删除吗？"
            style="margin-left: 10px"
            @confirm="deleteNav(scope.row.id)"
          >
            <el-button size="mini" type="text" slot="reference">
              <i class="el-icon-delete" /> 删除
            </el-button>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
    <!-- 新增模态框 -->
    <el-dialog :visible.sync="addNav" width="30%" top="12vh">
      <div class="dialog-title-container" slot="title" ref="navTitle" />
      <el-form label-width="80px" size="medium" :model="navForm">
      
        <!-- 导航栏名称 -->
        <el-form-item label="导航名称">
          <el-input v-model="navForm.name" style="width: 220px" />
        </el-form-item>
        <!-- 导航栏图标 -->
        <el-form-item label="导航图标">
          <el-popover placement="bottom-start" width="300" trigger="click">
            <el-row>
              <el-col
                v-for="(item, index) of iconList"
                :key="index"
                :md="12"
                :gutter="10"
              >
                <div class="icon-item" @click="checkIcon(item)">
                  <i :class="'iconfont ' + item" /> {{ item }}
                </div>
              </el-col>
            </el-row>
            <el-input
              :prefix-icon="'iconfont ' + navForm.icon"
              slot="reference"
              v-model="navForm.icon"
              style="width: 220px"
            />
          </el-popover>
        </el-form-item>
        <!-- 路由地址 -->
        <el-form-item label="访问路径">
          <el-input v-model="navForm.url" style="width: 220px" />
        </el-form-item>
        <!-- 导航父类 -->
        <el-form-item label="导航父类">
        <el-select v-model="navForm.parentId" placeholder="请选择">
        <el-option label="无" value="0"></el-option>
          <el-option
            v-for="item in navList"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          >
          </el-option>
        </el-select>
      </el-form-item>
        <!-- 显示排序 -->
        <el-form-item label="显示排序">
          <el-input-number
            v-model="navForm.orderNum"
            controls-position="right"
            :min="1"
            :max="10"
          />
        </el-form-item>
        <!-- 显示状态 -->
        <el-form-item label="显示状态">
          <el-radio-group v-model="navForm.isHidden">
            <el-radio :label="0">显示</el-radio>
            <el-radio :label="1">隐藏</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="addNav = false">取 消</el-button>
        <el-button type="primary" @click="saveOrUpdateNav"> 确 定 </el-button>
      </div>
    </el-dialog>
  </el-card>
</template>

<script>
export default {
  created() {
    this.listNavs();
    this.listPages();
  },
  data() {
    return {
      keywords: "",
      loading: true,
      addNav: false,
      show: true,
      navList: [],
      pageList: [],
      pageForum: {
        id: "",
        pageName: "",
        pageLabel: "",
        pageCover: ""
      },
      navForm: {
        id: "",
        name: "",
        icon: "",
        url: "",
        orderNum: 1,
        parentId: "",
        isHidden: 0,
      },
      iconList: [
        "el-icon-myshouye",
        "el-icon-myfabiaowenzhang",
        "el-icon-myyonghuliebiao",
        "el-icon-myxiaoxi",
        "el-icon-myliuyan",
        "el-icon-myshouye",
        "el-icon-myfabiaowenzhang",
        "el-icon-myyonghuliebiao",
        "el-icon-myxiaoxi",
        "el-icon-myliuyan",
        "iconpinglunzu",
        "iconxiangce1"
      ],
    };
  },
  methods: {
    listNavs() {
      this.axios
        .get("/api/admin/nav", {
          params: {
            keywords: this.keywords,
          },
        })
        .then(({ data }) => {
          this.navList = data.data;
          this.loading = false;
        });
    },
    listPages() {
      this.axios.get("/api/admin/pages").then(({ data }) => {
        this.pageList = data.data;
        this.loading = false;
      });
    },
    openModel(nav, type) {
      if (nav) {
        this.show = false;
        switch (type) {
          case 1:
            this.navForm = {
              id: null,
              name: "",
              icon: "",
              component: "",
              url: "",
              orderNum: 1,
              parentId: null,
              isHidden: 0,
            };
            this.$refs.navTitle.innerHTML = "新增导航栏";
            this.navForm.parentId = JSON.parse(JSON.stringify(nav.id));
            break;
          case 2:
            this.$refs.navTitle.innerHTML = "修改导航栏";
            console.log(nav);
            console.log(this.navForm);
            this.navForm = JSON.parse(JSON.stringify(nav));
            break;
        }
      } else {
        this.$refs.navTitle.innerHTML = "新增导航栏";
        this.show = true;
        this.navForm = {
          id: null,
          name: "",
          icon: "",
          url: "",
          orderNum: 1,
          parentId: null,
          isHidden: 0,
        };
      }
      this.addNav = true;
    },
    checkIcon(icon) {
      this.navForm.icon = icon;
    },
    saveOrUpdateNav() {
      if (this.navForm.name.trim() == "") {
        this.$message.error("导航栏名不能为空");
        return false;
      }
      if (this.navForm.icon.trim() == "") {
        this.$message.error("导航栏icon不能为空");
        return false;
      }

      this.axios.post("/api/admin/nav", this.navForm).then(({ data }) => {
        if (data.flag) {
          this.$notify.success({
            title: "成功",
            message: "操作成功",
          });
          this.listNavs();
        } else {
          this.$notify.error({
            title: "失败",
            message: "操作失败",
          });
        }
        this.addNav = false;
      });
    },
    deleteNav(id) {
      this.axios.delete("/api/admin/nav/" + id).then(({ data }) => {
        if (data.flag) {
          this.$notify.success({
            title: "成功",
            message: "删除成功",
          });
          this.listNavs();
        } else {
          this.$notify.error({
            title: "失败",
            message: data.message,
          });
        }
      });
    },
    setNav(val) {
      console.log(val);
      this.pageList.forEach((item, index)=>{
        if (item.id == val) {
          this.navForm.name = item.pageName;
          this.navForm.url = "/" + item.pageLabel;
        }
      });
    }
  },
};
</script>

<style scoped>
.icon-item {
  cursor: pointer;
  padding: 0.5rem 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>
