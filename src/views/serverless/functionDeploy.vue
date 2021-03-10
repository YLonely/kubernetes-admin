<template>
  <el-tabs v-model="activeName" type="border-card">
    <el-tab-pane label="使用镜像部署" name="image">
      <el-form ref="form" :model="form" label-width="80px" label-position="top">
        <el-form-item label="镜像名称:">
          <el-input v-model="form.imageName"></el-input>
        </el-form-item>
        <el-form-item label="函数名称:">
          <el-input v-model="form.funcName"></el-input>
        </el-form-item>
        <el-form-item label="环境变量:" style="margin-bottom: 10px">
        </el-form-item>
        <el-form-item v-for="(env, index) in form.envs" :key="index">
          <el-row :gutter="20" style="margin-top: -10px; margin-bottom: -10px">
            <el-col :span="12">
              <el-input
                placeholder="键:"
                v-model="form.envs[index].key"
              ></el-input>
            </el-col>
            <el-col :span="11">
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
        <el-form-item label="Labels:" style="margin-bottom: 10px">
        </el-form-item>
        <el-form-item v-for="(env, index) in form.labels" :key="index">
          <el-row :gutter="20" style="margin-top: -10px; margin-bottom: -10px">
            <el-col :span="12">
              <el-input
                placeholder="键:"
                v-model="form.labels[index].key"
              ></el-input>
            </el-col>
            <el-col :span="11">
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
            <el-dropdown-item command="text/javascript"
              >Javascript</el-dropdown-item
            >
            <el-dropdown-item command="text/x-python">Python</el-dropdown-item>
            <el-dropdown-item command="text/x-python-ml"
              >Python-Tensorflow</el-dropdown-item
            >
            <el-dropdown-item command="text/x-go">Golang</el-dropdown-item>
            <el-dropdown-item command="text/x-java">Java</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </span>
      <el-row> 函数语言：{{ readableCodeType }} </el-row>
      <div
        v-if="codeType === 'text/x-python' || codeType === 'text/x-python-ml'"
      >
        <el-row :gutter="10">
          <el-col
            style="
              border: 1px solid Gainsboro;
              margin-top: 20px;
              margin-left: 85px;
            "
            :span="11"
          >
            <codemirror v-model="code" :options="cmOptions" />
          </el-col>
          <el-col :span="11">
            <el-row> requirements.txt </el-row>
            <el-row>
              <el-input
                type="textarea"
                :autosize="{ minRows: 14, maxRows: 14 }"
                resize="none"
                placeholder="请输入依赖库"
                v-model="requirements"
                style="margin-top: 2px"
              >
              </el-input>
            </el-row>
          </el-col>
        </el-row>
      </div>
      <div v-else>
        <el-row>
          <el-col
            style="
              border: 1px solid Gainsboro;
              margin-top: 20px;
              margin-left: 85px;
            "
            :span="21"
          >
            <codemirror v-model="code" :options="cmOptions" />
          </el-col>
        </el-row>
      </div>
      <hr style="color: Gainsboro; margin-top: 40px" />
      <el-row style="margin-top: 20px">
        <el-form
          ref="form"
          :model="form"
          label-width="80px"
          label-position="top"
        >
          <el-form-item label="镜像名称:">
            <el-input
              v-model="form.imageName"
              placeholder="功能:版本，如 curl:v0"
            >
              <template slot="prepend">serverless.io/image/</template>
            </el-input>
          </el-form-item>
          <el-form-item label="函数检查点名称:" label-width="auto">
            <el-input :value="form.imageName" :readonly="true">
              <template slot="prepend">serverless.io/checkpoint/</template>
            </el-input>
          </el-form-item>
          <el-form-item label="函数名称:">
            <el-input v-model="form.funcName"></el-input>
          </el-form-item>
          <el-form-item label="环境变量:" style="margin-bottom: 10px">
          </el-form-item>
          <el-form-item v-for="(env, index) in form.envs" :key="index">
            <el-row
              :gutter="20"
              style="margin-top: -10px; margin-bottom: -10px"
            >
              <el-col :span="12">
                <el-input
                  placeholder="键:"
                  v-model="form.envs[index].key"
                ></el-input>
              </el-col>
              <el-col :span="11">
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
          <el-form-item label="Labels:" style="margin-bottom: 10px">
          </el-form-item>
          <el-form-item v-for="(env, index) in form.labels" :key="index">
            <el-row
              :gutter="20"
              style="margin-top: -10px; margin-bottom: -10px"
            >
              <el-col :span="12">
                <el-input
                  placeholder="键:"
                  v-model="form.labels[index].key"
                ></el-input>
              </el-col>
              <el-col :span="11">
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
      </el-row>
    </el-tab-pane>
  </el-tabs>
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
      activeName: "image",
      codeType: "text/javascript",
      readableCodeType: "Javascript",
      deployType: "fast",
      code: "",
      requirements: "",
      cmOptions: {
        tabSize: 4,
        mode: "text/javascript",
        theme: "idea",
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