<template>
  <div class="main-content">

    <div style="display: flex; margin-bottom: 10px">
      <div style="flex: 1">
        <el-button type="primary" class="btn-green" size="medium" @click="handleAdd">创建新的问卷</el-button>
      </div>
      <div>
        <el-input v-model="name" style="width: 300px; margin-right: 10px" size="medium" placeholder="请输入名称搜索问卷" clearable></el-input>
        <el-button type="primary" size="medium" class="btn-green" @click="load(1)">搜 索</el-button>
      </div>
    </div>

    <div style="margin-bottom: 10px">
      <div class="card" v-for="item in tableData" :key="item.id" style="margin-bottom: 5px">
        <div style="display: flex;">
          <div style="flex: 1; width: 0;">
            <div style="display: flex; align-items: center; margin-bottom: 10px">
              <el-tag v-if="item.saved === '否'" type="warning">未发布</el-tag>
              <el-tag v-if="item.saved === '是'" type="success">已发布</el-tag>
              <span style="font-size: 20px; margin-left: 10px">{{ item.name }}</span>
            </div>

            <div>
              <el-button type="text" style="color: #2a60c9; font-size: 16px" @click="handleEdit(item)"><i class="el-icon-edit"></i> 编辑</el-button>
              <el-button type="text" style="color: #2a60c9; font-size: 16px" @click="design(item.id)"><i class="el-icon-setting"></i> 设计</el-button>
              <el-button type="text" style="color: #2a60c9; font-size: 16px" @click="copy(item.id)"><i class="el-icon-document-copy"></i> 复制</el-button>
              <el-button type="text" style="color: #fc4b4b; font-size: 16px" v-if="item.open === '否'" @click="del(item.id)"><i class="el-icon-delete"></i> 删除</el-button>
              <el-button type="text" style="color: #2a60c9; font-size: 16px" @click="share(item.id)" v-if="item.saved === '是'"><i class="el-icon-share"></i> 分享</el-button>
              <el-button type="text" style="color: orange; font-size: 16px" @click="getPageCount(item.id)" v-if="item.saved === '是'"><i class="el-icon-s-marketing"></i> 数据统计</el-button>
            </div>
          </div>

          <div style="width: 100px">
            <img :src="item.img" alt="" style="width: 100%; border-radius: 5px; display: block">
          </div>
        </div>
      </div>
    </div>

    <div v-if="total">
      <el-pagination
          background
          @current-change="handleCurrentChange"
          :current-page="pageNum"
          :page-size="pageSize"
          layout="total, prev, pager, next"
          :total="total">
      </el-pagination>
    </div>

    <el-dialog title="信息" :visible.sync="fromVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
      <el-form :model="form" label-width="100px" style="padding-right: 50px" :rules="rules" ref="formRef">
        <el-form-item label="模板名称" prop="name">
          <el-input v-model="form.name" placeholder="模板名称"></el-input>
        </el-form-item>
        <el-form-item label="模板介绍" prop="descr">
          <el-input v-model="form.descr" placeholder="模板介绍"></el-input>
        </el-form-item>
        <el-form-item label="封面图" prop="img">
          <el-upload
              :action="$baseUrl + '/files/upload'"
              :headers="{ token: user.token }"
              list-type="picture"
              :on-success="handleImgSuccess"
          >
            <el-button type="primary">上传</el-button>
          </el-upload>
        </el-form-item>
        <el-form-item label="是否公开" prop="open" v-if="user.role === 'ADMIN'">
          <el-radio-group v-model="form.open">
            <el-radio label="是"></el-radio>
            <el-radio label="否"></el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="fromVisible = false">取 消</el-button>
        <el-button type="primary" @click="save">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog title="数据统计" :visible.sync="pageCountVisible" width="50%" :close-on-click-modal="false" destroy-on-close>
      <div v-if="pageCount.answerCount">
        <div style="margin-bottom: 10px">
          <span>共收集 <strong>{{ pageCount.answerCount }}</strong> 份问卷调查结果</span>
        </div>

        <div class="card" v-for="(item, index) in pageCount.questionList" :key="index" style="margin-bottom: 10px">
          <div style="margin-bottom: 10px"><strong>Q: {{ item.name }}</strong></div>
          <div v-if="item.questionItemList?.length">
            <el-table :data="item.questionItemList" v-if="item.type === '单选题' || item.type === '多选题'">
              <el-table-column label="选项" prop="content"></el-table-column>
              <el-table-column label="小计" prop="count"></el-table-column>
              <el-table-column label="比例">
                <template v-slot="scope">
                  <el-progress :percentage="scope.row.percentage" :color="randomColor"></el-progress>
                </template>
              </el-table-column>
            </el-table>

            <el-table :data="item.questionItemList" v-else>
              <el-table-column label="内容" prop="content"></el-table-column>
            </el-table>
          </div>
          <div style="margin-top: 10px">有效填写：{{ item.count  }}</div>
        </div>

      </div>
      <div v-else style="text-align: center; color: #888">
        您未收集到任何调查结果
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="pageCountVisible = false">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog title="分享" :visible.sync="shareVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
      <div style="display: flex; align-items: center">
        <span style="margin-right: 5px">链接</span> <el-input v-model="link" style="flex: 1; margin-right: 5px" />
        <el-button type="primary" @click="copyLink">复制</el-button>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="shareVisible = false">关 闭</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
export default {
  name: "FPages",
  data() {
    return {
      tableData: [],  // 所有的数据
      pageNum: 1,   // 当前的页码
      pageSize: 4,  // 每页显示的个数
      total: 0,
      name: null,
      fromVisible: false,
      form: {},
      rules: {
        name: [
          {required: true, message: '请输入名称', trigger: 'blur'},
        ],
        descr: [
          {required: true, message: '请输入简介', trigger: 'blur'},
        ]
      },
      user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
      pageCount: {},
      pageCountVisible: false,
      shareVisible: false,
      link: ''
    }
  },
  created() {
    this.load(1)
  },
  methods: {
    copy(pageId) {
      this.$request.post('/pages/copy?pageId=' + pageId).then(res => {
        if (res.code === '200') {
          this.$message.success("复制成功！")
          this.load(1)
        } else {
          this.$message.error(res.msg)
        }
      })
    },
    randomColor() {
      let arr = ['#f56c6c', '#e6a23c', '#5cb87a', '#1989fa', '#6f7ad3']
      return arr[Math.floor(Math.random() * arr.length)]  //  0.8 * 1  = 0
    },
    copyLink() {
      let _input = document.createElement("input");   // 直接构建input

      _input.value = this.link;  // 设置内容
      document.body.appendChild(_input);    // 添加临时实例
      _input.select();   // 选择实例内容
      document.execCommand("Copy");   // 执行复制
      document.body.removeChild(_input)
      this.$message.success("复制成功")
    },
    share(pageId) {
      this.link = location.href.substring(0, location.href.indexOf('/front')) + '/realPage?pageId=' + pageId
      this.shareVisible = true
    },
    getPageCount(pageId) {
      this.$request.get('/pageCount', {
        params: {
          pageId: pageId
        }
      }).then(res => {
        this.pageCount = res.data || {}
        this.pageCountVisible = true
      })
    },
    del(pageId) {   // 单个删除
      this.$confirm('您确定删除吗？', '确认删除', {type: "warning"}).then(response => {
        this.$request.delete('/pages/delete/' + pageId).then(res => {
          if (res.code === '200') {   // 表示操作成功
            this.$message.success('操作成功')
            this.load(1)
          } else {
            this.$message.error(res.msg)  // 弹出错误的信息
          }
        })
      }).catch(() => {
      })
    },
    design(pageId) {
      window.open('/front/design?pageId=' + pageId)
    },
    handleAdd() {   // 新增数据
      this.form = {}  // 新增数据的时候清空数据
      this.fromVisible = true   // 打开弹窗
    },
    handleEdit(row) {   // 编辑数据
      this.form = JSON.parse(JSON.stringify(row))  // 给form对象赋值  注意要深拷贝数据
      this.fromVisible = true   // 打开弹窗
    },
    save() {   // 保存按钮触发的逻辑  它会触发新增或者更新
      this.$refs.formRef.validate((valid) => {
        if (valid) {
          this.$request({
            url: this.form.id ? '/pages/update' : '/pages/add',
            method: this.form.id ? 'PUT' : 'POST',
            data: this.form
          }).then(res => {
            if (res.code === '200') {  // 表示成功保存
              this.$message.success('保存成功')
              this.load(1)
              this.fromVisible = false
            } else {
              this.$message.error(res.msg)  // 弹出错误的信息
            }
          })
        }
      })
    },
    load(pageNum) {  // 分页查询
      if (pageNum) this.pageNum = pageNum
      this.$request.get('/pages/selectPage', {
        params: {
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          name: this.name,
        }
      }).then(res => {
        this.tableData = res.data?.list
        this.total = res.data?.total
      })
    },
    reset() {
      this.name = null
      this.load(1)
    },
    handleCurrentChange(pageNum) {
      this.load(pageNum)
    },
    handleImgSuccess(res) {
      this.form.img = res.data  // 把后台返回的图片url赋值给表单属性 img
    },
  }
}
</script>

<style scoped>

</style>