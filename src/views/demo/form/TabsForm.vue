<!--
 * @FileDescription: 日常巡查/巡检日历页面
 * @Author: ufooz
 * @Date: 2022/4/27
 * @LastEditors: ufooz
 * @LastEditTime: 最后更新时间
 -->
<template>
  <PageWrapper title="巡检日历" v-loading="loading">
    <div class="mb-4">
      <a-button @click="addPane" class="mr-2"> Add </a-button>
      <a-button @click="handleReset" class="mr-2"> 重置表单 </a-button>
      <a-button @click="handleSetValues" class="mr-2"> 设置默认值 </a-button>
      <a-button @click="handleSubmit" class="mr-2" type="primary"> 提交表单 </a-button>
    </div>
    <CollapseContainer title="标签页+多级field表单">
      <Tabs v-model:activeKey="activeKey" hide-add type="editable-card" @edit="onEdit">
        <TabPane
          v-for="item in tabsFormSchema"
          :key="item.key"
          v-bind="omit(item, ['Form', 'key'])"
        >
          <BasicForm @register="item.Form[0]" />
        </TabPane>
      </Tabs>
    </CollapseContainer>
  </PageWrapper>
</template>

<script setup>
  import { ref } from 'vue';
  import { Tabs, TabPane } from 'ant-design-vue';
  import { PageWrapper } from '/src/components/Page';
  import { CollapseContainer } from '/src/components/Container';
  import { useMessage } from '/src/hooks/web/useMessage';
  import { omit } from 'lodash-es';
  import { deepMerge } from '/src/utils';
  import { BasicForm, useForm } from '/src/components/Form';

  const { createMessage } = useMessage();
  const activeKey = ref('tabs2');
  const loading = ref(false);
  const tabsFormSchema = [];

  // 公共属性
  const baseFormConfig = {
    showActionButtonGroup: false,
    labelWidth: 100,
  };

  // 为每个字段模拟默认值, { tabs1: { field1: '', field2: '' }, tabs2: { field1: '' }, ... }
  const mockDefaultValue = {};

  // 模拟5个标签页
  for (let i = 1; i <= 5; ++i) {
    const tabsKey = `tabs${i}`;

    // 每个标签页8个字段
    const schemas = [];
    const row = {};

    for (let j = 1; j <= 8; ++j) {
      schemas.push({
        field: `${tabsKey}.field${j}`,
        label: `${tabsKey}-field${j}`,
        component: 'Input',
        colProps: { span: 24 },
      });
      row[`field${j}`] = `field: ${tabsKey}.field${j}, default value`;
    }

    mockDefaultValue[tabsKey] = row;

    tabsFormSchema.push({
      key: tabsKey,
      tab: tabsKey,
      forceRender: true,
      Form: useForm(Object.assign({ schemas }, baseFormConfig)),
    });
  }

  async function handleReset() {
    for (const item of tabsFormSchema) {
      const { resetFields } = item.Form[1];
      await resetFields();
    }
  }

  async function handleSubmit() {
    let lastKey = '';
    loading.value = true;
    try {
      const values = {};
      for (const item of tabsFormSchema) {
        lastKey = item.key;
        const { validate, getFieldsValue } = item.Form[1];
        await validate();
        // 表单已支持多级key
        deepMerge(values, getFieldsValue());
      }

      console.log('submit values: ', values);
      createMessage.success('提交成功！请打开控制台查看');
    } catch (e) {
      // 验证失败或出错，切换到对应标签页
      activeKey.value = lastKey;
      console.log(e);
    } finally {
      loading.value = false;
    }
  }

  async function handleSetValues() {
    console.log('默认值为: ', mockDefaultValue);
    for (const item of tabsFormSchema) {
      const { setFieldsValue } = item.Form[1];
      await setFieldsValue(mockDefaultValue);
    }
  }
  
  // 添加新的pane
  // const activeKey = ref(panes.value[0].key);
  const newTabIndex = ref(0);

  const addPane = () => {
    activeKey.value = `newTab${newTabIndex.value++}`;
    const schemas1 = [];
    // const row1 = {};

    for (let j = 1; j <= 8; ++j) {
      schemas1.push({
        field: `newTab.field${j}`,
        label: `newTab-field${j}`,
        component: 'Input',
        colProps: { span: 24 },
      });
      // row1[`field${j}`] = `field: ${activeKey.value}.field${j}, default value`;
    }
    tabsFormSchema.push({
      key: activeKey.value,
      tab: activeKey.value,
      forceRender: true,
      Form: useForm(Object.assign({ schemas1 }, baseFormConfig)),
    });
  };

  const remove = (targetKey) => {
    // let lastIndex = 0;
    // panes.value.forEach((pane, i) => {
    //   if (pane.key === targetKey) {
    //     lastIndex = i - 1;
    //   }
    // });
    // panes.value = panes.value.filter((pane) => pane.key !== targetKey);
    //
    // if (panes.value.length && activeKey.value === targetKey) {
    //   if (lastIndex >= 0) {
    //     activeKey.value = panes.value[lastIndex].key;
    //   } else {
    //     activeKey.value = panes.value[0].key;
    //   }
    // }
  };

  const onEdit = (targetKey) => {
    remove(targetKey);
  };
</script>

<style scoped></style>
