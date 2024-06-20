<template>
  <el-breadcrumb :separator-icon="ArrowRight">
    <el-breadcrumb-item :to="{ path: '/dashboard' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>队伍管理</el-breadcrumb-item>
  </el-breadcrumb>
  <el-card>
    <!-- 搜索-->
    <el-row :gutter="20">
      <el-col :span="6">
        <el-input placeholder="请输入队伍名称" v-model="searchName" clearable @change="researchUser">
          <template #append>
            <el-button :icon="Search" @click="searchTeamByName"/>
          </template>
        </el-input>
      </el-col>
      <el-col :span="2">
        <el-button type="primary" @click="addTeamFormVisible = true">添加队伍</el-button>
      </el-col>
    </el-row>

    <!--表格信息-->
    <el-table :data="tableData" table-layout="fixed" border stripe>
      <el-table-column label="封面" width="100px" align="center" slot-scope="scope">
        <template #default="scope">
          <el-avatar shape="square" size="large"
                     :src="scope.row.coverImage"
                     style="width: 60px;height: 60px"
          />
        </template>
      </el-table-column>
      <el-table-column prop="id" align="center" width="160px" label="队伍id"/>
      <el-table-column prop="name" align="center" width="160px" label="队伍名称"/>
      <el-table-column prop="description" align="center" width="260px" label="队伍简介"/>
      <el-table-column prop="expireTime" label="过期时间" align="center" width="160px"/>
      <el-table-column prop="maxNum" label="最多人数" align="center" width="100px"/>
      <el-table-column prop="status" align="center" width="80px" label="状态"/>
      <el-table-column prop="userId" label="创建者Id" width="100px" align="center"/>
      <el-table-column prop="updateTime" label="更新时间" width="160px" align="center"/>
      <el-table-column fixed="right" label="操作" align="center" slot-scope="scope">
        <template #default="scope">
          <el-button type="primary" :icon="Edit" circle @click="UpdateTeamInfo(scope.row.id)"/>
          <el-button type="danger" :icon="Delete" circle @click="TeamDelete(scope.row.id)"/>
        </template>
      </el-table-column>
    </el-table>

    <!--分页-->
    <el-pagination
        v-model:current-page="currentPage"
        :page-size="pageSize"
        :background="true"
        layout="total, prev, pager, next"
        :total="total"
        @current-change="getTeamInfoList"
    />
    <!-- 添加队伍-->
    <el-dialog v-model="addTeamFormVisible" title="添加新队伍" width="30%" center>
      <el-form :model="addTeamForm" :rules="addTeamFormRoles">
        <el-form-item label="队伍昵称：" :label-width="formLabelWidth" prop="name">
          <el-input v-model="addTeamForm.name" autocomplete="off" placeholder="请输入队伍名称"/>
        </el-form-item>
        <el-form-item label="队伍简介：" :label-width="formLabelWidth" prop="description">
          <el-input v-model="addTeamForm.description" autocomplete="off" placeholder="请输入队伍简介"/>
        </el-form-item>
        <el-form-item label="过期时间：" :label-width="formLabelWidth">
          <el-date-picker
              v-model="addTeamForm.expireTime"
              type="datetime"
              placeholder="请选择过期时间，若不选择则永久有效"
              format="YYYY-MM-DD HH:mm:ss"
              value-format="YYYY-MM-DD HH:mm:ss"
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item label="限定人数：" :label-width="formLabelWidth">
          <el-input v-model="addTeamForm.maxNum" autocomplete="off" placeholder="请选择队伍限定人数"/>
        </el-form-item>
        <el-form-item label="队伍状态：" :label-width="formLabelWidth">
          <el-select
              v-model="addTeamForm.status"
              placeholder="状态"
              style="width: 240px"
          >
            <el-option
                v-for="item in statusValueOption"
                :key="item.value"
                :label="item.label"
                :value="item.value"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <template #footer>
      <span class="dialog-footer">
        <el-button @click="addTeamFormVisible = false">取消</el-button>
        <el-button type="primary" @click="handleAddTeam">
          提交
        </el-button>
      </span>
      </template>
    </el-dialog>

    <!-- 更新队伍-->
    <el-dialog v-model="updateTeamFormVisible" title="更新队伍" width="25%" center>
      <el-form :model="updateTeamForm">
        <el-form-item label="队伍图片：" :label-width="formLabelWidth">
          <el-upload
              ref="uploadRef"
              :http-request="handleUpload"
              :before-upload="beforeAvatarUpload"
          >
            <el-avatar
                :src="updateTeamForm.coverImage"
                shape="square" size="large" style="width: 200px;height: 100px" fit="fill">
              <template #placeholder>
                <div class="image-slot">
                  <el-icon>
                    <icon-picture/>
                  </el-icon>
                </div>
              </template>
            </el-avatar>
          </el-upload>
        </el-form-item>
        <el-form-item label="队伍ID" :label-width="formLabelWidth">
          <el-input v-model="updateTeamForm.id" clearable autocomplete="off" disabled/>
        </el-form-item>
        <el-form-item label="队伍名称" :label-width="formLabelWidth">
          <el-input v-model="updateTeamForm.name" clearable autocomplete="off"/>
        </el-form-item>
        <el-form-item label="队伍简介" :label-width="formLabelWidth">
          <el-input v-model="updateTeamForm.description" clearable autocomplete="off"/>
        </el-form-item>
        <el-form-item label="最大人数" :label-width="formLabelWidth">
          <el-input v-model="updateTeamForm.maxNum" clearable autocomplete="off"/>
        </el-form-item>
        <el-form-item label="状态" :label-width="formLabelWidth">
          <el-select
              v-model="updateTeamForm.status"
              placeholder="状态"
              style="width: 240px"
          >
            <el-option
                v-for="item in statusValueOption"
                :key="item.value"
                :label="item.label"
                :value="item.value"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="密码" :label-width="formLabelWidth">
          <el-input v-model="updateTeamForm.password" placeholder="请选择了加密后，才输入密码"
                    autocomplete="off" clearable :disabled="updateTeamForm.status === 0 ?? updateTeamForm.status ===1"/>
        </el-form-item>
        <el-form-item label="过期时间" :label-width="formLabelWidth">
          <el-date-picker
              v-model="updateTeamForm.expireTime"
              type="datetime"
              placeholder="请选择时间"
              format="YYYY-MM-DD HH:mm:ss"
              value-format="YYYY-MM-DD HH:mm:ss"
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item label="创建时间" :label-width="formLabelWidth">
          <div class="demo-date-picker">
            <el-date-picker
                v-model="updateTeamForm.createTime"
                type="datetime"
                placeholder="请选择时间"
                format="YYYY-MM-DD HH:mm:ss"
                value-format="YYYY-MM-DD HH:mm:ss"
                disabled
            >
            </el-date-picker>
          </div>
        </el-form-item>
        <el-form-item label="更新时间" :label-width="formLabelWidth">
          <div class="demo-date-picker">
            <el-date-picker
                v-model="updateTeamForm.updateTime"
                type="datetime"
                placeholder="请选择时间"
                format="YYYY-MM-DD HH:mm:ss"
                value-format="YYYY-MM-DD HH:mm:ss"
                disabled
            >
            </el-date-picker>
          </div>
        </el-form-item>
      </el-form>
      <template #footer>
      <span class="dialog-footer">
        <el-button @click="updateTeamFormVisible = false">取消</el-button>
        <el-button type="primary" @click="handleUpdateTeam">
          提交
        </el-button>
      </span>
      </template>
    </el-dialog>

  </el-card>
</template>

<script setup lang="ts">
import {onMounted, ref, reactive} from "vue";
import myAxios from "../../plugins/my-axios.ts";
import {ArrowRight, Delete, Edit, Remove, Search} from "@element-plus/icons-vue";
import {FormRules, UploadInstance, UploadProps} from "element-plus";
import {ElMessage, ElMessageBox} from 'element-plus'

const currentPage = ref(1)
const pageSize = ref(0)
const total = ref(0)
const tableData = ref([])
const searchName = ref('')
const addTeamFormVisible = ref(false)
const formLabelWidth = '100px'
const updateTeamFormVisible = ref(false)
const teamInfoFormVisible = ref(false)

const statusValueOption = [
  {
    value: '0',
    label: '公开(0)',
  },
  {
    value: '1',
    label: '私有(1)',
  },
  {
    value: '2',
    label: '加密(2)',
  }
]

onMounted(async () => {
  await getTeamInfoList()
})

const researchUser = () => {
  getTeamInfoList();
}
const getTeamInfoList = async () => {
  tableData.value = []
  currentPage.value = 10;
  const res = await myAxios.get("/team/admin/list/page")
  if (res?.data.code === 0) {
    res.data.data.records.forEach(team => {
      if (team.status == 0) {
        team.status = '公开'
      } else if (team.status == 1) {
        team.status = '私有'
      } else {
        team.status = '加密'
      }
      tableData.value.push(team)
    })
    currentPage.value = res.data.data.current
    pageSize.value = res.data.data.size
    total.value = res.data.data.total
  } else {
    ElMessage.error(res.data.message)
  }
}

const searchTeamByName = async () => {
  tableData.value = []
  currentPage.value = 1
  let res = await myAxios.get("/team/admin/search?name=" + searchName.value + "&currentPage=" + currentPage.value);
  console.log("res", res)
  if (res?.data.code === 0) {
    res.data.data.records.forEach(team => {
      if (team.status == 0) {
        team.status = '公开'
      } else if (team.status == 1) {
        team.status = '私有'
      } else {
        team.status = '加密'
      }
      tableData.value.push(team)
    })
    currentPage.value = res.data.data.current
    pageSize.value = res.data.data.size
    total.value = res.data.data.total
  } else {
    ElMessage.error(res.data.description)
  }
}

const handleAddTeam = async () => {
  let res = await myAxios.post("/team/add", addTeamForm);
  if (res.data.code === 0) {
    ElMessage({
      message: '队伍添加成功,新队伍ID为' + res.data.data,
      type: 'success',
    })
    addTeamFormVisible.value = false
    await getTeamInfoList()
  } else {
    ElMessage.error(res.data.description)
  }
}

// 通过 ID 搜索队伍信息
const UpdateTeamInfo = async (id: string) => {
  let res = await myAxios.get("/team/" + id);
  if (res.data.code === 0) {
    updateTeamFormVisible.value = true
    updateTeamForm.value = res.data.data
  } else {
    ElMessage.error(res.data.description)
  }
}

const searchedTeam = ref({
  userAccount: '',
  username: '',
  avatarUrl: '',
  id: ''
})
const DeleteTeam = async (id: string) => {
  let res = await myAxios.get("/team/delete/" + id);
  if (res.data.code === 0) {
    teamInfoFormVisible.value = true
    searchedTeam.value = res.data.data
  } else {
    ElMessage.error(res.data.description)
  }
}
// 更新队伍信息
const handleUpdateTeam = async () => {
  const userData = {
    id: updateTeamForm.value.id,
    name: updateTeamForm.value.name,
    description: updateTeamForm.value.description,
    maxNum: updateTeamForm.value.maxNum,
    expireTime: updateTeamForm.value.expireTime,
    status: updateTeamForm.value.status,
    password: updateTeamForm.value.password,
    createTime: updateTeamForm.value.createTime,
    updateTime: updateTeamForm.value.updateTime,
  }
  let res = await myAxios.post("/team/admin/update", userData);
  if (res.data.code === 0) {
    updateTeamFormVisible.value = false
    ElMessage({
      message: '队伍更新成功',
      type: 'success',
    })
    await getTeamInfoList()
  } else {
    ElMessage.error("更新失败" + res.data.description)
  }
}

const updateTeamForm = ref({
  id: '',
  description: '',
  coverImage: '',
  name: '',
  maxNum: '',
  expireTime: '',
  status: 0,
  password: '',
  createTime: '',
  updateTime: ''
})

const addTeamForm = reactive({
  name: '',
  description: '',
  expireTime: '',
  maxNum: '',
  status: 0,
})

interface RuleForm {
  name: string,
  description: string
}

const addTeamFormRoles = reactive<FormRules<RuleForm>>({
  name: [
    {required: true, message: '请输入队伍名', trigger: 'blur'}
  ],
  description: [
    {required: true, message: '请输入密码', trigger: 'blur'}
  ]
})

// 队伍删除（解散）
const TeamDelete = (id: any) => {
  ElMessageBox.confirm(
      '此操作无法撤销',
      '确认删除队伍',
      {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning',
      }
  )
      .then(async () => {
        let res = await myAxios.post("/team/admin/delete/" + id);
        if (res.data.code === 0) {
          ElMessage({
            type: 'success',
            message: '删除成功',
          })
          await getTeamInfoList()
        } else {
          ElMessage.error(res.data.description)
        }
      })
}

// 文件上传
const uploadRef = ref<UploadInstance>();

// 上传图片前校验
const beforeAvatarUpload: UploadProps['beforeUpload'] = (rawFile) => {
  if (rawFile.type !== 'image/jpg' && rawFile.type !== 'image/png' && rawFile.type !== 'image/jpeg') {
    ElMessage.error('仅允许上传 JPG/PNG/jpeg格式的图片')
    return false
  } else if (rawFile.size / 1024 / 1024 > 2) {
    ElMessage.error('图片不能超过2MB！')
    return false
  }
  return true
}

const handleUpload = async (param: any) => {
  let fileObj = param.file; // 相当于input里取得的files
  let fd = new FormData();// FormData 对象
  fd.append("file", fileObj);// 文件对象
  let res = await myAxios.post("/file/upload/team/" + updateTeamForm.value.id, fd, {
    headers: {
      "Content-Type": "multipart/form-data",
    },
  });
  if (res.data.code === 0) {
    ElMessage({
      message: "更新成功",
      type: "success",
    });
    updateTeamFormVisible.value = false;
    await getTeamInfoList();
  } else {
    ElMessage.error(res.data.description);
  }
};
</script>

<style scoped>
.dialog-footer button:first-child {
  margin-right: 10px;
}

.image {
  width: 100%;
  display: block;
}
</style>
