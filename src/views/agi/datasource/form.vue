<template>
  <el-dialog :title="form.id ? '编辑' : '新增'" v-model="visible"
             :close-on-click-modal="false" draggable>
    <el-form ref="dataFormRef" :model="form" :rules="dataRules" formDialogRef label-width="120px" v-loading="loading">
      <el-row :gutter="24">
        <el-col :span="12" class="mb20">
          <el-form-item label="数据源名称" prop="name">
            <el-input v-model="form.name" placeholder="请输入数据源名称"/>
          </el-form-item>
        </el-col>

        <el-col :span="12" class="mb20">
          <el-form-item label="数据库类型" prop="dsType">
            <el-select v-model="form.dsType" placeholder="请选择类型">
              <el-option :key="item.value" :label="item.label" :value="item.value"
                         v-for="item in aig_ds_type"/>
            </el-select>
          </el-form-item>
        </el-col>

        <el-col :span="24" class="mb20">
          <el-form-item label="描述" prop="description">
            <el-input type="textarea" v-model="form.description" placeholder="请输入描述"/>
          </el-form-item>
        </el-col>

        <el-col :span="12" class="mb20">
          <el-form-item label="主机" prop="host">
            <el-input v-model="form.host" placeholder="请输入主机"/>
          </el-form-item>
        </el-col>

        <el-col :span="12" class="mb20">
          <el-form-item label="端口" prop="port">
            <el-input v-model="form.port" placeholder="请输入端口"/>
          </el-form-item>
        </el-col>

        <el-col :span="12" class="mb20">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="form.username" placeholder="请输入用户名"/>
          </el-form-item>
        </el-col>

        <el-col :span="12" class="mb20">
          <el-form-item label="密码" prop="password">
            <el-input v-model="form.password" placeholder="请输入密码"/>
          </el-form-item>
        </el-col>

        <el-col :span="12" class="mb20">
          <el-form-item label="数据库名称" prop="dsName">
            <el-input v-model="form.dsName" placeholder="请输入数据库名称"/>
          </el-form-item>
        </el-col>

      </el-row>
    </el-form>
    <template #footer>
        <span class="dialog-footer">
          <el-button @click="visible = false">取 消</el-button>
          <el-button type="primary" @click="onSubmit" :disabled="loading">确 认</el-button>
        </span>
    </template>
  </el-dialog>
</template>

<script setup lang="ts" name="DatasourceDialog">
import {useDict} from '/@/hooks/dict';
import {useMessage} from "/@/hooks/message";
import {getObj, addObj, putObj, validateExist} from '/@/api/agi/datasource'
import {rule} from '/@/utils/validate';

const emit = defineEmits(['refresh']);

// 定义变量内容
const dataFormRef = ref();
const visible = ref(false)
const loading = ref(false)
// 定义字典
const {aig_ds_type} = useDict('aig_ds_type')
// 提交表单数据
const form = reactive({
  id: '',
  name: '',
  url: '',
  username: '',
  password: '',
  dsType: '',
  confType: '',
  dsName: '',
  instance: '',
  port: '',
  host: '',
  description: ''
});

// 定义校验规则
const dataRules = ref({
  name: [{required: true, message: '数据源名称不能为空', trigger: 'blur'}],
  url: [{required: true, message: 'jdbcurl不能为空', trigger: 'blur'}],
  username: [{required: true, message: '用户名不能为空', trigger: 'blur'}],
  password: [{required: true, message: '密码(加密)不能为空', trigger: 'blur'}],
  dsType: [{required: true, message: '数据库类型不能为空', trigger: 'blur'}],
  port: [{required: true, message: '端口不能为空', trigger: 'blur'}],
  host: [{required: true, message: '主机不能为空', trigger: 'blur'}],
})

// 打开弹窗
const openDialog = (id: string) => {
  visible.value = true
  form.id = ''

  // 重置表单数据
  nextTick(() => {
    dataFormRef.value?.resetFields();
  });

  // 获取datasource信息
  if (id) {
    form.id = id
    getDatasourceData(id)
  }
};

// 提交
const onSubmit = async () => {
  const valid = await dataFormRef.value.validate().catch(() => {
  });
  if (!valid) return false;

  try {
    loading.value = true;
    form.id ? await putObj(form) : await addObj(form);
    useMessage().success(form.id ? '修改成功' : '添加成功');
    visible.value = false;
    emit('refresh');
  } catch (err: any) {
    useMessage().error(err.msg);
  } finally {
    loading.value = false;
  }
};


// 初始化表单数据
const getDatasourceData = (id: string) => {
  // 获取数据
  loading.value = true
  getObj({id: id}).then((res: any) => {
    Object.assign(form, res.data[0])
  }).finally(() => {
    loading.value = false
  })
};

// 暴露变量
defineExpose({
  openDialog
});
</script>