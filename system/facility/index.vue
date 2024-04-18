<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="类别id" prop="tId">
        <el-select v-model="queryParams.tId" placeholder="请选择类别id" clearable>
          <el-option
            v-for="dict in dict.type.sys_facility_type"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="设施名称" prop="sName">
        <el-input
          v-model="queryParams.sName"
          placeholder="请输入设施名称"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="宗派名称" prop="sSect">
        <el-input
          v-model="queryParams.sSect"
          placeholder="请输入宗派名称"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="经度" prop="sLongitude">
        <el-input
          v-model="queryParams.sLongitude"
          placeholder="请输入经度"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="纬度" prop="sLatitude">
        <el-input
          v-model="queryParams.sLatitude"
          placeholder="请输入纬度"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="详细地址名称" prop="sAddress">
        <el-input
          v-model="queryParams.sAddress"
          placeholder="请输入详细地址名称"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="地区id" prop="aId">
        <el-input
          v-model="queryParams.aId"
          placeholder="请输入地区id"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="设施提示语" prop="sTip">
        <el-input
          v-model="queryParams.sTip"
          placeholder="请输入设施提示语"
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
          v-hasPermi="['system:facility:add']"
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
          v-hasPermi="['system:facility:edit']"
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
          v-hasPermi="['system:facility:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['system:facility:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="facilityList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="设施主键id" align="center" prop="sId" />
      <el-table-column label="类别id" align="center" prop="tId">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_facility_type" :value="scope.row.tId"/>
        </template>
      </el-table-column>
      <el-table-column label="设施名称" align="center" prop="sName" />
      <el-table-column label="宗派名称" align="center" prop="sSect" />
      <el-table-column label="经度" align="center" prop="sLongitude" />
      <el-table-column label="纬度" align="center" prop="sLatitude" />
      <el-table-column label="详细地址名称" align="center" prop="sAddress" />
      <el-table-column label="地区名称" align="center" prop="aName" />
      <el-table-column label="设施提示语" align="center" prop="sTip" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['system:facility:edit']"
          >修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['system:facility:remove']"
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

    <!-- 添加或修改设施管理对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body @close="handleClose">
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="类别id" prop="tId">
          <el-select v-model="form.tId" placeholder="请选择类别id">
            <el-option
              v-for="dict in dict.type.sys_facility_type"
              :key="dict.value"
              :label="dict.label"
              :value="parseInt(dict.value)"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="设施名称" prop="sName">
          <el-input v-model="form.sName" placeholder="请输入设施名称" />
        </el-form-item>
        <el-form-item label="宗派名称" prop="sSect">
          <el-input v-model="form.sSect" placeholder="请输入宗派名称" />
        </el-form-item>
        <el-form-item label="经度" prop="sLongitude">
          <el-input v-model="form.sLongitude" placeholder="请输入经度" />
        </el-form-item>
        <el-form-item label="纬度" prop="sLatitude">
          <el-input v-model="form.sLatitude" placeholder="请输入纬度" />
        </el-form-item>
        <el-form-item label="详细地址名称" prop="sAddress">
          <el-input v-model="form.sAddress" placeholder="请输入详细地址名称" />
        </el-form-item>




        <el-form-item label="地区名称" prop="aId">
          <el-select v-model="form.aId" filterable remote reserve-keyword placeholder="请输入地区名称"
              :remote-method="remoteMethod" :loading="loading" style="width:100%" clearable>
              <el-option v-for="item in areaList" :key="item.aId"  :label="item.municipality" :value="item.aId"/>
          </el-select>
        </el-form-item>




        <el-form-item label="设施图片存放地址" prop="sImg">
          <image-upload v-model="form.sImg"/>
        </el-form-item>
        <el-form-item label="设施视频存放地址" prop="sVideo">
          <file-upload :fileSize="100" :fileType="videoTypes" v-model="form.sVideo"/>
        </el-form-item>
        <el-form-item label="设施提示语" prop="sTip">
          <el-input v-model="form.sTip" placeholder="请输入设施提示语" />
        </el-form-item>
        <el-form-item label="设施详情介绍" prop="sDetails">
          <el-input v-model="form.sDetails" type="textarea" placeholder="请输入内容" />
        </el-form-item>
        <el-form-item label="设施资料展示pdf" prop="sData">
          <file-upload v-model="form.sData"/>
        </el-form-item>
        <el-form-item label="其他" prop="sNotes">
          <el-input v-model="form.sNotes" type="textarea" placeholder="请输入内容" />
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
import { listFacility, getFacility, delFacility, addFacility, updateFacility } from "@/api/system/facility";
import { listArea} from "@/api/system/area";
export default {
  name: "Facility",
  dicts: ['sys_facility_type'],
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
      // 设施管理表格数据
      facilityList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        tId: null,
        sName: null,
        sSect: null,
        sLongitude: null,
        sLatitude: null,
        sAddress: null,
        aId: null,
        sTip: null,
        municipality:''
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
      },
      videoTypes:[
        "mp4","flv","rmvb","mpg"
      ],

      queryParams: {
        pageNum: 1,
        pageSize: 10,
        country: null,
        province: null,
        municipality: ''
      },
      areaList:[]
    };
  },
  created() {
    this.getList();
    this.$nextTick(() => {
      listArea(this.queryParams).then(response => {
        this.areaList = response.rows;
      });
    })
  },
  methods: {
    handleClose() {
      listArea(this.queryParams).then(response => {
        this.areaList = response.rows;
      });
    },
    remoteMethod(query) {
      if (query) {
        const params = {
          pageNum: 1,
          pageSize: 10,
          municipality:query
        }
        this.loading = true;
        listArea(params).then(response => {
          this.areaList = response.rows;
          this.loading = false;
          console.log(this.areaList)
        });
      } else {
        this.areaList = []
      }
    },




    /** 查询设施管理列表 */
    getList() {
      this.loading = true;
      listFacility(this.queryParams).then(response => {
        this.facilityList = response.rows;
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
        sId: null,
        tId: null,
        aName:null,
        sName: null,
        sSect: null,
        sLongitude: null,
        sLatitude: null,
        sAddress: null,
        aId: null,
        sImg: null,
        sVideo: null,
        sTip: null,
        sDetails: null,
        sData: null,
        sNotes: null
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
      this.ids = selection.map(item => item.sId)
      this.single = selection.length!==1
      this.multiple = !selection.length
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset();
      this.open = true;
      this.title = "添加设施管理";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const sId = row.sId || this.ids
      getFacility(sId).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "修改设施管理";
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.sId != null) {
            updateFacility(this.form).then(response => {
              this.$modal.msgSuccess("修改成功");
              this.open = false;
              this.getList();
            });
          } else {
            addFacility(this.form).then(response => {
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
      const sIds = row.sId || this.ids;
      this.$modal.confirm('是否确认删除设施管理编号为"' + sIds + '"的数据项？').then(function() {
        return delFacility(sIds);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('system/facility/export', {
        ...this.queryParams
      }, `facility_${new Date().getTime()}.xlsx`)
    }
  }
};
</script>
