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
        <el-date-picker
          type="datetime"
          placeholder="每日开始时间"
          v-model="dataForm.startTime"
          value-format="yyyy-MM-dd HH:mm:ss"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="每日结束时间" prop="endTime">
        <el-date-picker
          type="datetime"
          placeholder="每日结束时间"
          v-model="dataForm.endTime"
                    value-format="yyyy-MM-dd HH:mm:ss"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="启用状态" prop="status">
        <el-select v-model="dataForm.status" placeholder="上下线状态">
          <el-option :value="true" label="上线"></el-option>
          <el-option :value="false" label="下线"></el-option>
        </el-select>
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
      requstMethod: "",
      dataForm: {
        id: 0,
        name: "",
        startTime: "",
        endTime: "",
        status: "",
        createTime: "",
      },
      dataRule: {
        name: [
          { required: true, message: "场次名称不能为空", trigger: "blur" },
        ],
        startTime: [
          { required: true, message: "每日开始时间不能为空", trigger: "blur" },
        ],
        endTime: [
          { required: true, message: "每日结束时间不能为空", trigger: "blur" },
        ],
        status: [
          { required: true, message: "启用状态不能为空", trigger: "blur" },
        ],
      },
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
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.data.name;
              this.dataForm.startTime = data.data.startTime;
              this.dataForm.endTime = data.data.endTime;
              this.dataForm.status = data.data.status;
              this.dataForm.createTime = data.data.createTime;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          if (this.dataForm.id) {
            this.requestMethod = "put";
          } else {
            this.requestMethod = "post";
          }
          this.$http({
            url: this.$http.adornUrl(`/coupon/seckillsession`),
            method: this.requestMethod,
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              name: this.dataForm.name,
              startTime: this.dataForm.startTime,
              endTime: this.dataForm.endTime,
              status: this.dataForm.status            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
};
</script>
