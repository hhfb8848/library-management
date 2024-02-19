<template>
  <div style="height: 100vh; overflow: hidden; position: relative">
    <el-form
      :label-position="left"
      label-width="120px"
    
    >
      <el-form-item label="登录名称：">
        <span>{{ user.username }}</span>
      </el-form-item>
      <el-form-item label="手机号：">
        <span>{{ user.phone }}</span>
      </el-form-item>
      <el-form-item label="邮箱号：">
      <span>{{ user.email }}</span>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import request from "@/utils/request";
import Cookies from "js-cookie";

export default {
  name: "center",
  data() {
    return {
      user: Cookies.get("admin") ? JSON.parse(Cookies.get("admin")) : {},
    };
  },
  methods: {},
};
</script>

<style></style>
