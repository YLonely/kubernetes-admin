<template>
  <el-card class="box-card">
    <el-form ref="form" :model="form" label-width="80px" label-position="top">
      <el-form-item label="镜像名称:">
        <el-row :gutter="10">
          <el-col :span="21">
            <el-input v-model="form.imageName"></el-input>
          </el-col>
          <el-col :span="3">
            <el-button size="mini" type="primary">上传镜像</el-button>
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
import { codemirror } from "vue-codemirror";
import "codemirror/lib/codemirror.css";
import "codemirror/mode/javascript/javascript.js";
import "codemirror/mode/python/python.js";
import "codemirror/mode/go/go.js";
import "codemirror/mode/clike/clike.js";

// import theme style
import "codemirror/theme/idea.css";
export default {
  components: {
    codemirror,
  },
  data() {
    return {
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
    handleCodeType: function (type) {
      this.activeName = "code";
      this.codeType = type;
      if (type === "text/x-python-ml") {
        this.cmOptions.mode = "text/x-python";
      } else {
        this.cmOptions.mode = type;
      }
      switch (type) {
        case "text/x-python":
        case "text/x-python-ml":
          this.readableCodeType = "Python";
          break;
        case "text/javascript":
          this.readableCodeType = "Javascript";
          break;
        case "text/x-go":
          this.readableCodeType = "Golang";
          break;
        case "text/x-java":
          this.readableCodeType = "Java";
          break;
      }
    },
    createFromImage: function () {},
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
  },
};
</script>