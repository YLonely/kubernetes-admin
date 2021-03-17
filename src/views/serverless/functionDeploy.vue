<template>
  <el-card class="box-card">
    <el-form ref="form" :model="form" label-width="80px" label-position="top">
      <el-form-item label="镜像名称:">
        <el-row :gutter="10">
          <el-col :span="21">
            <el-input v-model="form.imageName"></el-input>
          </el-col>
          <el-col :span="3">
            <el-button
              size="mini"
              type="primary"
              @click="
                uploadDialogVisible = true;
                fileList = [];
              "
              >创建</el-button
            >
            <el-dialog
              title="创建函数镜像以及检查点"
              :visible.sync="uploadDialogVisible"
            >
              <el-upload
                action=""
                :before-remove="beforeRemove"
                multiple
                :limit="1"
                :on-exceed="handleExceed"
                :file-list="fileList"
              >
                <el-button size="small" type="primary"
                  >点击上传函数镜像文件</el-button
                >
                <div slot="tip" class="el-upload__tip">
                  只能上传tar文件，且不超过5GB
                </div>
              </el-upload>
              <el-form label-position="top">
                <el-form-item label="镜像名称">
                  <el-input
                    placeholder="函数功能+版本，如 curl:v0"
                    v-model="uploadForm.imageName"
                  >
                    <template slot="prepend">serverless.io/image/</template>
                  </el-input>
                </el-form-item>
                <el-form-item label="检查点名称">
                  <el-input v-model="uploadForm.checkpointName">
                    <template slot="prepend">serverless.io/checkpoint/</template>
                  </el-input>
                </el-form-item>
              </el-form>
            </el-dialog>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="函数名称:">
        <el-row>
          <el-col :span="21">
            <el-input v-model="form.funcName"></el-input>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="环境变量:" style="margin-bottom: 10px">
      </el-form-item>
      <el-form-item v-for="(env, index) in form.envs" :key="index">
        <el-row :gutter="10" style="margin-top: -10px; margin-bottom: -10px">
          <el-col :span="11">
            <el-input
              placeholder="键:"
              v-model="form.envs[index].key"
            ></el-input>
          </el-col>
          <el-col :span="10">
            <el-input
              placeholder="值:"
              v-model="form.envs[index].value"
            ></el-input>
          </el-col>
          <el-col :span="1">
            <el-button
              :icon="
                index === form.envs.length - 1
                  ? 'el-icon-circle-plus-outline'
                  : 'el-icon-remove-outline'
              "
              circle
              size="mini"
              @click="handleEnv(index)"
            >
            </el-button>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="Labels:" style="margin-bottom: 10px"> </el-form-item>
      <el-form-item v-for="(env, index) in form.labels" :key="index">
        <el-row :gutter="10" style="margin-top: -10px; margin-bottom: -10px">
          <el-col :span="11">
            <el-input
              placeholder="键:"
              v-model="form.labels[index].key"
            ></el-input>
          </el-col>
          <el-col :span="10">
            <el-input
              placeholder="值:"
              v-model="form.labels[index].value"
            ></el-input>
          </el-col>
          <el-col :span="1">
            <el-button
              :icon="
                index === form.labels.length - 1
                  ? 'el-icon-circle-plus-outline'
                  : 'el-icon-remove-outline'
              "
              circle
              size="mini"
              @click="handleLabel(index)"
            >
            </el-button>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click.prevent.native="createFromImage"
          >立即创建</el-button
        >
      </el-form-item>
    </el-form>
  </el-card>
</template>

<script>
import Axios from "axios";
const URLOpenfaas = "http://192.168.150.128:9528/serverless";
const instance = Axios.create({
  baseURL: URLOpenfaas,
  timeout: 1000,
});
export default {
  data() {
    return {
      uploadDialogVisible: false,
      fileList: [],
      uploadForm: {
        imageName: "",
        checkpointName: "",
      },
      form: {
        imageName: "",
        funcName: "",
        envs: [
          {
            key: "",
            value: "",
          },
        ],
        labels: [
          {
            key: "",
            value: "",
          },
        ],
      },
    };
  },
  methods: {
    createFromImage: function () {
      let envVars = {};
      let labels = {};
      for (let i = 0; i < this.form.envs.length; i++) {
        if (this.form.envs[i].key && this.form.envs[i].value) {
          envVars[this.form.envs[i].key] = this.form.envs[i].value;
        }
      }
      for (let i = 0; i < this.form.labels.length; i++) {
        if (this.form.labels[i].key && this.form.labels[i].value) {
          labels[this.form.labels[i].key] = this.form.labels[i].value;
        }
      }
      instance
        .post(
          "/system/functions",
          {
            annotations: {},
            envVars,
            image: this.form.imageName,
            labels,
            namespace: "openfaas-fn",
            secrets: [],
            service: this.form.funcName,
          },
          {
            headers: { "Content-Type": "application/json" },
          }
        )
        .then(() => {
          this.$message("部署成功");
        })
        .catch((error1) => {
          this.$message("发生错误：" + error1.statusText + "\n" + error1.data);
        });
    },
    handleEnv: function (index) {
      if (index === this.form.envs.length - 1) {
        this.form.envs.push({});
      } else {
        this.form.envs.splice(index, 1);
      }
    },
    handleLabel: function (index) {
      if (index === this.form.labels.length - 1) {
        this.form.labels.push({});
      } else {
        this.form.labels.splice(index, 1);
      }
    },
    handleExceed(files, fileList) {
      this.$message.warning(
        `当前限制选择 1 个文件，本次选择了 ${files.length} 个文件，共选择了 ${
          files.length + fileList.length
        } 个文件`
      );
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`);
    },
  },
};
</script>