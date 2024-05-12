<template>
  <div class="page-layout">
    <div class="top-view">
      <div class="left-view">
        <div class="title-view">
          <span :style="isDocument ? 'cursor: pointer' : ''" :title="isDocument ? (isStretch ? '收起侧栏' : '弹出侧栏') : ''" @click="iconClick">
            <svg-icon name="md" customStyle="width: 20px; height: 20px; margin: 5px 5px 0 0"></svg-icon>
          </span>
          <span :style="isDocument ? 'cursor: pointer' : ''" :title="isDocument ? (onlyPreview ? '编辑模式' : '预览模式') : ''" @click="titleClick">
            云文档
          </span>
        </div>
        <div class="menu-view">
          <router-link to="/document">文档</router-link>
          <router-link to="/picture">图片</router-link>
          <router-link to="/tool">工具</router-link>
        </div>
      </div>
      <el-dropdown class="right-view">
        <div class="text-view">{{ name }}</div>
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item style="user-select: none" @click="publishClick">公开文档</el-dropdown-item>
            <el-dropdown-item style="user-select: none" @click="dialogVisible = true">修改密码</el-dropdown-item>
            <el-dropdown-item style="user-select: none" @click="logout">退出登录</el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
    <router-view class="content-view" :onlyPreview="onlyPreview" :isStretch="isStretch"></router-view>
    <el-dialog v-model="dialogVisible" title="修改密码" width="400px" :show-close="false" :before-close="dialogClose">
      <form>
        <el-input v-model.trim="form.password" size="large" type="password" clearable placeholder="请输入原密码"></el-input>
        <el-input style="margin: 10px 0" v-model.trim="form.newPassword" size="large" type="password" clearable placeholder="请输入新密码"></el-input>
        <el-input v-model.trim="form.confirmPassword" size="large" type="password" clearable placeholder="请再次输入密码"></el-input>
      </form>
      <template #footer>
        <span class="dialog-footer">
          <el-button :loading="dialogLoading" @click="dialogClose">取消</el-button>
          <el-button type="primary" :loading="dialogLoading" @click="updatePassword">保存</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import { ref, watch } from "vue";
import { ElMessage, ElMessageBox } from "element-plus";
import SvgIcon from "@/components/svg-icon";
import Token from "@/store/token";
import TokenApi from "@/api/token";
import UserApi from "@/api/user";
import router from "@/router";
import DocCache from "@/store/doc-cache";

const hostUrl = ref(location.origin);
const name = ref(Token.getName());
const dialogVisible = ref(false);
const dialogLoading = ref(false);
const form = ref({ password: "", newPassword: "", confirmPassword: "" });
const onlyPreview = ref(true);
const isStretch = ref(true);
const isDocument = ref(router.currentRoute.value.name === "document");

watch(
  () => router.currentRoute.value.name,
  (val) => {
    if (val === "document") {
      isDocument.value = true;
    } else {
      isDocument.value = false;
    }
  }
);

/**
 * 点击退出登录
 */
const logout = () => {
  ElMessageBox.confirm("是否退出登录？", "提示", {
    confirmButtonText: "退出登录",
    cancelButtonText: "取消",
    type: "info",
  })
    .then(() => {
      DocCache.removeDoc();
      TokenApi.signOut();
      Token.removeToken();
    })
    .catch(() => {});
};

/**
 * 更新密码
 */
const updatePassword = () => {
  if (form.value.password === "" || form.value.newPassword === "" || form.value.confirmPassword === "") {
    ElMessage.warning("请填写密码");
    return;
  }
  if (form.value.newPassword !== form.value.confirmPassword) {
    ElMessage.warning("两次密码不一致");
    return;
  }

  dialogLoading.value = true;
  UserApi.updatePassword(form.value.password, form.value.newPassword)
    .then((res) => {
      ElMessage.success("修改成功");
      dialogLoading.value = false;
      dialogClose();
    })
    .catch(() => {
      dialogLoading.value = false;
    });
};

/**
 * 弹窗关闭
 */
const dialogClose = () => {
  if (dialogLoading.value) {
    return;
  }
  dialogVisible.value = false;
  form.value.password = "";
  form.value.newPassword = "";
  form.value.confirmPassword = "";
};

/**
 * 点击标题
 */
const titleClick = () => {
  if (isDocument.value) {
    onlyPreview.value = !onlyPreview.value;
  }
};

/**
 * 点击标题图标
 */
const iconClick = () => {
  if (isDocument.value) {
    isStretch.value = !isStretch.value;
  }
};

/**
 * 点击公开文档
 */
const publishClick = () => {
  let url = hostUrl.value + "/#/open/publish";
  window.open(url, "_blank");
};
</script>

<style lang="scss" scoped>
.page-layout {
  position: fixed;
  width: 100%;
  height: 100%;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
}
.top-view {
  height: 49px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  user-select: none;
  border-bottom: 1px #eee solid;
  padding: 0 2%;
  white-space: nowrap;
  .left-view {
    display: flex;
    align-items: center;
    height: 100%;
    flex: 1;
    .title-view {
      font-size: 18px;
      font-weight: bold;
      display: flex;
      align-items: center;
    }
    .menu-view {
      margin-left: 2%;
      display: flex;
      align-items: center;
      height: 100%;
      font-size: 14px;
      a {
        text-decoration: none;
        color: #303133;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        border-bottom: 2px solid transparent;
        box-sizing: border-box;
        padding: 4px 20px 0 20px;
        transition: all 0.3s;
      }
      a:hover {
        background: rgba(0, 148, 193, 0.1);
      }
      .router-link-active {
        color: #0094c1;
        border-color: #0094c1;
      }
    }
  }
  .right-view {
    height: 100%;
    .text-view {
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      padding: 0 20px;
      transition: all 0.3s;
      color: #303133;
      outline: none;
    }
    .text-view:hover {
      color: #0094c1;
    }
  }
}
.content-view {
  width: 100%;
  height: calc(100% - 50px);
  overflow: auto;
  background-color: #fcfcfc;
}
</style>