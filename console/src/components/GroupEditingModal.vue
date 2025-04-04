<script lang="ts" setup>
import { VButton, VModal, VSpace } from "@halo-dev/components";
import { v4 as uuid } from "uuid";
import { computed, ref, watch } from "vue";
import apiClient from "@/utils/api-client";
import cloneDeep from "lodash.clonedeep";
import { useMagicKeys } from "@vueuse/core";
import type { LinkGroup } from "@/types";

const props = withDefaults(
  defineProps<{
    visible: boolean;
    group: LinkGroup | null;
  }>(),
  {
    visible: false,
    group: null,
  }
);

const emit = defineEmits<{
  (event: "update:visible", visible: boolean): void;
  (event: "close"): void;
}>();

const initialFormState: LinkGroup = {
  apiVersion: "core.halo.run/v1alpha1",
  kind: "LinkGroup",
  metadata: {
    name: uuid(),
  },
  spec: {
    displayName: "",
    priority: 0,
    links: [],
  },
};

const formState = ref<LinkGroup>(initialFormState);
const saving = ref(false);

const isUpdateMode = computed(() => {
  return !!formState.value.metadata.creationTimestamp;
});

const handleCreateGroup = async () => {
  try {
    saving.value = true;
    if (isUpdateMode.value) {
      await apiClient.put(
        `/apis/core.halo.run/v1alpha1/linkgroups/${formState.value.metadata.name}`,
        formState.value
      );
    } else {
      await apiClient.post(
        "/apis/core.halo.run/v1alpha1/linkgroups",
        formState.value
      );
    }
    onVisibleChange(false);
  } catch (e) {
    console.error("Failed to create link group", e);
  } finally {
    saving.value = false;
  }
};

const onVisibleChange = (visible: boolean) => {
  emit("update:visible", visible);
  if (!visible) {
    emit("close");
  }
};

watch(props, (newVal) => {
  const { Command_Enter } = useMagicKeys();
  let keyboardWatcher;
  if (newVal.visible) {
    keyboardWatcher = watch(Command_Enter, (v) => {
      if (v) {
        // TODO
      }
    });
  } else {
    keyboardWatcher?.unwatch();
  }

  if (newVal.visible && props.group) {
    formState.value = cloneDeep(props.group);
    return;
  }
  formState.value = cloneDeep(initialFormState);
});
</script>
<template>
  <VModal
    :visible="visible"
    :width="500"
    title="编辑分组"
    @update:visible="onVisibleChange"
  >
    <FormKit
      id="link-group-form"
      :classes="{ form: 'w-full' }"
      type="form"
      @submit="handleCreateGroup"
    >
      <FormKit
        v-model="formState.spec.displayName"
        help="可根据此名称查询链接"
        label="分组名称"
        type="text"
        validation="required"
      ></FormKit>
    </FormKit>
    <template #footer>
      <VSpace>
        <VButton type="secondary" @click="$formkit.submit('link-group-form')">
          提交 ⌘ + ↵
        </VButton>
        <VButton @click="onVisibleChange(false)">取消 Esc</VButton>
      </VSpace>
    </template>
  </VModal>
</template>
