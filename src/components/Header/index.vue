<template>
    <header class="min-w-full text-gray-600">
      <div class="container mx-auto flex flex-col flex-wrap items-center px-5 py-3 shadow-md md:flex-row">
        <a
          class="mb-4 flex items-center font-medium text-gray-900 md:mb-0"
          :href="metadata.website"
          target="_blank"
        >
          <img
            src="/logo.svg"
            class="h-10"
            alt="logo"
          />
          <span class="ml-3 text-xl">Open-Lineage</span>
        </a>
        <nav class="flex flex-wrap items-center justify-center space-x-2 text-base md:ml-4	md:mr-auto md:border-l md:border-gray-400 md:py-1 md:pl-4">
          <a-select
            v-model:value="selectedValue"
            style="{ width: 100 }"
            @change="handleChange"
            :options="[
              { value: 'hive', label: 'Hive' },
              { value: 'mysql', label: 'Mysql' },
            ]"
          />
          <a-button
            type="primary"
            :icon="['ant-design:rocket-outlined', '']"
            class="bg-[#1677ff]"
            :style="{ display: 'inline-flex', alignItems: 'center', justifyContent: 'center' }"
            @click="handleParseSql"
          >
            解析血缘
          </a-button>
          <a-button
            type="default"
            :icon="['ant-design:code-sandbox-outlined', '']"
            :style="{ display: 'inline-flex', alignItems: 'center', justifyContent: 'center' }"
            @click="handleChangeModel(model === 'test' ? 'default' : 'test')"
          >
            {{ model === 'test' ? '退出测试' : '开始测试' }}
          </a-button>
          <template v-if="model === 'test'">
            <a-select
              v-model:value="selectedSize"
              style="{ width: 100 }"
              @change="handleChangeSize"
              :options="[
                { value: '100', label: '100节点' },
                { value: '500', label: '500节点' },
                { value: '1000', label: '1000节点' },
              ]"
            />
          </template>
        </nav>
        <Setting
          v-model:open="open"
        >
          <template #children>
            <a-form label-col="{ span: 12 }" wrapper-col="{ span: 12 }" style="{ width: 240 }">
              <a-form-item label="设置代码主题颜色" name="codeTheme">
                <a-select
                  style="{ width: 120 }"
                  v-model:value="theme"
                  @change="(value) => setTheme(value)"
                  :defaultValue="theme"
                  :options="[
                    { value: 'vs-light', label: 'light' },
                    { value: 'vs-dark', label: 'dark' },
                  ]"
                />
              </a-form-item>
              <a-form-item label="支持设置水印文字：" name="waterMaker">
                <a-input
                  v-model:value="textWaterMarker"
                  @input="(e) => setTextWaterMarker(e.target.value)"
                />
              </a-form-item>
              <a-form-item label="选择线条高亮颜色" name="highlight">
                <ColorPicker
                  :defaultColor="highlightColor"
                  @change="(value: string) => setHighlightColor(value)"
                />
              </a-form-item>
            </a-form>
          </template>
        </Setting>
        <div class="ml-2 hidden items-center rounded-md shadow-sm ring-1 ring-gray-900/5 lg:flex">
          <HeaderButton
            :isActive="layout === 'vertical'"
            label="Switch to vertical split layout"
            @click="setLayout('vertical')"
          >
            <path
              d="M12 3h9a2 2 0 0 1 2 2v12a2 2 0 0 1-2 2h-9"
              fill="none"
            />
            <path d="M3 17V5a2 2 0 0 1 2-2h7a1 1 0 0 1 1 1v14a1 1 0 0 1-1 1H5a2 2 0 0 1-2-2Z" />
          </HeaderButton>
          <HeaderButton
            :isActive="layout === 'editor'"
            label="Switch to preview-only layout"
            @click="setLayout('editor')"
          >
            <path
              fill="none"
              d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"
            />
          </HeaderButton>
          <HeaderButton
            :isActive="layout === 'preview'"
            label="Switch to preview-only layout"
            @click="setLayout('preview')"
          >
            <path
              d="M23 17V5a2 2 0 0 0-2-2H5a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2Z"
              fill="none"
            />
          </HeaderButton>
        </div>
      </div>
    </header>
  </template>
  
  <script lang="ts" setup>
  import { ref, computed } from 'vue';
  import metadata from '../../config/metadata';
  import Setting from "./setting.vue";
  import HeaderButton from './headerButton.vue';
  import {
    Button,
    Form,
    Input,
    Popover,
    Select,
  } from 'ant-design-vue';
  import ColorPicker from '../ColorPicker';
  
  interface HeaderProps {
    /** 模式 */
    model: string;
    /** 编辑器主题色 */
    theme: string;
    /** 布局方式 */
    layout: string;
    /** 水印文字 */
    textWaterMarker: string;
    /** 高亮颜色 */
    highlightColor: string;
    /** 解析 sql */
    handleParseSql: () => void;
    /** 设置编辑器主题 */
    setTheme: (value: string) => void;
    /** 设置布局方式 */
    setLayout: (value: string) => void;
    /** 设置节点数量 */
    setNodeSize: (value: number) => void;
    /** 设置水印文字 */
    setTextWaterMarker: (text: string) => void;
    /** 设置线条高亮色 */
    setHighlightColor: (color: string) => void;
    /** 切换模式 */
    handleChangeModel: (model: string) => void;
  }
  
  const props = defineProps<HeaderProps>();
  
  const open = ref(false);
  const selectedValue = ref('');
  const selectedSize = ref('');
  const theme = computed(() => props.theme);
  const textWaterMarker = computed(() => props.textWaterMarker);
  const highlightColor = computed(() => props.highlightColor);
  
  const handleChange = (value: string) => {
    console.log(`selected ${value}`);
  };
  
  const handleChangeSize = (value: string) => {
    props.setNodeSize(Number(value));
  };
  
  const handleParseSql = () => {
    props.handleParseSql();
  };
  
  const handleChangeModel = (model: string) => {
    props.handleChangeModel(model);
  };
  
  const setTheme = (value: string) => {
    props.setTheme(value);
  };
  
  const setTextWaterMarker = (text: string) => {
    props.setTextWaterMarker(text);
  };
  
  const setHighlightColor = (color: string) => {
    props.setHighlightColor(color);
  };
  </script>