<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="120px"
    >
      <el-form-item label="场次名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="场次名称"></el-input>
      </el-form-item>
      <el-form-item label="每日开始时间" prop="startTime">
        <el-date-picker value-format="yyyy-MM-dd HH:mm:ss" type="datetime" placeholder="每日开始时间" v-model="dataForm.startTime" ></el-date-picker>
      </el-form-item>
      <el-form-item label="每日结束时间" prop="endTime">
        <el-date-picker value-format="yyyy-MM-dd HH:mm:ss" type="datetime" placeholder="每日结束时间" v-model="dataForm.endTime" ></el-date-picker>
      </el-form-item>
      <el-form-item label="启用状态" prop="status">
        <el-input v-model="dataForm.status" placeholder="启用状态"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: 0,
        name: "",
        startTime: "",
        endTime: "",
        status: "",
        createTime: ""
      },
      dataRule: {
        name: [
          { required: true, message: "场次名称不能为空", trigger: "blur" }
        ],
        startTime: [
          { required: true, message: "每日开始时间不能为空", trigger: "blur" }
        ],
        endTime: [
          { required: true, message: "每日结束时间不能为空", trigger: "blur" }
        ],
        status: [
          { required: true, message: "启用状态不能为空", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/coupon/seckillsession/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.seckillSession.name;
              this.dataForm.startTime = data.seckillSession.startTime;
              this.dataForm.endTime = data.seckillSession.endTime;
              this.dataForm.status = data.seckillSession.status;
              this.dataForm.createTime = data.seckillSession.createTime;
            }
          });
        }
      });
    },
    format(date,format){

//替换 year yyyy
      format = format.replace("yyyy",date.getFullYear().toString());


//替换 month MM  this.getMonth()返回0-11
      format = format.replace("MM",(date.getMonth()+1).toString());


//替换 day dd
      format = format.replace("dd",date.getDate().toString());


//替换 hour HH
      format = format.replace("HH",date.getHours().toString());


//替换 minutes mm
      format = format.replace("mm",date.getMinutes().toString());


//替换 seconds ss
      format = format.replace("ss",date.getSeconds().toString());


//替换 weekdays ww  this.getDay()返回的是0-6
//       format = format.replace("ww",this.getWeekday(this.getDay()));


      return format;

    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/coupon/seckillsession/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              name: this.dataForm.name,
              startTime: this.dataForm.startTime,
              endTime: this.dataForm.endTime,
              status: this.dataForm.status,
              createTime: this.format(new Date(),"yyyy-MM-dd HH:mm:ss")
            })
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                }
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    }
  }
};
</script>
