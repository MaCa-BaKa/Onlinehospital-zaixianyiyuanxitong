<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="96px">
      <el-form-item label="宝物名称" prop="trName">
        <el-input
          v-model="queryParams.trName"
          placeholder="请输入宝物名称"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="关键人物名称" prop="pName">
        <el-input
          v-model="queryParams.pName"
          placeholder="请输入关键人物名称"
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
          v-hasPermi="['system:keyman_1:add']"
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
          v-hasPermi="['system:keyman_1:edit']"
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
          v-hasPermi="['system:keyman_1:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['system:keyman_1:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="keyman_1List" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="宝物和人物关系id" align="center" prop="id" />
      <el-table-column label="宝物名称" align="center" prop="trName" />
      <el-table-column label="关键人物名称" align="center" prop="pName" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['system:keyman_1:edit']"
          >修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['system:keyman_1:remove']"
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

    <!-- 添加或修改宝物和人物关系表对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body @close="handleClose">
      <el-form ref="form" :model="form" :rules="rules" label-width="100px">
        <el-form-item label="宝物名称" prop="trId">
          <!-- <el-input v-model="form.trId" placeholder="请输入宝物id" /> -->
          <el-select v-model="form.trId" filterable remote reserve-keyword placeholder="请输入宝物名称"
              :remote-method="remoteMethod" :loading="loading" style="width:100%" clearable>
              <el-option v-for="item in options" :key="item.trId"  :label="item.trName" :value="item.trId"/>
          </el-select>
        </el-form-item>
        <el-form-item label="关键人物名称" prop="pId">
          <!-- <el-input v-model="form.pId" placeholder="请输入关键人物id" /> -->
          <el-select v-model="form.pId" filterable remote reserve-keyword placeholder="请输入关键人物名称"
              :remote-method="remoteMethod2" :loading="loading" style="width:100%" clearable>
              <el-option v-for="item in options2" :key="item.pId"  :label="item.pName" :value="item.pId"/>
          </el-select>
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
import { listKeyman_1, getKeyman_1, delKeyman_1, addKeyman_1, updateKeyman_1 } from "@/api/system/keyman_1";
import { listTreasure } from "@/api/system/treasure";
import { listKeyman} from "@/api/system/keyman";
import { mount } from "sortablejs";

export default {
  name: "Keyman_1",
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
      // 宝物和人物关系表表格数据
      keyman_1List: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        trId: null,
        pId: null,
        trName:null,
        pName:null
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
      },


      options:[],
      options2:[]
    };
  },
  created() {
    this.getList();
    this.$nextTick(() => {
      listTreasure().then(response => {
        this.options = response.rows
      })
      listKeyman().then(response => {
        this.options2 = response.rows
      })
    })
    
  },
  methods: {
    handleClose() {
      listTreasure().then(response => {
        this.options = response.rows
      })
      listKeyman().then(response => {
        this.options2 = response.rows
      })
    },
    // 宝物模糊查询输入框列表
    remoteMethod(query) {
      if (query) {
        const params = {
          trName:query
        }
        this.loading = true;
        listTreasure(params).then(response => {
          this.options = response.rows
          console.log(this.options)
          this.loading = false;
        })
      } else {
        this.options = []
      }
    },

    // 关键人物模糊查询输入框列表
    remoteMethod2(query) {
      if (query) {
        const params = {
          pName:query
        }
        this.loading = true;
        listKeyman(params).then(response => {
          this.options2 = response.rows
          this.loading = false;
        })
      } else {
        this.options2 = []
      }
    },





    /** 查询宝物和人物关系表列表 */
    getList() {
      this.loading = true;
      listKeyman_1(this.queryParams).then(response => {
        this.keyman_1List = response.rows;
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
        id: null,
        trId: null,
        pId: null
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
      this.ids = selection.map(item => item.id)
      this.single = selection.length!==1
      this.multiple = !selection.length
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset();
      this.resetQuery()
      this.open = true;
      this.title = "添加宝物和人物关系表";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const id = row.id || this.ids
      getKeyman_1(id).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "修改宝物和人物关系表";
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.id != null) {
            updateKeyman_1(this.form).then(response => {
              this.$modal.msgSuccess("修改成功");
              this.open = false;
              this.getList();
            });
          } else {
            addKeyman_1(this.form).then(response => {
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
      const ids = row.id || this.ids;
      this.$modal.confirm('是否确认删除宝物和人物关系表编号为"' + ids + '"的数据项？').then(function() {
        return delKeyman_1(ids);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('system/keyman_1/export', {
        ...this.queryParams
      }, `keyman_1_${new Date().getTime()}.xlsx`)
    }
  }
};
</script>
