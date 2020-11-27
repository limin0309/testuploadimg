<template>
  <div class="basicForm">
    <!-- <el-card>
      <div slot="header" class="clearfix">
        <span class="titleName">发布车源</span>
        <span
          >请将带<a class="colorRule">*</a
          >的必填项填写完整后发布，更高的车辆资料完整度，能提高车辆检索排名</span
        >
      </div> -->

    <el-page-header content="ddddddd" title="title">
      <!-- <div>
      <el-link href="https://element.eleme.io" target="_blank"
        >默认链接</el-link
      >
      <el-link type="primary">主要链接</el-link>
    </div> -->
      <el-card class="hhhhh">
        <div slot="header" class="clearfix">
          <span class="titleName">发布车源</span>
          <span
            >请将带<a class="colorRule">*</a
            >的必填项填写完整后发布，更高的车辆资料完整度，能提高车辆检索排名</span
          >
        </div>
        <el-form
          :model="ruleForm"
          :rules="rules"
          ref="ruleForm"
          label-width="100px"
          class="demo-ruleForm"
        >
          <el-form-item label="车架号(vin)" prop="vin">
            <el-input v-model="ruleForm.vin"></el-input>
          </el-form-item>
          <el-form-item label="车辆名称" prop="options">
            <el-cascader
              :options="options"
              v-model="ruleForm.options"
            ></el-cascader>
          </el-form-item>
          <el-form-item label="变速箱" prop="biansu">
            <el-select v-model="ruleForm.biansu" placeholder="请选择变速箱">
              <el-option label="变速箱一" value="sudu1"></el-option>
              <el-option label="变速箱二" value="sudu2"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="排量" prop="region">
            <el-select v-model="ruleForm.region" placeholder="请选择变速箱">
              <el-option label="排量一" value="2.0"></el-option>
              <el-option label="排量二" value="3.0"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="是否上牌" prop="card">
            <el-radio-group v-model="ruleForm.card">
              <el-radio label="已上牌"></el-radio>
              <el-radio label="未上牌"></el-radio>
            </el-radio-group>
          </el-form-item>

          <el-form-item label="活动时间" required>
            <el-col :span="11">
              <el-form-item prop="date1">
                <el-date-picker
                  type="date"
                  placeholder="选择日期"
                  v-model="ruleForm.date1"
                  style="width: 100%"
                ></el-date-picker>
              </el-form-item>
            </el-col>
            <el-col class="line" :span="2">-</el-col>
            <el-col :span="11">
              <el-form-item prop="date2">
                <el-time-picker
                  placeholder="选择时间"
                  v-model="ruleForm.date2"
                  style="width: 100%"
                ></el-time-picker>
              </el-form-item>
            </el-col>
          </el-form-item>
          <el-form-item label="即时配送" prop="delivery">
            <el-switch v-model="ruleForm.delivery"></el-switch>
          </el-form-item>
          <el-form-item label="活动性质" prop="type">
            <el-checkbox-group v-model="ruleForm.type">
              <el-checkbox label="美食/餐厅线上活动" name="type"></el-checkbox>
              <el-checkbox label="地推活动" name="type"></el-checkbox>
              <el-checkbox label="线下主题活动" name="type"></el-checkbox>
              <el-checkbox label="单纯品牌曝光" name="type"></el-checkbox>
            </el-checkbox-group>
          </el-form-item>
          <el-form-item label="特殊资源" prop="resource">
            <el-radio-group v-model="ruleForm.resource">
              <el-radio label="线上品牌商赞助"></el-radio>
              <el-radio label="线下场地免费"></el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="活动形式" prop="desc">
            <el-input type="textarea" v-model="ruleForm.desc"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')"
              >立即创建</el-button
            >
            <el-button @click="resetForm('ruleForm')">重置</el-button>
          </el-form-item>
        </el-form>
      </el-card>
    </el-page-header>

    <!-- </el-card> -->
  </div>
</template>
<script>
export default {
  data() {
    return {
      options: [
        {
          value: 'guangdong',
          label: '广东省',
          children: [
            {
              value: 'guangzhou',
              label: '广州市',
              children: [
                {
                  value: 'tianhe',
                  label: '天河区',
                },
                {
                  value: 'haizhu',
                  label: '海珠区',
                },
              ],
            },
            {
              value: 'dongguan',
              label: '东莞市',
              children: [
                {
                  value: 'changan',
                  label: '长安镇',
                },
                {
                  value: 'humen',
                  label: '虎门镇',
                },
              ],
            },
          ],
        },
        {
          value: 'hunan',
          label: '湖南省',
          children: [
            {
              value: 'changsha',
              label: '长沙市',
              children: [
                {
                  value: 'yuelu',
                  label: '岳麓区',
                },
              ],
            },
          ],
        },
      ],
      ruleForm: {
        vin: '', // vin
        options: [], // 车辆名称
        biansu: '', // 变速箱
        region: '', // 排量
        card: '', // 是否上牌
        date1: '',
        date2: '',
        delivery: false,
        type: [],
        resource: '',
        desc: '',
      },
      rules: {
        vin: [
          { required: true, message: '请输入车架号', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' },
        ],
        options: [{ required: true, message: '请输入', trigger: 'change' }],
        biansu: [
          { required: true, message: '请选择变速箱', trigger: 'change' },
        ],
        region: [{ required: true, message: '请选择排量', trigger: 'change' }],
        card: [
          { required: true, message: '请选择是否上牌', trigger: 'change' },
        ],
        date1: [
          {
            type: 'date',
            required: true,
            message: '请选择日期',
            trigger: 'change',
          },
        ],
        date2: [
          {
            type: 'date',
            required: true,
            message: '请选择时间',
            trigger: 'change',
          },
        ],
        type: [
          {
            type: 'array',
            required: true,
            message: '请至少选择一个活动性质',
            trigger: 'change',
          },
        ],
        resource: [
          { required: true, message: '请选择活动资源', trigger: 'change' },
        ],
        desc: [{ required: true, message: '请填写活动形式', trigger: 'blur' }],
      },
    };
  },
  methods: {
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        console.log(this.ruleForm, 'this.ruleForm');

        if (valid) {
          alert('submit!');
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
  },
};
</script>
<style scoped>
.titleName {
  font-size: 16px;
  font-weight: 500;
  padding-right: 40px;
}
.colorRule {
  color: red;
}
.el-card__body {
  padding: 0;
}
.basicForm {
  width: 800px;
}
</style>
