<template>
  <div class="admin-container">
    <Button type="primary" icon="md-add" @click="addModal = true">点击添加一条记录</Button>
    <Input
      v-model="searchInput"
      prefix="ios-contact"
      placeholder="请输入名字"
      style="width: auto; margin-left: 20px"
    />
    <Button type="primary" icon="ios-search" style="margin-left: 10px;" @click="handleSearch">搜索</Button>
    <Button type="warning" icon="md-refresh" style="margin-left: 10px;" @click="handleReset">重置</Button>

    <!-- 主体显示表格 -->
    <Table :columns="columns" :data="transStudentData" style="margin-top: 10px;">
      <!-- 插槽，显示每行的编辑功能 -->
      <template slot-scope="{ row }" slot="action">
        <Button type="primary" size="default" style="margin-right: 5px" @click="teacherEdit(row)">编辑</Button>
        <Button type="error" size="default" @click="teacherDelete(row)">删除</Button>
      </template>
    </Table>

    <!-- 添加modal -->
    <Modal v-model="addModal" title="添加教师信息" @on-visible-change="handleOpenAdd" @on-ok="handleAdd">
      <Form :model="addForm" :label-width="80" :rules="addRule" ref="addForm">
        <FormItem label="姓名" prop="name">
          <i-input v-model="addForm.name" style="width: 150px;"></i-input>
          <!-- <span style="margin-left: 10px;">长度2~10位</span> -->
        </FormItem>

        <FormItem label="性别" prop="sex">
          <Select v-model="addForm.sex" style="width: 150px;">
            <Option value="m">男</Option>
            <Option value="f">女</Option>
          </Select>
        </FormItem>

        <FormItem label="密码" prop="password">
          <i-input v-model="addForm.password" style="width: 150px;"></i-input>
          <!-- <span style="margin-left: 10px;">长度5~10位</span> -->
        </FormItem>

        <FormItem label="负责班级" prop="class_name">
          <i-input v-model="addForm.class_name" style="width: 150px;"></i-input>
        </FormItem>

        <!-- todo 这里增加下拉框让用户选择课程，然后传入id-->
        <FormItem label="课程">
          <Select v-model="addForm.course_id" style="width: 150px;">
            <Option :value="course.id" :label="course.name" v-for="course in courses" :key="course.id"></Option>
          </Select>
        </FormItem>
      </Form>
    </Modal>

    <!-- 编辑modal -->
    <Modal
      v-model="editModal"
      title="编辑教师信息"
      @on-visible-change="handleOpenEdit"
      @on-ok="handleEdit"
    >
      <Form :model="editForm" :label-width="80" :rules="editRule" ref="editForm">
        <FormItem label="姓名" prop="name">
          <i-input v-model="editForm.name" style="width: 150px;"></i-input>
          <!-- <span style="margin-left: 10px;">长度2~10位</span> -->
        </FormItem>

        <FormItem label="性别" prop="sex">
          <Select v-model="editForm.sex" style="width: 150px;">
            <Option value="m">男</Option>
            <Option value="f">女</Option>
          </Select>
        </FormItem>

        <FormItem label="新密码" prop="password">
          <i-input v-model="editForm.password" style="width: 150px;" placeholder=""></i-input>
          <!-- <span style="margin-left: 10px;">长度5~10位</span> -->
        </FormItem>

        <FormItem label="负责班级" prop="class_name">
          <i-input v-model="editForm.class_name" style="width: 150px;"></i-input>
        </FormItem>
      </Form>
    </Modal>
  </div>
</template>

<script>
import adminModel from "@/api/admin.js";
import { deepClone } from '@/libs/tools.js';

export default {
  data () {
    const validateName = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入用户名'));
      } else if (value.length < 2 || value.length > 10) {
        callback(new Error('请输入长度2~10的名字！'));
      } else {
        callback();
      }
    };

    const validatePass = (rule, value, callback) => {
      if (value === '' || value === ' ') {
        callback(new Error('请输入密码'));
      } else if (value.length < 5 || value.length > 10) {
        callback(new Error('请输入长度5~10的密码！'));
      } else {
        callback();
      }
    };
    return {
      editModal: false,
      addModal: false,
      //  编辑表格
      editForm: {
        id: -1,
        name: '',
        password: '',
        sex: 'f',
        class_name: ''
      },
      //  添加表格
      addForm: {
        name: '',
        password: '',
        sex: '',
        class_name: '',
        course_id: 0
      },
      //  课程临时
      courses: [],
      //  编辑弹窗的验证
      editRule: {
        name: [
          { validator: validateName, trigger: 'blur' }
        ]
      },
      addRule: {
        name: [
          { validator: validateName, required: true, trigger: 'blur' }
        ],
        sex: [
          { required: true, message: '请选择性别', trigger: 'blur' }
        ],
        password: [
          { validator: validatePass, required: true, trigger: 'blur' }
        ]
      },
      //  搜索框的内容
      searchInput: ''
    }
  },
  props: {
    columns: {
      type: Array,
      required: true
    },
    tableData: {
      type: Array,
      required: true
    }
  },
  methods: {
    //  点击删除一条信息
    teacherDelete (row) {
      this.$Modal.confirm({
        title: '警告',
        content: '<p>确认要删除该用户的信息吗？</p>',
        onOk: () => {
          adminModel.deleteUser({ role: 'teacher', id: row.id }).then((res) => {
            if (res.retcode === 0) {
              console.log(res);

              this.$Message.success('删除成功!');

              // 刷新数据
              this.$emit('upSuccess', 'teacher');
            } else {
              this.$Message.error({ content: '删除失败，请稍后重试', duration: 4 });
            }
          });
        }
      });

      console.log(row);
    },
    // 点击编辑一条信息
    teacherEdit (row) {
      // 初始化数据
      // todo
      this.editForm.id = row.id;
      this.editForm.name = row.name;
      if (row.sex === '男') {
        this.editForm.sex = 'm';
      } else {
        this.editForm.sex = 'f';
      }
      this.editForm.class_name = row.class_name;

      // 打开弹窗
      this.editModal = true;
    },
    //  点开编辑的初始化
    handleOpenEdit (status) {
      if (status) {
        //  打开前的初始化

      } else {
        //  关闭的初始化
        //  关闭的初始化
        const refName = 'editForm';
        this.$refs[refName].resetFields();
      }
    },
    //  点开添加的初始化
    handleOpenAdd () {
      this.addForm.name = '';
      this.addForm.password = '';
      this.addForm.sex = '';
      this.addForm.class_name = '';

      adminModel.getCourseData().then((res) => {
        if (res.retcode === 0) {
          console.log(res);
          console.log('这里是课程的数组：');
          console.log(res.data);

          this.courses = res.data;
        }
      });
    },
    //  点击弹窗的确认编辑
    handleEdit () {
      this.loading1 = false;
      const refName = 'editForm';
      this.$refs[refName].validate((valid) => {
        if (valid) {
          console.log('confirm');

          const pForm = deepClone(this.editForm);

          console.table(pForm);

          Object.keys(pForm) // 这里取到key
            .forEach((key) => { // 这里取到value
              console.log(pForm[key]);
              if (!pForm[key]) {
                delete pForm[key];
              }
            });

          adminModel.editInfo({ role: 'teacher', editForm: pForm }).then((res) => {
            if (res.retcode === 0) {
              console.log(res);

              this.$Message.success('修改成功!');
              this.editModal = false;
              this.$emit('upSuccess', 'teacher');
            } else {
              this.$Message.error({ content: '修改失败，请稍后重试', duration: 4 });
            }
          })
        } else {
          this.$Message.error({ content: '请检查后重新提交!', duration: 4 });
        }
      });
    },
    //  点击弹窗的确认添加
    handleAdd () {
      const refName = 'addForm';
      this.$refs[refName].validate((valid) => {
        if (valid) {
          const pForm = deepClone(this.addForm);
          // alert('pass');

          adminModel.addUser({ role: 'teacher', postData: pForm }).then((res) => {
            if (res.retcode === 0) {
              console.log(res);

              this.$Message.success('添加成功!');
              this.eaddModal = false;

              // 刷新数据
              this.$emit('upSuccess', 'teacher');
            } else {
              this.$Message.error({ content: '添加失败，请稍后重试', duration: 4 });
            }
          });
        } else {
          this.$Message.error({ content: '请检查完整后重新提交!', duration: 4 });
        }
      });
    },
    // 搜索名字
    handleSearch () {
      if (!this.searchInput) {
        this.$emit('upSuccess', 'teacher');
      } else {
        adminModel.searchUser({ role: 'teacher', name: this.searchInput }).then((res) => {
          if (res.retcode === 0) {
            console.log(res);

            this.$emit('searchUser', res.data, 'teacher');
          } else {
            this.$Message.error({ content: '未找到该用户，请核对后重试', duration: 4 });
          }
        })
      }
    },
    handleReset () {
      this.searchInput = '';
      this.$emit('upSuccess', 'teacher');
    }
  },
  computed: {
    transStudentData () {
      const data = [];
      if (this.tableData[0]) {
        console.log(this.tableData);
        this.tableData.forEach((elem, idx) => {
          data.push(elem);

          // 性别过滤
          if (elem.sex === 'f') {
            data[idx].sex = '女';
          } else {
            data[idx].sex = '男';
          }
        });
      }

      return data;
    }
  }
}
</script>

<style scoped>
</style>
