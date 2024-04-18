<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="设施名称" prop="sName">
        <el-input
          v-model="queryParams.sName"
          placeholder="请输入设施名称"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="人物名称" prop="pName">
        <el-input
          v-model="queryParams.pName"
          placeholder="请输入人物名称"
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
          v-hasPermi="['system:keyman_fac:add']"
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
          v-hasPermi="['system:keyman_fac:edit']"
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
          v-hasPermi="['system:keyman_fac:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['system:keyman_fac:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="keyman_facList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="设施和人物关系id" align="center" prop="id" />
      <el-table-column label="设施名称" align="center" prop="sName" />
      <el-table-column label="人物名称" align="center" prop="pName" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['system:keyman_fac:edit']"
          >修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['system:keyman_fac:remove']"
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




    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body @close="handleClose">
      <el-form ref="form" :model="form" :rules="rules" label-width="100px">
        <el-form-item label="设施名称" prop="sId">
          <!-- <el-input v-model="form.trId" placeholder="请输入宝物id" /> -->
          <el-select v-model="form.sId" filterable remote reserve-keyword placeholder="请输入关键人物名称"
              :remote-method="remoteMethod" :loading="loading" style="width:100%" clearable>
              <el-option v-for="item in option" :key="item.sId"  :label="item.sName" :value="item.sId"/>
          </el-select>
        </el-form-item>
        <el-form-item label="关键人物名称" prop="pId">
          <!-- <el-input v-model="form.pId" placeholder="请输入关键人物id" /> -->
          <el-select v-model="form.pId" filterable remote reserve-keyword placeholder="请输入关键人物名称"
              :remote-method="remoteMethod2" :loading="loading" style="width:100%" clearable>
              <el-option v-for="item in options" :key="item.pId"  :label="item.pName" :value="item.pId"/>
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
import { listKeyman_fac, getKeyman_fac, delKeyman_fac, addKeyman_fac, updateKeyman_fac } from "@/api/system/keyman_fac";
import { listKeyman} from "@/api/system/keyman";
import { listFacility } from "@/api/system/facility";
export default {
  name: "Keyman_fac",
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
      // 设施和人物关系表表格数据
      keyman_facList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        sId: null,
        pId: null,
        pName:'',
        sName:''
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
      },

      option:[],
      options:[]
    };
  },
  created() {
    this.getList();
    this.$nextTick(() => {
      listFacility().then(response => {
        this.option = response.rows
      })
      listKeyman().then(response => {
        this.options = response.rows
      })
    })
  },
  methods: {
    handleClose() {
      listFacility().then(response => {
        this.option = response.rows
      })
      listKeyman().then(response => {
        this.options = response.rows
      })
    },
    // 设施名称模糊查询输入框列表
  remoteMethod(query) {
      if (query) {
        const params = {
          sName:query
        }
        this.loading = true;
        listFacility(params).then(response => {
          this.option = response.rows
          this.loading = false;
        })
      } else {
        this.option = []
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
          this.options = response.rows
          this.loading = false;
        })
      } else {
        this.options = []
      }
    },
    







    /** 查询设施和人物关系表列表 */
    getList() {
      this.loading = true;
      listKeyman_fac(this.queryParams).then(response => {
        this.keyman_facList = response.rows;
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
        sId: null,
        pId: null,
        pName:'',
        sName:''
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
      this.open = true;
      this.title = "添加设施和人物关系表";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const id = row.id || this.ids
      getKeyman_fac(id).then(response => {
        this.form = response.data;
        // Object.assign(this.form,response.data)
        console.log(response.data)
        console.log(this.form)
        this.open = true;
        this.title = "修改设施和人物关系表";
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.id != null) {
            updateKeyman_fac(this.form).then(response => {
              this.$modal.msgSuccess("修改成功");
              this.open = false;
              this.getList();
            });
          } else {
            addKeyman_fac(this.form).then(response => {
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
      this.$modal.confirm('是否确认删除设施和人物关系表编号为"' + ids + '"的数据项？').then(function() {
        return delKeyman_fac(ids);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('system/keyman_fac/export', {
        ...this.queryParams
      }, `keyman_fac_${new Date().getTime()}.xlsx`)
    }
  }
};
</script>
