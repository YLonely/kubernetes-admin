<template>
  <el-tabs v-model="activeName" type="border-card">
    <el-tab-pane label="使用镜像部署" name="image">
      <el-form ref="form" :model="form" label-width="80px">
        <el-form-item label="镜像名称:">
          <el-input v-model="form.imageName"></el-input>
        </el-form-item>
        <el-form-item label="函数名称:">
          <el-input v-model="form.funcName"></el-input>
        </el-form-item>
        <el-form-item label="部署方式:">
          <el-radio v-model="deployType" label="fast"
            >快速部署(使用优化技术加速无服务器函数启动)</el-radio
          >
          <el-radio v-model="deployType" label="normal">普通部署</el-radio>
        </el-form-item>
        <el-form-item label="环境变量:"> </el-form-item>
        <el-form-item v-for="(env, index) in form.envs" :key="index">
          <el-row :gutter="20" style="margin-top: -10px; margin-bottom: -10px">
            <el-col :span="10">
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
        <el-form-item label="Labels:"> </el-form-item>
        <el-form-item v-for="(env, index) in form.labels" :key="index">
          <el-row :gutter="20" style="margin-top: -10px; margin-bottom: -10px">
            <el-col :span="10">
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
    </el-tab-pane>
    <el-tab-pane name="code">
      <span slot="label">
        <el-dropdown @command="handleCodeType">
          <span class="el-dropdown-link">
            全新部署<i class="el-icon-arrow-down el-icon--right"></i>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="javascript">Javascript</el-dropdown-item>
            <el-dropdown-item command="python">Python</el-dropdown-item>
            <el-dropdown-item command="python-ml"
              >Python-Tensorflow</el-dropdown-item
            >
            <el-dropdown-item command="golang">Golang</el-dropdown-item>
            <el-dropdown-item command="java">Java</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </span>
      <div v-if="codeType === 'python' || codeType === 'python-ml'">
        <codemirror v-model="code" :options="cmOptions" />
      </div>
      <div v-else>else</div>
    </el-tab-pane>
  </el-tabs>
</template>

<script>
import { codemirror } from "vue-codemirror";
import "codemirror/lib/codemirror.css";
import "codemirror/mode/javascript/javascript.js";

// import theme style
import "codemirror/theme/base16-dark.css";
export default {
  components: {
    codemirror,
  },
  data() {
    return {
      activeName: "image",
      codeType: "javascript",
      deployType: "fast",
      code: "hello",
      cmOptions: {
        tabSize: 4,
        mode: "text/javascript",
        theme: "base16-dark",
        lineNumbers: true,
        line: true,
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
    handleCodeType: function (type) {
      this.activeName = "code";
      this.codeType = type;
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