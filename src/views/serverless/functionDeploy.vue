<template>
  <el-tabs v-model="activeName" type="border-card">
    <el-tab-pane label="使用镜像部署" name="image">
      <el-form ref="form" :model="form" label-width="80px">
        <el-form-item label="镜像名称">
          <el-input v-model="form.imageName"></el-input>
        </el-form-item>
        <el-form-item label="函数名称">
          <el-input v-model="form.funcName"></el-input>
        </el-form-item>
        <el-form-item label="环境变量:"> </el-form-item>
        <el-form-item v-for="(env, index) in form.envs" :key="env.key">
          <el-row :gutter="20">
            <el-col :span="10">
              <el-input placeholder="键："></el-input>
            </el-col>
            <el-col :span="10">
              <el-input placeholder="值："></el-input>
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
              >Python-MachineLearning</el-dropdown-item
            >
            <el-dropdown-item command="golang">Golang</el-dropdown-item>
            <el-dropdown-item command="java">Java</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </span>
      <div v-if="codeType === 'python' || codeType === 'python-ml'">python</div>
      <div v-else>else</div>
    </el-tab-pane>
  </el-tabs>
</template>

<script>
import func from "vue-editor-bridge";
export default {
  data() {
    return {
      activeName: "image",
      codeType: "javascript",
      form: {
        imageName: "",
        funcName: "",
        envs: [
          {
            key: "key1",
            value: "value1",
          },
          {
            key: "key2",
            value: "value2",
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
    handleEnv: function (index) {},
  },
};
</script>