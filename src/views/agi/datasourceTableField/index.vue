<template>
  <div class="layout-padding">
    <div class="layout-padding-auto layout-padding-view">
      <el-row v-show="showSearch">
        <el-form :model="state.queryForm" ref="queryRef" :inline="true" @keyup.enter="getDataList">
          <el-form-item label="数据表" prop="tableId">
            <el-select v-model="state.queryForm.tableId" placeholder="请选择表">
              <el-option label="请选择">0</el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button icon="search" type="primary" @click="getDataList">查 询</el-button>
            <el-button icon="Refresh" @click="resetQuery">重 置</el-button>
          </el-form-item>
        </el-form>
      </el-row>
      <el-row>
        <div class="mb8" style="width: 100%">
          <el-button plain :disabled="multiple" icon="Delete" type="primary"
                     v-auth="'agi_datasourceTableField_del'" @click="handleDelete(selectObjs)">
            删 除
          </el-button>
          <right-toolbar v-model:showSearch="showSearch" :export="'agi_datasourceTableField_export'"
                         @exportExcel="exportExcel" class="ml10 mr20" style="float: right;"
                         @queryTable="getDataList"></right-toolbar>
        </div>
      </el-row>
      <el-table :data="state.dataList" v-loading="state.loading" border
                :cell-style="tableStyle.cellStyle" :header-cell-style="tableStyle.headerCellStyle"
                @selection-change="selectionChangHandle"
                @sort-change="sortChangeHandle">
        <el-table-column type="selection" width="40" align="center"/>
        <el-table-column type="index" label="#" width="50"/>
        <el-table-column prop="dsId" label="数据源" width="200"/>
        <el-table-column prop="tableId" label="数据表" width="200"/>
        <el-table-column prop="fieldName" label="字段名称" width="200"/>
        <el-table-column prop="fieldType" label="字段类型" width="200"/>
        <el-table-column prop="fieldComment" label="字段注释" width="300"/>
        <el-table-column prop="customComment" label="字段注释(自定义)" show-overflow-tooltip/>
        <el-table-column prop="weight" label="权重" width="100"/>
        <el-table-column prop="updateTime" label="更新时间" width="200"/>
      </el-table>
      <pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" v-bind="state.pagination"/>
    </div>

    <!-- 编辑、新增  -->
    <form-dialog ref="formDialogRef" @refresh="getDataList(false)"/>

    <!-- 导入excel (需要在 upms-biz/resources/file 下维护模板) -->
    <upload-excel
        ref="excelUploadRef"
        title="导入"
        url="/agi/datasourceTableField/import"
        temp-url="/admin/sys-file/local/file/datasourceTableField.xlsx"
        @refreshDataList="getDataList"
    />
  </div>
</template>

<script setup lang="ts" name="systemDatasourceTableField">
import {BasicTableProps, useTable} from "/@/hooks/table";
import {fetchList, delObjs} from "/@/api/agi/datasourceTableField";
import {useMessage, useMessageBox} from "/@/hooks/message";
import {useDict} from '/@/hooks/dict';

// 引入组件
const FormDialog = defineAsyncComponent(() => import('./form.vue'));
// 定义查询字典

// 定义变量内容
const formDialogRef = ref()
const excelUploadRef = ref();
// 搜索变量
const queryRef = ref()
const showSearch = ref(true)
// 多选变量
const selectObjs = ref([]) as any
const multiple = ref(true)

const state: BasicTableProps = reactive<BasicTableProps>({
  queryForm: {},
  pageList: fetchList
})

//  table hook
const {
  getDataList,
  currentChangeHandle,
  sizeChangeHandle,
  sortChangeHandle,
  downBlobFile,
  tableStyle
} = useTable(state)

// 清空搜索条件
const resetQuery = () => {
  // 清空搜索条件
  queryRef.value?.resetFields()
  // 清空多选
  selectObjs.value = []
  getDataList()
}

// 导出excel
const exportExcel = () => {
  downBlobFile('/agi/datasourceTableField/export', Object.assign(state.queryForm, {ids: selectObjs}), 'datasourceTableField.xlsx')
}

// 多选事件
const selectionChangHandle = (objs: { id: string }[]) => {
  selectObjs.value = objs.map(({id}) => id);
  multiple.value = !objs.length;
};

// 删除操作
const handleDelete = async (ids: string[]) => {
  try {
    await useMessageBox().confirm('此操作将永久删除');
  } catch {
    return;
  }

  try {
    await delObjs(ids);
    getDataList();
    useMessage().success('删除成功');
  } catch (err: any) {
    useMessage().error(err.msg);
  }
};
</script>