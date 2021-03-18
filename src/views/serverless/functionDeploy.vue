<template>
  <el-card class="box-card">
    <el-form ref="form" :model="form" label-width="80px" label-position="top">
      <el-form-item label="镜像名称:">
        <el-row :gutter="10">
          <el-col :span="21">
            <el-autocomplete
              v-model="form.imageName"
              :fetch-suggestions="fetchImages"
              clearable
              style="width: 100%"
            >
            </el-autocomplete>
          </el-col>
          <el-col :span="3">
            <el-button
              size="mini"
              type="primary"
              @click="
                uploadDialogVisible = true;
                fileList = [];
                uploadForm.imageName = '';
              "
              >创建</el-button
            >
            <el-dialog
              title="创建函数镜像以及检查点"
              :visible.sync="uploadDialogVisible"
            >
              <el-upload
                action="http://192.168.150.128:9528/manager/image/upload"
                :before-remove="beforeRemove"
                multiple
                :limit="1"
                :on-change="fileChange"
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
                  <el-input v-model="uploadForm.imageName" :readonly="true">
                    <template slot="prepend"
                      >serverless.io/checkpoint/</template
                    >
                  </el-input>
                </el-form-item>
                <el-form-item style="margin-top: 20px">
                  <el-button
                    type="primary"
                    @click.prevent.native="handleCreateImage"
                    :loading="uploadForm.loading"
                    >立即创建</el-button
                  >
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
const URLManager = "http://192.168.150.128:9528/manager";
const invokerOpenfaas = Axios.create({
  baseURL: URLOpenfaas,
  timeout: 1000,
});
const invokerManager = Axios.create({
  baseURL: URLManager,
  timeout: 10000,
});
const errMsg = new Map([
  ["initial", "初始化失败"],
  ["import image", "导入镜像失败"],
  ["start container", "启动函数异常"],
  ["checkpoint", "无法正常创建检查点"],
]);
const defaultCapacity = 10;
export default {
  data() {
    return {
      uploadDialogVisible: false,
      fileList: [],
      uploadForm: {
        tarFileName: "",
        imageName: "",
        loading: false,
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
      if (this.form.imageName.includes("/checkpoint/")) {
        let parts = this.form.imageName.split(":");
        let name = parts[0];
        let version = parts[1];
        name = "checkpoint-" + name.replaceAll(/\//g, "_");
        labels[name] = version;
      }
      invokerOpenfaas
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
    fileChange(file, fileList) {
      this.uploadForm.tarFileName = file.name;
    },
    fetchImages(queryString, cb) {
      invokerManager
        .post(
          "/image",
          {
            prefix: "serverless.io/",
            namespace: "k8s.io",
          },
          {
            headers: { "Content-Type": "application/json" },
          }
        )
        .then((resp) => {
          let images = [];
          if (resp.data.images) {
            resp.data.images.forEach((item, index, array) => {
              images.push({
                value: item,
              });
            });
          }
          cb(images);
        })
        .catch((err) => {
          this.$message.error("调用错误：" + err.statusText + "\n" + err.data);
        });
    },
    handleCreateImage: function () {
      this.uploadForm.loading = true;
      invokerManager
        .post(
          "/checkpoint",
          {
            tar_file_name: this.uploadForm.tarFileName,
            image_name: "serverless.io/image/" + this.uploadForm.imageName,
            checkpoint_name:
              "serverless.io/checkpoint/" + this.uploadForm.imageName,
            namespace: "k8s.io",
          },
          {
            headers: { "Content-Type": "application/json" },
          }
        )
        .then((resp) => {
          let data = resp.data;
          if (data.error === "") {
            invokerManager
              .post(
                "/namespace/update",
                {
                  checkpoint_name:
                    "serverless.io/checkpoint/" + this.uploadForm.imageName,
                  checkpoint_namespace: "k8s.io",
                  capacity: defaultCapacity,
                },
                {
                  headers: { "Content-Type": "application/json" },
                }
              )
              .then((resp1) => {
                if (resp1.data.message === "OK") {
                  this.$message("创建成功！");
                } else {
                  this.$message.error("初始化函数资源失败：" + resp1.message);
                }
              })
              .catch((error1) => {
                this.$message.error(
                  "调用错误：" + error1.statusText + "\n" + error1.data
                );
              });
          } else {
            let msg = this.errMsg.get(data.error);
            if (msg === undefined) {
              this.$message.error("创建失败：未知错误");
            } else {
              this.$message.error("创建失败：" + msg);
            }
          }
        })
        .catch((err) => {
          this.$message.error("调用错误：" + err.statusText + "\n" + err.data);
        })
        .finally(() => {
          this.uploadForm.loading = false;
        });
    },
  },
};
</script>