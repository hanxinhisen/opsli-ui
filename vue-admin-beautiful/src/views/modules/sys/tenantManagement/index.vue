<template>
  <div class="tenantManagement-container">
    <vab-query-form>
      <vab-query-form-left-panel :span="12">
        <el-button
          v-if="$perms('system_tenant_insert')"
          icon="el-icon-plus"
          type="primary"
          @click="handleInsert"
        > 添加 </el-button>

        <el-button
          v-if="$perms('system_tenant_delete')"
          :disabled="!selectRows.length > 0"
          icon="el-icon-delete"
          type="danger"
          @click="handleDelete"
        > 批量删除 </el-button>

      </vab-query-form-left-panel>
      <vab-query-form-right-panel :span="12">
        <el-form :inline="true" :model="queryForm" @submit.native.prevent>
          <el-form-item>
            <el-input
              v-model.trim="queryForm.id_EQ"
              placeholder="请输入租户ID"
              clearable
            />
          </el-form-item>
          <el-form-item>
            <el-input
              v-model.trim="queryForm.tenantName_LIKE"
              placeholder="请输入租户名称"
              clearable
            />
          </el-form-item>
          <el-form-item>
            <el-button icon="el-icon-search" type="primary" @click="queryData">
              查询
            </el-button>
          </el-form-item>
        </el-form>
      </vab-query-form-right-panel>
    </vab-query-form>

    <el-table
      v-loading="listLoading"
      :data="list"
      :element-loading-text="elementLoadingText"
      @selection-change="setSelectRows"
    >
      <el-table-column show-overflow-tooltip type="selection"></el-table-column>

      <el-table-column show-overflow-tooltip label="序号" width="95">
        <template slot-scope="scope">
          {{(queryForm.pageNo - 1) * queryForm.pageSize + scope.$index + 1}}
        </template>
      </el-table-column>

      <el-table-column
        show-overflow-tooltip
        prop="id"
        label="租户ID"
      ></el-table-column>

      <el-table-column
        show-overflow-tooltip
        prop="tenantName"
        label="租户名称"
      ></el-table-column>

      <el-table-column
        show-overflow-tooltip
        prop="izUsable"
        label="是否启用"
      >

        <template slot-scope="scope">
          <span>
            <el-tag v-if="scope.row.izUsable === '0' " type="info">
              {{ $getDictNameByValue('no_yes', scope.row.izUsable) }}
            </el-tag>
            <el-tag v-if="scope.row.izUsable === '1' " type="success">
              {{ $getDictNameByValue('no_yes', scope.row.izUsable) }}
            </el-tag>
          </span>
        </template>

      </el-table-column>

      <el-table-column
        show-overflow-tooltip
        prop="remark"
        label="备注"
      ></el-table-column>

      <el-table-column
        show-overflow-tooltip
        fixed="right"
        label="操作"
        width="200"
      >
        <template v-slot="scope">
          <el-button
            v-if="$perms('system_tenant_update')"
            type="text"
            @click="handleUpdate(scope.row)"
          > 编辑 </el-button>
          <el-button
            v-if="$perms('system_tenant_delete')"
            type="text"
            @click="handleDelete(scope.row)"
          > 删除 </el-button>
        </template>

      </el-table-column>
    </el-table>
    <el-pagination
      background
      :current-page="queryForm.pageNo"
      :page-size="queryForm.pageSize"
      :layout="layout"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    ></el-pagination>

    <edit ref="edit" @fetchData="fetchData"></edit>
  </div>
</template>

<script>
  import { getList, doDelete, doDeleteAll } from "@/api/tenantManagement";
  import Edit from "./components/TenantManagementEdit";
  import {isNull} from "@/utils/validate";

  export default {
    name: "TenantManagement",
    components: { Edit },
    data() {
      return {
        list: null,
        listLoading: true,
        layout: "total, sizes, prev, pager, next, jumper",
        total: 0,
        selectRows: "",
        elementLoadingText: "正在加载...",
        queryForm: {
          pageNo: 1,
          pageSize: 10,
          id_EQ:"",
          tenantName_LIKE: "",
        },
      };
    },
    created() {
      this.fetchData();
    },
    methods: {
      setSelectRows(val) {
        this.selectRows = val;
      },
      handleInsert(row) {
        this.$refs["edit"].showEdit();
      },
      handleUpdate(row) {
        if (row.id) {
          this.$refs["edit"].showEdit(row);
        }
      },
      handleDelete(row) {
        if (row.id) {
          this.$baseConfirm("你确定要删除当前项吗", null, async () => {
            const { msg } = await doDelete({ id: row.id });
            this.$baseMessage(msg, "success");
            await this.fetchData();
          });
        } else {
          if (this.selectRows.length > 0) {
            const ids = this.selectRows.map((item) => item.id).join();
            this.$baseConfirm("你确定要删除选中项吗", null, async () => {
              const { msg } = await doDeleteAll({ ids });
              this.$baseMessage(msg, "success");
              await this.fetchData();
            });
          } else {
            this.$baseMessage("未选中任何行", "error");
            return false;
          }
        }
      },
      handleSizeChange(val) {
        this.queryForm.pageSize = val;
        this.fetchData();
      },
      handleCurrentChange(val) {
        this.queryForm.pageNo = val;
        this.fetchData();
      },
      queryData() {
        this.queryForm.pageNo = 1;
        this.fetchData();
      },
      async fetchData() {
        this.listLoading = true;
        const { data } = await getList(this.queryForm);
        if(!isNull(data)){
          this.list = data.rows;
          this.total = data.total;
          setTimeout(() => {
            this.listLoading = false;
          }, 300);
        }
      },
    },
  };
</script>
