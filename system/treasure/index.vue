<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="宝物类别id" prop="tId">
        <el-input
          v-model="queryParams.tId"
          placeholder="请输入宝物类别id"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="宝物名称" prop="trName">
        <el-input
          v-model="queryParams.trName"
          placeholder="请输入宝物名称"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="宝物宗派类别" prop="trSect">
        <el-input
          v-model="queryParams.trSect"
          placeholder="请输入宝物宗派类别"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
          v-hasPermi="['system:treasure:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['system:treasure:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['system:treasure:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['system:treasure:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="treasureList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="宝物id" align="center" prop="trId" />
      <el-table-column label="宝物类别id" align="center" prop="tId" />
      <el-table-column label="宝物名称" align="center" prop="trName" />
      <el-table-column label="宝物宗派类别" align="center" prop="trSect" />
      <el-table-column label="宝物提示语" align="center" prop="trTip" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['system:treasure:edit']"
          >修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['system:treasure:remove']"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改宝物信息对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="宝物类别id" prop="tId">
          <el-input v-model="form.tId" placeholder="请输入宝物类别id" />
        </el-form-item>
        <el-form-item label="宝物名称" prop="trName">
          <el-input v-model="form.trName" placeholder="请输入宝物名称" />
        </el-form-item>
        <el-form-item label="宝物宗派类别" prop="trSect">
          <el-input v-model="form.trSect" placeholder="请输入宝物宗派类别" />
        </el-form-item>
        <el-form-item label="宝物图片地址" prop="trImg">
          <image-upload v-model="form.trImg"/>
        </el-form-item>
        <el-form-item label="宝物视频地址" prop="trVideo">
          <file-upload :fileSize="100" :fileType="videoTypes" v-model="form.trVideo"/>
        </el-form-item>
        <el-form-item label="宝物提示语" prop="trTip">
          <el-input v-model="form.trTip" type="textarea" placeholder="请输入内容" />
        </el-form-item>
        <el-form-item label="详情介绍" prop="trDetails">
          <el-input v-model="form.trDetails" type="textarea" placeholder="请输入内容" />
        </el-form-item>
        <el-form-item label="资料展示pdf" prop="trData">
          <file-upload v-model="form.trData"/>
        </el-form-item>
        <el-form-item label="其他" prop="notes">
          <el-input v-model="form.notes" type="textarea" placeholder="请输入内容" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { listTreasure, getTreasure, delTreasure, addTreasure, updateTreasure } from "@/api/system/treasure";

export default {
  name: "Treasure",
  data() {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 宝物信息表格数据
      treasureList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        tId: null,
        trName: null,
        trSect: null,
        trTip: null,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
      },
      videoTypes:[
        "mp4","flv","rmvb","mpg"
      ]
    };
  },
  created() {
    this.getList();
  },
  methods: {
    /** 查询宝物信息列表 */
    getList() {
      this.loading = true;
      listTreasure(this.queryParams).then(response => {
        this.treasureList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        trId: null,
        tId: null,
        trName: null,
        trSect: null,
        trImg: null,
        trVideo: null,
        trTip: null,
        trDetails: null,
        trData: null,
        notes: null
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.trId)
      this.single = selection.length!==1
      this.multiple = !selection.length
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset();
      this.open = true;
      this.title = "添加宝物信息";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const trId = row.trId || this.ids
      getTreasure(trId).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "修改宝物信息";
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.trId != null) {
            updateTreasure(this.form).then(response => {
              this.$modal.msgSuccess("修改成功");
              this.open = false;
              this.getList();
            });
          } else {
            addTreasure(this.form).then(response => {
              this.$modal.msgSuccess("新增成功");
              this.open = false;
              this.getList();
            });
          }
        }
      });
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const trIds = row.trId || this.ids;
      this.$modal.confirm('是否确认删除宝物信息编号为"' + trIds + '"的数据项？').then(function() {
        return delTreasure(trIds);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('system/treasure/export', {
        ...this.queryParams
      }, `treasure_${new Date().getTime()}.xlsx`)
    }
  }
};
</script>
