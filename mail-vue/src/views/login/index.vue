<template>
  <div class="mil-login-page">
    <!-- 背景图像容器 -->
    <div class="mil-images" v-if="settingStore.settings.background">
      <img :src="backgroundImage" alt="background" class="mil-background">
    </div>
    <!-- 如果没有背景图，使用云朵动画 -->
    <div id="background-wrap" v-else>
      <div class="x1 cloud"></div>
      <div class="x2 cloud"></div>
      <div class="x3 cloud"></div>
      <div class="x4 cloud"></div>
      <div class="x5 cloud"></div>
    </div>

    <!-- 主内容容器 -->
    <div class="container">
      <div class="mil-login-panel">
        <div class="row align-items-center">
          <!-- 登录/注册表单区域 -->
          <div class="col-12 col-lg-8 col-xl-6 mil-mb-30">
            <div class="mil-form-container">
              <!-- 标题区域 -->
              <div class="mil-header">
                <h1 class="mil-title">{{ settingStore.settings.title }}</h1>
                <div class="mil-mode-switch">
                  <el-switch
                    v-model="darkMode"
                    :active-color="'#DBA91C'"
                    :inactive-color="'#E9F1F4'"
                    @change="toggleTheme"
                    style="--el-switch-on-color: #DBA91C"
                  >
                    <template #active>
                      <Icon icon="ph:moon-fill" width="14" height="14" />
                    </template>
                    <template #inactive>
                      <Icon icon="ph:sun-fill" width="14" height="14" />
                    </template>
                  </el-switch>
                </div>
              </div>

              <!-- 表单切换标签 -->
              <div class="mil-form-tabs">
                <el-tabs v-model="activeTab" class="mil-custom-tabs">
                  <el-tab-pane :label="$t('loginBtn')" name="login">
                    <!-- 登录表单 -->
                    <div class="mil-form-group">
                      <div class="mil-input-group">
                        <Icon icon="ph:envelope-simple-fill" class="mil-input-icon" />
                        <el-input
                          v-model="form.email"
                          :placeholder="$t('emailAccount')"
                          :class="settingStore.settings.loginDomain === 0 ? 'mil-email-input' : ''"
                          @keyup.enter="submit"
                        >
                          <template #append v-if="settingStore.settings.loginDomain === 0">
                            <div @click.stop="openSelect('login')" class="mil-domain-selector">
                              <el-select
                                ref="mySelect"
                                v-model="suffix"
                                :placeholder="$t('select')"
                                class="mil-select"
                              >
                                <el-option
                                  v-for="item in domainList"
                                  :key="item"
                                  :label="item"
                                  :value="item"
                                />
                              </el-select>
                              <div class="mil-domain-display">
                                <span>{{ suffix }}</span>
                                <Icon icon="mingcute:down-small-fill" width="20" height="20" />
                              </div>
                            </div>
                          </template>
                        </el-input>
                      </div>

                      <div class="mil-input-group">
                        <Icon icon="ph:lock-key-fill" class="mil-input-icon" />
                        <el-input
                          v-model="form.password"
                          :placeholder="$t('password')"
                          type="password"
                          show-password
                          @keyup.enter="submit"
                        />
                      </div>

                      <el-button
                        class="mil-login-btn"
                        type="primary"
                        @click="submit"
                        :loading="loginLoading"
                      >
                        <Icon icon="ph:sign-in-fill" width="16" height="16" />
                        <span>{{ $t('loginBtn') }}</span>
                      </el-button>

                      <!-- 忘记密码链接 -->
                      <div class="mil-forgot-password">
                        <a href="#" class="mil-link" @click.prevent="forgotPassword">
                          {{ $t('forgotPassword') }}
                        </a>
                      </div>
                    </div>
                  </el-tab-pane>

                  <el-tab-pane :label="$t('regBtn')" name="register" v-if="settingStore.settings.register === 0">
                    <!-- 注册表单 -->
                    <div class="mil-form-group">
                      <div class="mil-input-group">
                        <Icon icon="ph:envelope-simple-fill" class="mil-input-icon" />
                        <el-input
                          v-model="registerForm.email"
                          :placeholder="$t('emailAccount')"
                          class="mil-email-input"
                        >
                          <template #append>
                            <div @click.stop="openSelect('register')" class="mil-domain-selector">
                              <el-select
                                ref="mySelectRegister"
                                v-model="suffix"
                                :placeholder="$t('select')"
                                class="mil-select"
                              >
                                <el-option
                                  v-for="item in domainList"
                                  :key="item"
                                  :label="item"
                                  :value="item"
                                />
                              </el-select>
                              <div class="mil-domain-display">
                                <span>{{ suffix }}</span>
                                <Icon icon="mingcute:down-small-fill" width="20" height="20" />
                              </div>
                            </div>
                          </template>
                        </el-input>
                      </div>

                      <div class="mil-input-group">
                        <Icon icon="ph:lock-key-fill" class="mil-input-icon" />
                        <el-input
                          v-model="registerForm.password"
                          :placeholder="$t('password')"
                          type="password"
                          show-password
                        />
                      </div>

                      <div class="mil-input-group">
                        <Icon icon="ph:lock-key-fill" class="mil-input-icon" />
                        <el-input
                          v-model="registerForm.confirmPassword"
                          :placeholder="$t('confirmPwd')"
                          type="password"
                          show-password
                        />
                      </div>

                      <!-- 注册码输入 -->
                      <div class="mil-input-group" v-if="settingStore.settings.regKey === 0 || settingStore.settings.regKey === 2">
                        <Icon icon="ph:key-fill" class="mil-input-icon" />
                        <el-input
                          v-model="registerForm.code"
                          :placeholder="settingStore.settings.regKey === 0 ? $t('regKey') : $t('regKeyOptional')"
                          type="text"
                        />
                      </div>

                      <!-- 人机验证 -->
                      <div v-show="verifyShow" class="mil-turnstile-container">
                        <div
                          class="register-turnstile"
                          :data-sitekey="settingStore.settings.siteKey"
                          data-callback="onTurnstileSuccess"
                          data-error-callback="onTurnstileError"
                        >
                          <span class="mil-error-text" v-if="botJsError">{{ $t('verifyModuleFailed') }}</span>
                        </div>
                      </div>

                      <el-button
                        class="mil-register-btn"
                        type="primary"
                        @click="submitRegister"
                        :loading="registerLoading"
                      >
                        <Icon icon="ph:user-plus-fill" width="16" height="16" />
                        <span>{{ $t('regBtn') }}</span>
                      </el-button>
                    </div>
                  </el-tab-pane>
                </el-tabs>
              </div>

              <!-- 社交登录选项（隐藏） -->
              <div class="mil-social-login">
                <div class="mil-divider">
                  <span>{{ $t('orLoginWith') }}</span>
                </div>
                <div class="mil-social-buttons">
                  <el-button class="mil-social-btn mil-google">
                    <Icon icon="logos:google-icon" width="20" height="20" />
                  </el-button>
                  <el-button class="mil-social-btn mil-github">
                    <Icon icon="logos:github-icon" width="20" height="20" />
                  </el-button>
                  <el-button class="mil-social-btn mil-microsoft">
                    <Icon icon="logos:microsoft-icon" width="20" height="20" />
                  </el-button>
                </div>
              </div>
            </div>
          </div>

          <!-- 右侧信息区域 -->
          <div class="col-12 col-lg-4 col-xl-6 mil-jce mil-l-jcs mil-mb-30 mil-info-section">
            <div class="mil-info-card">
              <div class="mil-qr-section">
                <h3 class="mil-info-title">{{ $t('mobileAccess') }}</h3>
                <div class="mil-qr">
                  <img :src="qrCode" alt="App QR Code">
                  <p class="mil-qr-desc">{{ $t('scanQRCode') }}</p>
                </div>
              </div>
              
              <div class="mil-features">
                <div class="mil-feature-item">
                  <Icon icon="ph:shield-check-fill" class="mil-feature-icon" />
                  <span>{{ $t('secureLogin') }}</span>
                </div>
                <div class="mil-feature-item">
                  <Icon icon="ph:device-mobile-fill" class="mil-feature-icon" />
                  <span>{{ $t('responsiveDesign') }}</span>
                </div>
                <div class="mil-feature-item">
                  <Icon icon="ph:rocket-launch-fill" class="mil-feature-icon" />
                  <span>{{ $t('fastPerformance') }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 对话框框架（用于忘记密码等） -->
    <div class="mil-dialog-frame" :class="{ 'mil-active': dialogVisible }">
      <div class="container">
        <div class="mil-dialog">
          <i class="fal fa-times" @click="closeDialog"></i>
          <div class="mil-scroll">
            <!-- 对话框内容根据类型动态显示 -->
            <div v-if="dialogType === 'forgot'">
              <h1 class="mil-mb-60">{{ $t('forgotPassword') }}</h1>
              <div class="mil-form-group">
                <el-input
                  v-model="resetEmail"
                  :placeholder="$t('enterEmail')"
                  class="mil-mb-30"
                  @keyup.enter="sendResetLink"
                />
                <el-button type="primary" class="mil-reset-btn" @click="sendResetLink" :loading="resetLoading">
                  {{ $t('sendResetLink') }}
                </el-button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, reactive, onMounted, onUnmounted, nextTick, watch } from 'vue';
import { useSettingStore } from "@/store/setting.js";
import { useUiStore } from "@/store/ui.js";
import { login } from "@/request/login.js";
import { register } from "@/request/login.js";
import { isEmail } from "@/utils/verify-utils.js";
import { Icon } from "@iconify/vue";
import { cvtR2Url } from "@/utils/convert.js";
import { useI18n } from "vue-i18n";
import { ElMessage } from 'element-plus';
import { loginUserInfo } from "@/request/my.js";
import { useAccountStore } from "@/store/account.js";
import { useUserStore } from "@/store/user.js";
import { permsToRouter } from "@/perm/perm.js";
import router from "@/router";

const { t } = useI18n();
const settingStore = useSettingStore();
const uiStore = useUiStore();
const accountStore = useAccountStore();
const userStore = useUserStore();

// 响应式数据
const darkMode = ref(uiStore.dark);
const activeTab = ref('login');
const loginLoading = ref(false);
const registerLoading = ref(false);
const dialogVisible = ref(false);
const dialogType = ref('');
const resetEmail = ref('');
const resetLoading = ref(false); // 忘记密码加载状态

// 表单数据
const form = reactive({
  email: '',
  password: '',
});

const registerForm = reactive({
  email: '',
  password: '',
  confirmPassword: '',
  code: null
});

// 域名相关
const mySelect = ref();
const mySelectRegister = ref();
const suffix = ref(settingStore.domainList[0] || '');
const domainList = settingStore.domainList;

// 人机验证相关
const verifyShow = ref(false);
let verifyToken = '';
let turnstileId = null;
let botJsError = ref(false);
let verifyErrorCount = 0;

// 计算属性
const backgroundImage = computed(() => {
  return settingStore.settings.background ? cvtR2Url(settingStore.settings.background) : '';
});

const qrCode = computed(() => {
  // 二维码生成逻辑（保留原逻辑，可根据实际需求替换）
  return 'https://api.qrserver.com/v1/create-qr-code/?size=150x150&data=' + 
         encodeURIComponent(window.location.origin);
});

// 监听暗模式变化，同步响应式数据
watch(
  () => uiStore.dark,
  (newVal) => {
    darkMode.value = newVal;
  },
  { immediate: true }
);

// 检查是否需要人机验证
const needsVerification = () => {
  return settingStore.settings.registerVerify === 0 || 
         (settingStore.settings.registerVerify === 2 && settingStore.settings.regVerifyOpen);
};

// 初始化人机验证
const initializeTurnstile = () => {
  try {
    if (window.turnstile && !turnstileId) {
      turnstileId = window.turnstile.render('.register-turnstile');
      botJsError.value = false;
    }
  } catch (e) {
    botJsError.value = true;
    console.error('人机验证JS加载失败:', e);
    ElMessage.error(t('verifyModuleFailed'));
  }
};

// 重置注册表单
const resetRegisterForm = () => {
  registerForm.email = '';
  registerForm.password = '';
  registerForm.confirmPassword = '';
  registerForm.code = '';
  verifyToken = '';
  verifyShow.value = false;
  verifyErrorCount = 0;
  turnstileId = null;
};

// 处理注册错误
const handleRegisterError = (error) => {
  if (error.code === 400) {
    verifyToken = '';
    settingStore.settings.regVerifyOpen = true;
    verifyShow.value = true;
    
    nextTick(() => {
      if (window.turnstile && turnstileId) {
        window.turnstile.reset(turnstileId);
      } else {
        initializeTurnstile();
      }
    });
  }
  ElMessage.error(error.message || t('regFailed'));
};

// 发送密码重置链接
const sendResetLink = async () => {
  if (!resetEmail.value) {
    ElMessage.warning(t('emptyEmailMsg'));
    return;
  }
  
  if (!isEmail(resetEmail.value)) {
    ElMessage.warning(t('notEmailMsg'));
    return;
  }
  
  resetLoading.value = true;
  try {
    // 替换为实际的重置密码接口
    // await sendResetPasswordEmail(resetEmail.value);
    ElMessage.success(t('resetLinkSent'));
    closeDialog();
  } catch (error) {
    ElMessage.error(error.message || t('resetLinkFailed'));
  } finally {
    resetLoading.value = false;
  }
};

// 方法
const toggleTheme = () => {
  uiStore.toggleDark();
};

const openSelect = (type = 'login') => {
  // 明确指定选择器类型，不依赖activeTab
  const selectRef = type === 'login' ? mySelect : mySelectRegister;
  if (selectRef.value) {
    selectRef.value.toggleMenu();
  }
};

// 登录表单验证
const validateForm = () => {
  const email = settingStore.settings.loginDomain === 0 
    ? form.email + suffix.value 
    : form.email;
  
  if (!form.email) {
    ElMessage.warning(t('emptyEmailMsg'));
    return false;
  }
  
  if (!isEmail(email)) {
    ElMessage.warning(t('notEmailMsg'));
    return false;
  }
  
  if (!form.password) {
    ElMessage.warning(t('emptyPwdMsg'));
    return false;
  }
  
  return true;
};

// 注册表单验证
const validateRegisterForm = () => {
  if (!registerForm.email) {
    ElMessage.warning(t('emptyEmailMsg'));
    return false;
  }
  
  const email = registerForm.email + suffix.value;
  if (!isEmail(email)) {
    ElMessage.warning(t('notEmailMsg'));
    return false;
  }
  
  if (!registerForm.password) {
    ElMessage.warning(t('emptyPwdMsg'));
    return false;
  }
  
  if (registerForm.password.length < 6) {
    ElMessage.warning(t('pwdLengthMsg'));
    return false;
  }
  
  if (registerForm.password !== registerForm.confirmPassword) {
    ElMessage.warning(t('confirmPwdFailMsg'));
    return false;
  }
  
  if (settingStore.settings.regKey === 0 && !registerForm.code) {
    ElMessage.warning(t('emptyRegKeyMsg'));
    return false;
  }
  
  return true;
};

const submit = async () => {
  if (!validateForm()) return;
  
  loginLoading.value = true;
  
  const email = settingStore.settings.loginDomain === 0 
    ? form.email + suffix.value 
    : form.email;
  
  try {
    const data = await login(email, form.password);
    localStorage.setItem('token', data.token);
    
    // 获取用户信息
    const user = await loginUserInfo();
    accountStore.currentAccountId = user.accountId;
    userStore.user = user;
    
    // 添加动态路由
    const routers = permsToRouter(user.permKeys);
    routers.forEach(routerData => {
      router.addRoute('layout', routerData);
    });
    
    // 跳转到主页面
    await router.replace({ name: 'layout' });
    uiStore.showNotice();
    ElMessage.success(t('loginSuccess'));
  } catch (error) {
    ElMessage.error(error.message || t('loginFailed'));
  } finally {
    loginLoading.value = false;
  }
};

const submitRegister = async () => {
  if (!validateRegisterForm()) return;
  
  // 人机验证检查
  if (!verifyToken && needsVerification()) {
    if (!verifyShow.value) {
      verifyShow.value = true;
      await nextTick(() => {
        initializeTurnstile();
      });
    } else if (!botJsError.value) {
      ElMessage.warning(t('botVerifyMsg'));
    }
    return;
  }
  
  registerLoading.value = true;
  
  const formData = {
    email: registerForm.email + suffix.value,
    password: registerForm.password,
    token: verifyToken,
    code: registerForm.code
  };
  
  try {
    const response = await register(formData);
    
    // 更新设置中的验证状态
    if (response.regVerifyOpen !== undefined) {
      settingStore.settings.regVerifyOpen = response.regVerifyOpen;
    }
    
    ElMessage.success(t('regSuccessMsg'));
    activeTab.value = 'login';
    resetRegisterForm();
  } catch (error) {
    handleRegisterError(error);
  } finally {
    registerLoading.value = false;
  }
};

const forgotPassword = () => {
  dialogType.value = 'forgot';
  dialogVisible.value = true;
};

const closeDialog = () => {
  dialogVisible.value = false;
  resetEmail.value = '';
  resetLoading.value = false;
};

// 窗口全局方法（用于人机验证）
window.onTurnstileSuccess = (token) => {
  verifyToken = token;
};

window.onTurnstileError = (error) => {
  if (verifyErrorCount >= 4) return;
  verifyErrorCount++;
  console.warn('人机验证加载失败:', error);
  
  setTimeout(() => {
    if (window.turnstile) {
      if (!turnstileId) {
        turnstileId = window.turnstile.render('.register-turnstile');
      } else {
        window.turnstile.reset(turnstileId);
      }
    }
  }, 1500);
};

onMounted(() => {
  // 初始化视口高度
  setViewportHeight();
  window.addEventListener('resize', setViewportHeight);
  
  // 初始化云朵动画（仅无背景图时）
  if (!settingStore.settings.background) {
    initCloudAnimation();
  }
});

onUnmounted(() => {
  window.removeEventListener('resize', setViewportHeight);
  // 清理全局方法，避免内存泄漏
  delete window.onTurnstileSuccess;
  delete window.onTurnstileError;
});

const setViewportHeight = () => {
  const vh = window.innerHeight * 0.01;
  document.documentElement.style.setProperty('--vh', `${vh}px`);
};

// 初始化云朵动画（添加随机位置，增强视觉效果）
const initCloudAnimation = () => {
  const clouds = document.querySelectorAll('.cloud');
  clouds.forEach(cloud => {
    // 随机设置云朵的垂直位置
    const randomTop = Math.floor(Math.random() * 80);
    cloud.style.top = `${randomTop}%`;
  });
};
</script>

<!-- 全局样式：滚动条 + 基础重置 -->
<style lang="scss">
// 全局滚动条样式
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb {
  background: #DBA91C;
  border-radius: 4px;
  transition: background 0.3s ease;
}

::-webkit-scrollbar-thumb:hover {
  background: #c49910;
}

// 基础样式重置
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}
</style>

<!-- 组件样式 -->
<style lang="scss" scoped>
.mil-login-page {
  min-height: 100vh;
  min-height: calc(var(--vh, 1vh) * 100);
  background: linear-gradient(135deg, #121212 0%, #1a1a1a 100%);
  position: relative;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.mil-images {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  
  .mil-background {
    width: 100%;
    height: 100%;
    object-fit: cover;
    object-position: center;
    filter: brightness(0.6);
    transition: filter 0.3s ease;
  }
}

#background-wrap {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  overflow: hidden;
}

.container {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  position: relative;
  z-index: 1;
}

.mil-login-panel {
  background: rgba(18, 18, 18, 0.85);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1px solid #2C2C2C;
  border-radius: 20px;
  padding: 40px;
  width: 100%;
  max-width: 1200px;
  box-shadow: 
    0 20px 40px rgba(0, 0, 0, 0.3),
    0 0 0 1px rgba(219, 169, 28, 0.1);
  transition: all 0.3s ease;
  
  &:hover {
    box-shadow: 
      0 25px 50px rgba(0, 0, 0, 0.4),
      0 0 0 1px rgba(219, 169, 28, 0.2);
  }
  
  @media (max-width: 768px) {
    padding: 30px 20px;
    border-radius: 15px;
    margin: 0 10px;
  }
}

.mil-form-container {
  .mil-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 40px;
    flex-wrap: wrap;
    gap: 20px;
    
    .mil-title {
      color: #E0E0E0;
      font-size: clamp(24px, 3vw, 32px);
      font-weight: 700;
      margin: 0;
      letter-spacing: -0.5px;
    }
    
    .mil-mode-switch {
      :deep(.el-switch) {
        .el-switch__core {
          border-color: #2C2C2C;
          transition: all 0.3s ease;
        }
        
        &:hover .el-switch__core {
          border-color: #DBA91C;
        }
      }
    }
  }
}

.mil-form-tabs {
  :deep(.mil-custom-tabs) {
    .el-tabs__header {
      border-bottom: 1px solid #2C2C2C;
      margin-bottom: 30px;
      
      .el-tabs__nav-wrap::after {
        background-color: #2C2C2C;
        height: 1px;
      }
      
      .el-tabs__item {
        color: #999;
        font-weight: 600;
        font-size: 16px;
        padding: 0 20px 15px;
        transition: all 0.3s ease;
        position: relative;
        
        &:hover {
          color: #E0E0E0;
        }
        
        &.is-active {
          color: #DBA91C;
          
          &::after {
            content: '';
            position: absolute;
            bottom: -1px;
            left: 50%;
            transform: translateX(-50%);
            width: 20px;
            height: 3px;
            background: #DBA91C;
            border-radius: 2px;
          }
        }
      }
      
      .el-tabs__active-bar {
        background-color: #DBA91C;
        height: 3px;
        border-radius: 2px;
      }
    }
  }
}

.mil-form-group {
  .mil-input-group {
    position: relative;
    margin-bottom: 24px;
    
    .mil-input-icon {
      position: absolute;
      left: 16px;
      top: 50%;
      transform: translateY(-50%);
      color: #DBA91C;
      z-index: 2;
      font-size: 18px;
      transition: color 0.3s ease;
    }
    
    :deep(.el-input) {
      .el-input__wrapper {
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid #2C2C2C;
        border-radius: 10px;
        box-shadow: none;
        padding-left: 48px;
        transition: all 0.3s ease;
        
        &:hover {
          border-color: #DBA91C;
          background: rgba(255, 255, 255, 0.08);
        }
        
        &.is-focus {
          border-color: #DBA91C;
          box-shadow: 0 0 0 1px rgba(219, 169, 28, 0.2);
          background: rgba(255, 255, 255, 0.1);
        }
        
        .el-input__inner {
          color: #E0E0E0;
          background: transparent;
          font-size: 14px;
          
          &::placeholder {
            color: #666;
            font-size: 14px;
          }
        }
      }
      
      &.mil-email-input {
        .el-input__wrapper {
          border-radius: 10px 0 0 10px;
          border-right: none;
        }
      }
    }
    
    .mil-domain-selector {
      position: relative;
      cursor: pointer;
      min-width: 100px;
      transition: all 0.3s ease;
      
      &:hover {
        background: rgba(255, 255, 255, 0.05);
        
        .mil-domain-display {
          color: #DBA91C;
        }
      }
      
      .mil-select {
        position: absolute;
        opacity: 0;
        pointer-events: none;
        width: 100%;
      }
      
      .mil-domain-display {
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 8px;
        color: #E0E0E0;
        padding: 0 12px;
        height: 100%;
        font-size: 14px;
        transition: all 0.3s ease;
        
        .iconify {
          transition: transform 0.3s ease;
        }
        
        &:hover .iconify {
          transform: translateY(1px);
        }
      }
    }
  }
}

.mil-login-btn,
.mil-register-btn,
.mil-reset-btn {
  width: 100%;
  height: 56px;
  background: linear-gradient(135deg, #DBA91C 0%, #c49910 100%);
  border: none;
  border-radius: 10px;
  color: #121212;
  font-size: 16px;
  font-weight: 600;
  margin-top: 10px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  
  &:hover {
    background: linear-gradient(135deg, #c49910 0%, #ad8810 100%);
    transform: translateY(-2px);
    box-shadow: 
      0 10px 20px rgba(219, 169, 28, 0.3),
      0 0 20px rgba(219, 169, 28, 0.1);
  }
  
  &:active {
    transform: translateY(0);
    box-shadow: 0 5px 10px rgba(219, 169, 28, 0.2);
  }
  
  &:disabled,
  &.is-loading {
    opacity: 0.6;
    cursor: not-allowed;
    transform: none;
    box-shadow: none;
  }
  
  .iconify {
    font-size: 18px;
  }
}

.mil-forgot-password {
  text-align: center;
  margin-top: 20px;
  
  .mil-link {
    color: #999;
    font-size: 14px;
    text-decoration: none;
    transition: all 0.3s ease;
    display: inline-block;
    
    &:hover {
      color: #DBA91C;
      transform: translateY(-1px);
    }
  }
}

.mil-info-section {
  @media (max-width: 992px) {
    margin-top: 40px;
    display: flex;
    justify-content: center;
  }
}

.mil-info-card {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid #2C2C2C;
  border-radius: 15px;
  padding: 30px;
  transition: all 0.3s ease;
  
  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 15px 30px rgba(0, 0, 0, 0.2);
    border-color: rgba(219, 169, 28, 0.3);
  }
  
  .mil-qr-section {
    text-align: center;
    margin-bottom: 30px;
    
    .mil-info-title {
      color: #E0E0E0;
      font-size: 20px;
      margin-bottom: 20px;
      font-weight: 600;
    }
    
    .mil-qr {
      img {
        width: 150px;
        height: 150px;
        border: 1px solid #2C2C2C;
        border-radius: 10px;
        margin-bottom: 15px;
        transition: all 0.3s ease;
        
        &:hover {
          border-color: #DBA91C;
          transform: scale(1.05);
        }
      }
      
      .mil-qr-desc {
        color: #999;
        font-size: 14px;
        margin: 0;
        line-height: 1.5;
      }
    }
  }
}

.mil-features {
  .mil-feature-item {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 15px;
    color: #E0E0E0;
    padding: 10px;
    border-radius: 8px;
    transition: all 0.3s ease;
    
    &:hover {
      background: rgba(255, 255, 255, 0.05);
      transform: translateX(5px);
    }
    
    &:last-child {
      margin-bottom: 0;
    }
    
    .mil-feature-icon {
      color: #DBA91C;
      font-size: 18px;
      min-width: 24px;
    }
    
    span {
      font-size: 14px;
      font-weight: 500;
    }
  }
}

.mil-turnstile-container {
  margin: 20px 0;
  
  .register-turnstile {
    min-height: 65px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .mil-error-text {
    font-size: 12px;
    color: #F56C6C;
    display: block;
    text-align: center;
    margin-top: 10px;
  }
}

.mil-social-login {
  margin-top: 40px;
  display: none; /* 暂时隐藏 */
  
  .mil-divider {
    display: flex;
    align-items: center;
    margin: 30px 0;
    color: #666;
    
    &::before,
    &::after {
      content: '';
      flex: 1;
      height: 1px;
      background: #2C2C2C;
    }
    
    span {
      padding: 0 15px;
      font-size: 14px;
      text-transform: uppercase;
      letter-spacing: 1px;
    }
  }
  
  .mil-social-buttons {
    display: flex;
    justify-content: center;
    gap: 15px;
    
    .mil-social-btn {
      width: 50px;
      height: 50px;
      border-radius: 50%;
      border: 1px solid #2C2C2C;
      background: rgba(255, 255, 255, 0.05);
      transition: all 0.3s ease;
      
      &:hover {
        background: rgba(255, 255, 255, 0.1);
        transform: translateY(-2px);
        border-color: #DBA91C;
      }
      
      .iconify {
        font-size: 20px;
      }
    }
  }
}

.mil-dialog-frame {
  background-color: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(40px);
  -webkit-backdrop-filter: blur(40px);
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 9999;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.4s ease-in-out;
  
  &.mil-active {
    opacity: 1;
    pointer-events: all;
  }
  
  .container {
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

.mil-dialog {
  position: relative;
  overflow: hidden;
  border: 1px solid #2C2C2C;
  height: 80vh;
  max-height: 600px;
  width: 100%;
  max-width: 500px;
  background-color: #121212;
  border-radius: 20px;
  transform: translateY(30px) scale(0.9);
  transition: transform 0.4s ease-in-out;
  
  .mil-dialog-frame.mil-active & {
    transform: translateY(0) scale(1);
  }
  
  .mil-scroll {
    overflow-y: auto;
    padding: 60px;
    height: 100%;
    width: 100%;
    
    @media (max-width: 768px) {
      padding: 40px 20px;
    }
  }
  
  .fa-times {
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 20px;
    position: absolute;
    top: 20px;
    right: 20px;
    border: 1px solid #2C2C2C;
    background-color: rgba(0, 0, 0, 0.5);
    border-radius: 50%;
    color: #E0E0E0;
    cursor: pointer;
    transition: all 0.3s ease-in-out;
    z-index: 10;
    
    &:hover {
      transform: rotate(90deg);
      color: #DBA91C;
      border-color: #DBA91C;
      background-color: rgba(0, 0, 0, 0.7);
    }
  }
}

.mil-mb-30 {
  margin-bottom: 30px;
}

.mil-mb-60 {
  margin-bottom: 60px;
}

.mil-jce {
  justify-content: end;
}

.mil-l-jcs {
  @media (max-width: 992px) {
    justify-content: start;
  }
}

// 云朵动画样式
@keyframes animateCloud {
  0% {
    margin-left: -500px;
    opacity: 0;
  }
  10% {
    opacity: 0.3;
  }
  90% {
    opacity: 0.3;
  }
  100% {
    margin-left: 100%;
    opacity: 0;
  }
}

.cloud {
  background: linear-gradient(to bottom, #fff 5%, #f1f1f1 100%);
  border-radius: 100px;
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.2);
  height: 120px;
  width: 350px;
  position: absolute;
  opacity: 0;
  
  &:after,
  &:before {
    content: "";
    position: absolute;
    background: #fff;
    z-index: -1;
  }
  
  &:after {
    border-radius: 100px;
    height: 100px;
    left: 50px;
    top: -50px;
    width: 100px;
    box-shadow: inset 0 2px 5px rgba(0, 0, 0, 0.1);
  }
  
  &:before {
    border-radius: 200px;
    height: 180px;
    width: 180px;
    right: 50px;
    top: -90px;
    box-shadow: inset 0 2px 5px rgba(0, 0, 0, 0.1);
  }
}

.x1 {
  animation: animateCloud 30s linear infinite;
  transform: scale(0.65);
  animation-delay: 0s;
}

.x2 {
  animation: animateCloud 25s linear infinite;
  transform: scale(0.5);
  animation-delay: 5s;
}

.x3 {
  animation: animateCloud 35s linear infinite;
  transform: scale(0.7);
  animation-delay: 10s;
}

.x4 {
  animation: animateCloud 20s linear infinite;
  transform: scale(0.4);
  animation-delay: 15s;
}

.x5 {
  animation: animateCloud 40s linear infinite;
  transform: scale(0.55);
  animation-delay: 20s;
}

// 响应式调整
@media (max-width: 992px) {
  .mil-login-panel {
    max-width: 700px;
  }
  
  .mil-info-card {
    max-width: 400px;
    margin: 0 auto;
  }
}

@media (max-width: 768px) {
  .mil-form-container .mil-header {
    flex-direction: column;
    text-align: center;
    gap: 15px;
  }
  
  .mil-form-tabs :deep(.mil-custom-tabs) .el-tabs__item {
    font-size: 14px;
    padding: 0 15px 10px;
  }
  
  .mil-login-btn,
  .mil-register-btn,
  .mil-reset-btn {
    height: 48px;
    font-size: 15px;
  }
  
  .mil-dialog {
    height: 70vh;
    max-height: 500px;
  }
}

@media (max-width: 480px) {
  .container {
    padding: 10px;
  }
  
  .mil-login-panel {
    padding: 20px 15px;
  }
  
  .mil-form-group .mil-input-group {
    margin-bottom: 20px;
  }
}
</style>
