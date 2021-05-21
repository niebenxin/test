<template>
  <div v-cloak>
    <el-form :inline="true" class="demo-form-inline">
      <el-form-item label="">
        <el-input v-model="name" placeholder="名称    |    性别    |    班级"></el-input>
      </el-form-item>
      <el-form-item label="">
        <el-select placeholder="班级" v-model="cname">
          <el-option v-for="c in cla"
                     :key="c.classno"
                     :label="c.classname"
                     :value="c.classno">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" @click="query()">搜索</el-button>
        <el-button type="info" @click="rest('name')">重置</el-button>
      </el-form-item>
    </el-form>

    <div style="margin-top: 20px;margin-left: -1169px;">
      <el-button type="success" plain @click="dialogFormVisible = true">Add</el-button>
      <el-button type="danger" plain @click="dels()"><i class="el-icon-delete-solid"></i> 批量删除</el-button>
      <el-button type="info" plain @click="toggleSelection()">取消选择</el-button>
    </div>

    <el-table
      ref="multipleTable"
      :data="tableData.filter(data => !search || data.name.toLowerCase().includes(search.toLowerCase()))"
      style="width: 100%"
      :row-class-name="tableRowClassName"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55">
      </el-table-column>
      <el-table-column
        type="index"
        label="ID"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="姓名"
        width="180">
      </el-table-column>
      <el-table-column
        :formatter="sexformatter"
        prop="sex"
        label="性别"
        width="180">
      </el-table-column>
      <el-table-column
        prop="mgr"
        label="班主任"
        width="180">
      </el-table-column>
      <el-table-column
        prop="classes.classname"
        label="班级名">
      </el-table-column>
      <el-table-column
        prop="bdate"
        :formatter="timeFormatter"
        label="入学时间">
      </el-table-column>
      <el-table-column label="图片" prop="image">
        <template slot-scope="scope">
          <el-popover placement="right" title trigger="hover">
            <img :src="imgUrl+scope.row.img" style="max-height: 200px;max-width: 300px"/>
            <img slot="reference" :src="imgUrl+scope.row.img" :alt="imgUrl+scope.row.img"
                 style="max-height: 50px;max-width: 130px"/>
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column>
        <template slot="header" slot-scope="scope">
          <el-input
            v-model="search"
            size="mini"
            placeholder="输入关键字搜索"/>
        </template>
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            @click="update(scope.row)"><i class="el-icon-edit"></i> Edit
          </el-button>
          <el-button
            size="mini"
            type="text"
            @click="del(scope.row)"><i class="el-icon-delete"></i> Delete
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="block">
      <el-pagination
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[3, 5, 7, 10]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </div>

    <!--添加-->
    <el-dialog title="Add" :visible.sync="dialogFormVisible" :before-close="handleClose">
      <el-form :model="form">
        <el-form-item label="学生姓名" :label-width="formLabelWidth">
          <el-input v-model="form.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="性别" :label-width="formLabelWidth">
          <el-radio-group v-model="form.sex" size="medium">
            <el-radio border label="男" value="M"></el-radio>
            <el-radio border label="女" value="F"></el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="班主任名称" :label-width="formLabelWidth">
          <el-input v-model="form.mgr" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="班级名称" :label-width="formLabelWidth">
          <el-select v-model="form.classno" placeholder="请选择班级">
            <el-option v-for="c in cla"
                       :key="c.classno"
                       :label="c.classname"
                       :value="c.classno">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="入学时间" style="margin-left: 55px">
          <el-col :span="6">
            <el-date-picker type="date" placeholder="选择日期" v-model="date1" style="width: 100%;"></el-date-picker>
          </el-col>
          <el-col class="line" :span="2">-</el-col>
          <el-col :span="6">
            <el-time-picker placeholder="选择时间" v-model="form.bdate" style="width: 100%;"></el-time-picker>
          </el-col>
        </el-form-item>
        <el-upload
          class="upload-demo"
          drag
          :on-success="handleSuccess"
          :on-error="handleError"
          action="http://localhost:8080/student/uploadVue"
          multiple>
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
          <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
        </el-upload>
        <div class="demo-image__placeholder" style="position: absolute;margin-left: 411px;margin-top: -199px;">
          <div class="block">
            <el-image :src="imgUrl+this.form.img" style="width: 93%;height: 100%;">
              <div slot="error" class="image-slot">
                <i class="el-icon-picture-outline"></i>
              </div>
            </el-image>
          </div>
        </div>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="save()">添加学生</el-button>
        <el-button @click="resetForm('form')">重置</el-button>
      </div>
    </el-dialog>

    <!--修改-->
    <el-dialog title="Upd" :visible.sync="dialogFormVisibleUpd">
      <el-form :model="ruleForm" status-icon  ref="ruleForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="学生姓名" :label-width="formLabelWidth">
          <el-input v-model="ruleForm.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="性别" :label-width="formLabelWidth">
          <el-radio-group v-model="ruleForm.sex" size="medium">
            <el-radio border label="男" value="M"></el-radio>
            <el-radio border label="女" value="F"></el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="班主任名称" :label-width="formLabelWidth">
          <el-input v-model="ruleForm.mgr" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="班级名称" :label-width="formLabelWidth">
          <el-select v-model="ruleForm.classno" placeholder="请选择班级">
            <el-option v-for="c in cla"
                       :key="c.classno"
                       :label="c.classname"
                       :value="c.classno">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="入学时间" style="margin-left: 55px">
          <el-col :span="6">
            <el-date-picker type="date" placeholder="选择日期" v-model="ruleForm.bdate" style="width: 100%;"></el-date-picker>
          </el-col>
          <el-col class="line" :span="2">-</el-col>
          <el-col :span="6">
            <el-time-picker placeholder="选择时间" v-model="ruleForm.bdate" style="width: 100%;"></el-time-picker>
          </el-col>
        </el-form-item>
        <el-upload
          class="upload-demo"
          drag
          :on-success="handleSuccess1"
          :on-error="handleError1"
          action="http://localhost:8080/student/uploadVue"
          multiple>
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">将文件拖到此处，或<em>点击修改</em></div>
          <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
        </el-upload>
        <div  class="demo-image__placeholder" style="position: absolute;margin-left: 411px;margin-top: -199px;">
          <div class="block">
            <el-image :src="imgUrl+this.ruleForm.img" style="width: 93%;height: 100%;">
              <div slot="error" class="image-slot">
                <i class="el-icon-picture-outline"></i>
              </div>
            </el-image>
          </div>
        </div>
        <el-form-item>
          <el-button type="primary" @click="submitForm('ruleForm')">修改</el-button>
          <el-button @click="resetForm('ruleForm')">重置</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<style>
  .el-table .warning-row {
    background: oldlace;
  }

  .el-table .success-row {
    background: #f0f9eb;
  }

  [v-cloak] {
    display: none;
  }

  .el-col-6 {
    width: 29%;
    margin-left: 53px;
  }

  .upload-demo {
    margin-left: -258px;
  }
  .el-upload-list__item:first-child {
    margin-top: 10px;
    width: 40%;
    margin-left: 288px;
  }
</style>

<script>
  import axios from 'axios';

  export default {
    methods: {
      tableRowClassName({row, rowIndex}) {
        if (rowIndex === 1) {
          return 'warning-row';
        } else if (rowIndex === 3) {
          return 'success-row';
        } else if (rowIndex % 2 != 0) {
          return 'success-row';
        }
        return '';
      },
      update: function (value) {
        this.dialogFormVisibleUpd=true;
        axios.get("http://localhost:8080/student/queryById/" + value.id)
          .then((response) => {
            this.ruleForm=response.data;
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      submitForm(){
        axios.get("http://localhost:8080/student/updates",this.ruleForm)
          .then((response) => {
            if(response.data){
              this.dialogFormVisibleUpd=false;
              axios.get('http://localhost:8080/student/queryAllVue?page=' + this.currentPage + '&limit=' + this.pageSize)
                .then((response) => {
                  this.tableData = response.data.vueData;
                  this.total = response.data.vueTotal;
                  this.currentPage = response.data.vueCurrentPage;
                  this.pageSize = response.data.vuePageSize;
                })
                .catch(function (error) {
                  console.log(error);
                });
              this.$message({
                message: '修改成功',
                type: 'success'
              });
            }else {
              this.$message.error('修改失败');
            }
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      del: function (value) {
        axios.get("http://localhost:8080/student/delete/" + value.id)
          .then((response) => {
            if (response.data == '1') {
              this.$message({
                message: '删除成功',
                type: 'success'
              });
              axios.get('http://localhost:8080/student/queryAllVue?page=' + this.currentPage + '&limit=' + this.pageSize)
                .then((response) => {
                  this.tableData = response.data.vueData;
                  this.total = response.data.vueTotal;
                  this.currentPage = response.data.vueCurrentPage;
                  this.pageSize = response.data.vuePageSize;
                })
                .catch(function (error) {
                  console.log(error);
                });
            } else {
              this.$message.error('删除失败');
            }
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      query: function () {
        axios.get('http://localhost:8080/student/queryAllVue?page=1&limit=' + this.pageSize + '&name=' + this.name)
          .then((response) => {
            this.tableData = response.data.vueData;
            this.total = response.data.vueTotal;
            this.currentPage = response.data.vueCurrentPage;
            this.pageSize = response.data.vuePageSize;
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      sexformatter: function (row) {
        if (row.sex === 'M') {
          return '男';
        } else {
          return '女';
        }
      },
      handleSizeChange: function (val) {
        axios.get('http://localhost:8080/student/queryAllVue?page=' + this.currentPage + '&limit=' + val)
          .then((response) => {
            this.tableData = response.data.vueData;
            this.total = response.data.vueTotal;
            this.currentPage = response.data.vueCurrentPage;
            this.pageSize = response.data.vuePageSize;
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      handleCurrentChange: function (val) {
        axios.get('http://localhost:8080/student/queryAllVue?page=' + val + '&limit=' + this.pageSize)
          .then((response) => {
            this.tableData = response.data.vueData;
            this.total = response.data.vueTotal;
            this.currentPage = response.data.vueCurrentPage;
            this.pageSize = response.data.vuePageSize;
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      handleSelectionChange(val) {
        this.ids = '';
        for (var i = 0; i < val.length; i++) {
          if (i >= val.length - 1) {
            this.ids = this.ids + val[i].id;
          } else {
            this.ids = this.ids + val[i].id + ",";
          }
        }
      },
      dels: function () {
        axios.get('http://localhost:8080/student/deletesVue?ids=' + this.ids)
          .then((response) => {
            if (response.data) {
              this.$message({
                message: '删除成功',
                type: 'success'
              });
              axios.get('http://localhost:8080/student/queryAllVue?page=' + this.currentPage + '&limit=' + this.pageSize)
                .then((response) => {
                  this.tableData = response.data.vueData;
                  this.total = response.data.vueTotal;
                  this.currentPage = response.data.vueCurrentPage;
                  this.pageSize = response.data.vuePageSize;
                })
                .catch(function (error) {
                  console.log(error);
                });
            } else {
              this.$message.error('删除失败');
            }
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      toggleSelection(rows) {
        if (rows) {
          rows.forEach(row => {
            this.$refs.multipleTable.toggleRowSelection(row);
          });
        } else {
          this.$refs.multipleTable.clearSelection();
        }
      },
      rest: function (name) {
        this.name = '';
        axios.get('http://localhost:8080/student/queryAllVue?page=' + this.currentPage + '&limit=' + this.pageSize)
          .then((response) => {
            this.tableData = response.data.vueData;
            this.total = response.data.vueTotal;
            this.currentPage = response.data.vueCurrentPage;
            this.pageSize = response.data.vuePageSize;
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      handleClose(done) {
        this.$confirm('确认关闭？')
          .then(_ => {
            done();
          })
          .catch(_ => {
          });
      },
      resetForm(formName) {
        this.$refs[formName].resetFields();
      },
      save() {
        alert(this.form.bdate)
        axios.post('http://localhost:8080/student/saves',this.form)
          .then((response) => {
            if(response.data){
              this.dialogFormVisible=false;
              axios.get('http://localhost:8080/student/queryAllVue?page=' + this.currentPage + '&limit=' + this.pageSize)
                .then((response) => {
                  this.tableData = response.data.vueData;
                  this.total = response.data.vueTotal;
                  this.currentPage = response.data.vueCurrentPage;
                  this.pageSize = response.data.vuePageSize;
                })
                .catch(function (error) {
                  console.log(error);
                });
              this.$message({
                message: '添加成功',
                type: 'success'
              });
            }else {
              this.$message.error('添加失败');
            }
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      /*上传成功的钩子*/
      handleSuccess(response, file, fileList) {
        this.form.img = response.data[0].img;
      },
      /*上传失败的钩子*/
      handleError(response, file, fileList) {
        alert("上传失败");
      },
      /*修改上传成功的钩子*/
      handleSuccess1(response, file, fileList) {
        this.ruleForm.img = response.data[0].img;
      },
      /*修改上传失败的钩子*/
      handleError1(response, file, fileList) {
        alert("上传失败");
      },
      timeFormatter(row){
        var date = new Date(row.bdate);
        return date.getFullYear() + '-' + (date.getMonth() + 1) + '-' + date.getDate();
      },
    },
    data() {
      return {
        tableData: [],
        name: '',
        cla: [],
        cname: '',
        //第几页
        currentPage: 1,
        /*一页几条*/
        pageSize: 3,
        /*总条数*/
        total: 0,
        /*批量删除*/
        ids: '',
          search: '',
        date1:'',
        dialogFormVisible: false,
        dialogFormVisibleUpd:false,
        form: {
          name: '',
          sex: '男',
          mgr: '',
          classno: '',
          bdate: '',
          img: ''
        },
        formLabelWidth: '120px',
        imgUrl: 'http://localhost:8080/upload/images/',
        ruleForm: {
          id:'',
          name: '',
          sex: '男',
          mgr: '',
          classno: '',
          bdate: '',
          img: ''
        },
      }
    },
    beforeCreate: function () {
      axios.get('http://localhost:8080/classes/queryAll')
        .then((response) => {
          this.cla = response.data;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    mounted() {
      axios.get('http://localhost:8080/student/queryAllVue?page=' + this.currentPage + '&limit=' + this.pageSize)
        .then((response) => {
          this.tableData = response.data.vueData;
          this.total = response.data.vueTotal;
          this.currentPage = response.data.vueCurrentPage;
          this.pageSize = response.data.vuePageSize;
        })
        .catch(function (error) {
          console.log(error);
        });
    }
  }
</script>
