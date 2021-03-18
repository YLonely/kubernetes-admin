<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="12">
        <iframe
          src="http://192.168.150.128:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=1&from=now-30s&to=now"
          frameborder="0"
        ></iframe>
      </el-col>
      <el-col :span="12">
        <iframe
          src="http://192.168.150.128:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=2&from=now-30s&to=now"
          frameborder="0"
        ></iframe>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col :span="12">
        <iframe
          src="http://192.168.150.128:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=7&from=now-30s&to=now"
          frameborder="0"
        ></iframe>
      </el-col>
      <el-col :span="12">
        <iframe
          src="http://192.168.150.128:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=8&from=now-30s&to=now"
          frameborder="0"
        ></iframe>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-row :gutter="20">
        <el-col :span="12">
          <iframe
            src="http://192.168.150.128:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=3&from=now-30s&to=now"
            frameborder="0"
          ></iframe>
        </el-col>
        <el-col :span="12">
          <iframe
            src="http://192.168.150.128:31114/d-solo/PQW4NA8Mz/original-open-faas-dashboard?orgId=1&refresh=5s&panelId=4&from=now-30s&to=now"
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