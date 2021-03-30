<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="12">
        <el-row>
          <iframe
            src="http://133.133.135.25:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=7&from=now-30s&to=now"
            frameborder="0"
            style="height: 248px"
          ></iframe>
        </el-row>
        <el-row>
          <iframe
            src="http://133.133.135.25:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=10&from=now-30s&to=now"
            frameborder="0"
            style="height: 249px"
          ></iframe>
        </el-row>
      </el-col>
      <el-col :span="12">
        <iframe
          src="http://133.133.135.25:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=8&from=now-30s&to=now"
          frameborder="0"
          style="height: 500px"
        ></iframe>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-row :gutter="20">
        <el-col :span="12" style="padding-left: 20px">
          <iframe
            src="http://133.133.135.25:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=3&from=now-30s&to=now"
            frameborder="0"
          ></iframe>
        </el-col>
        <el-col :span="12" style="padding-right: 20px">
          <iframe
            src="http://133.133.135.25:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=2&from=now-30s&to=now"
            frameborder="0"
          ></iframe>
        </el-col>
      </el-row>
    </el-row>
  </div>
</template>

<style scoped>
iframe {
  height: 230px;
  width: 100%;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
}
</style>