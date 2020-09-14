<template>
  <el-container>
    <el-header style="padding: 10px">
      <el-row>
        <el-col :span="6">
          <h2 style="margin: 2px">
            <i class="el-icon-first-aid-kit" /> 医院叫号系统
          </h2>
        </el-col>
        <el-col :span="6" :offset="12">
          <el-button type="primary" @click="dialogFormVisible = true">我要挂号</el-button>
        </el-col>
      </el-row>

      <el-dialog title="挂号信息" :visible.sync="dialogFormVisible">
        <el-form :model="registerForm" :rules="rules" ref="registerForm">
          <el-form-item label="姓名 *" prop="name">
            <el-input v-model="registerForm.name" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="身份证号 *" prop="idNumber">
            <el-input
              v-model="registerForm.idNumber"
              autocomplete="off"
              placeholder="为测试方便输入四位数字即可"
            />
          </el-form-item>
          <el-form-item label="选择就诊科室 *" prop="department">
            <el-select v-model="registerForm.department" placeholder="请选择科室" style="width :100%">
              <el-option
                v-for="depart in departmentInfo"
                :label="depart.name"
                :value="depart.id.toString()"
                :key="depart.id"
                v-show="!depart.exam"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="选择就诊医生 *" prop="doctor">
            <el-select v-model="registerForm.doctor" placeholder="请选择医生" style="width :100%">
              <el-option
                v-for="doctor in doctorInfo"
                :key="doctor.id"
                :label="doctor.level"
                :value="doctor.id.toString()"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary"
                     @click="dialogFormVisible = false, partientRegister()">提交信息</el-button>
        </div>
      </el-dialog>
    </el-header>

    <el-main>
      <el-col :span="24">
        <el-menu
          default-active="1"
          class="el-menu-vertical-demo"
        >
          <el-submenu
            v-for="depart in departmentInfo"
            :key="depart.id"
            :index="depart.id.toString()"
          >
            <template slot="title">
              <h1>
                <i v-if="depart.exam" class="el-icon-s-platform"></i>
                <i v-else class="el-icon-office-building"></i>
                <span style="font-size: 20px">{{ depart.name }}</span>
              </h1>
            </template>

            <el-submenu
              v-for="doctor in doctorInfo"
              :key="doctor.id"
              :index="depart.id.toString() + '-' + doctor.id"
            >
              <template slot="title">
                <h3>
                  {{ depart.exam ? '机器' + doctor.id : doctor.level }}, 当前排队人数：
                  {{ filterPatient(doctor.id, depart.id).length }}
                </h3>
              </template>
              <el-col :span="20" :offset="2">
                <el-table :data="filterPatient(doctor.id, depart.id)" stripe>
                  <el-table-column prop="patientNumber" label="患者序号"></el-table-column>
                  <el-table-column prop="patientName" label="患者姓名"></el-table-column>
                  <el-table-column prop="patientIdNumber" label="身份证号"></el-table-column>
                  <el-table-column prop="patientTime" label="就诊时间"></el-table-column>
                  <el-table-column prop="patientState" label="就诊状态"></el-table-column>
                </el-table>
                <el-row style="margin-top: 16px;">
                  <el-button @click="startDiagnose(doctor.id, depart.id)"
                          :disabled="!filterPatient(doctor.id, depart.id).length">开始就诊</el-button>
                  <el-button @click="stopDiagnose(doctor.id, depart.id)"
                          :disabled="!filterPatient(doctor.id, depart.id).length||
                          !filterPatient(doctor.id,depart.id).find(
                          value=>value.patientState === diagnoseState.examing)">
                    结束就诊</el-button>
                </el-row>
              </el-col>
            </el-submenu>
          </el-submenu>
        </el-menu>
      </el-col>
    </el-main>

    <el-footer style="padding: 7px; font-weight: 600">
      1851381 赵中楷(1851381@tongji.edu.cn)
      <br />指导教师：武妍
    </el-footer>

    <el-dialog
      title="B超诊断"
      :visible.sync="dialogExamVisible">
      <h4>是否需要B超检查？</h4>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogExamVisible = false, ending()">不 需 要</el-button>
        <el-button type="primary" @click="dialogExamVisible = false, pushToExam()">需 要</el-button>
      </span>
    </el-dialog>
  </el-container>
</template>

<script>
import Vue from 'vue';
import moment from 'moment';

export default Vue.extend({
  name: 'Hospitalcall',
  methods: {
    startDiagnose(doctorId, departId) {
      // 开始就诊，改变就诊状态
      const curQueue = this.filterPatient(doctorId, departId);
      if (curQueue.length === 0) {
        return;
      }
      // console.log(curQueue);
      const patient = curQueue[0];
      patient.patientState = this.diagnoseState.examing;
    },
    stopDiagnose(doctorId, departId) {
      // 结束就诊
      const examDepart = this.departmentInfo.find((value) => !!value.exam);

      this.judgingDocktorId = doctorId;
      this.judgingDepartId = departId;

      if (examDepart && examDepart.id !== departId) {
        this.dialogExamVisible = true;
      } else {
        this.ending();
      }
    },
    ending() {
      // 结束就诊，移除病人信息
      const curQueue = this.filterPatient(this.judgingDocktorId, this.judgingDepartId);
      // console.log(curQueue);
      this.patientInfo.splice(this.patientInfo.indexOf(curQueue[0]), 1);
      // console.log(this.patientInfo);
    },
    pushToExam() {
      // 压入B超室队列
      const curQueue = this.filterPatient(this.judgingDocktorId, this.judgingDepartId);
      const examDepart = this.departmentInfo.find((value) => !!value.exam);

      const examPatients = this.patientInfo.filter((value) => value.department === examDepart.id);
      const counter = {};
      this.doctorInfo.forEach((value) => { counter[value.id] = 0; });
      if (examPatients) {
        examPatients.forEach((value) => { counter[value.doctor] += 1; });
      }

      const numArr = Object.values(counter);
      let maxIdx = numArr.length ? 0 : -1;
      numArr.forEach((value, idx) => { maxIdx = value < numArr[maxIdx] ? idx : maxIdx; });
      if (maxIdx !== -1) {
        curQueue[0].doctor = Number(Object.keys(counter)[maxIdx]);
        curQueue[0].department = examDepart.id;
        curQueue[0].patientNumber = counter[curQueue[0].doctor] + 1;
        curQueue[0].patientState = this.diagnoseState.waiting;
        // console.log(curQueue[0]);
      }
    },
    partientRegister() {
      // 挂号
      const { department, doctor } = this.registerForm;
      const curLength = this.filterPatient(Number(doctor), Number(department)).length;

      this.patientInfo.push({
        department: Number(department),
        doctor: Number(doctor),
        patientNumber: curLength + 1,
        patientName: this.registerForm.name,
        patientIdNumber: this.registerForm.idNumber,
        patientState: this.diagnoseState.waiting,
        patientTime: moment().format('YYYY-MM-DD hh:mm:ss'),
        requireExam: false,
      });
      this.$refs.registerForm.resetFields();
      // console.log(this.patientInfo.patientNumber);
    },
    filterPatient(doctor, department) {
      // 筛选病人
      return this.patientInfo.filter(
        (value) => value.department === department && value.doctor === doctor,
      );
    },
  },
  data() {
    const departmentNumber = 15;
    return {
      // departmentNumber: 15,
      dialogExamVisible: false,
      dialogFormVisible: false,
      registerForm: {
        name: '',
        idNumber: '',
        department: null,
        doctor: null,
      },
      formLabelWidth: '120px',
      rules: {
        name: [
          { required: true, message: '请输入患者姓名', triggger: 'change' },
        ],
        idNumber: [
          { required: true, message: '请输入患者身份证号', triggger: 'change' },
          // eslint-disable-next-line object-curly-newline
          { min: 4, max: 4, message: '请输入4位身份证号', trigger: 'change' },
        ],
        department: [
          { required: true, message: '请选择就诊科室', triggger: 'change' },
        ],
        doctor: [
          { required: true, message: '请选择就诊科室', triggger: 'change' },
        ],
      },
      diagnoseState: {
        waiting: '等待就诊',
        examing: '诊断中',
      },
      patientInfo: [],
      doctorInfo: [
        {
          id: 1,
          level: '专家',
        },
        {
          id: 2,
          level: '普通医师1',
        },
        {
          id: 3,
          level: '普通医师2',
        },
      ],
      departmentInfo: (() => {
        const arr = [];
        for (let i = 1; i <= departmentNumber; i += 1) {
          arr.push({
            id: i,
            name: `科室${i.toString()}`,
          });
        }
        arr.push({
          id: departmentNumber + 1,
          name: 'B超室',
          exam: true,
        });
        return arr;
      })(),
    };
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.el-header,
.el-footer {
  background-color: #b3c0d1;
  color: #333;
  text-align: center;
  /* line-height: 60px; */
}

.el-main {
  background-color: #e9eef3;
  color: #333;
  text-align: center;
  /* line-height: 160px; */
}

</style>
