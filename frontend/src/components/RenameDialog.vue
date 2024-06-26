<template>
  <Dialog v-model="open" :options="{ title: 'Rename', size: 'xs' }">
    <template #body-content>
      <div class="flex items-center justify-center">
        <Input
          ref="input"
          class="w-full"
          v-model="newName"
          type="text"
          @keyup.enter="$resources.rename.submit" />
        <Badge
          v-if="entity.file_ext"
          :variant="'subtle'"
          theme="gray"
          size="sm"
          class="ml-2"
          :label="entity.file_ext"></Badge>
      </div>
      <ErrorMessage class="mt-2" :message="errorMessage" />
      <div class="flex mt-8">
        <Button
          variant="solid"
          class="w-full"
          :loading="$resources.rename.loading"
          @click="$resources.rename.submit">
          Rename
        </Button>
      </div>
    </template>
  </Dialog>
</template>

<script>
import { ref } from "vue";
import { Dialog, Input, ErrorMessage, Badge } from "frappe-ui";
import { useFocus } from "@vueuse/core";
import { formatSize, formatDate } from "@/utils/format";

export default {
  name: "RenameDialog",
  components: {
    Dialog,
    Input,
    ErrorMessage,
    Badge,
  },
  setup() {
    const input = ref();
    const { focused } = useFocus(input, { initialValue: true });
    return {
      input,
      focused,
    };
  },
  props: {
    modelValue: {
      type: Boolean,
      required: true,
    },
    entity: {
      type: Object,
      required: false,
      default: null,
    },
  },
  emits: ["update:modelValue", "success"],
  data() {
    return {
      newName: "",
      errorMessage: "",
      extension: "",
    };
  },
  computed: {
    entityName() {
      return this.entity?.name;
    },
    fullName() {
      if (this.entity?.file_ext) {
        return this.newName + this.entity.file_ext;
      } else {
        return this.newName;
      }
    },
    open: {
      get() {
        return this.modelValue;
      },
      set(value) {
        this.$emit("update:modelValue", value);
        if (!value) {
          this.newName = "";
          this.errorMessage = "";
        }
      },
    },
  },
  resources: {
    rename() {
      return {
        url: "drive.api.files.call_controller_method",
        method: "POST",
        params: {
          method: "rename",
          entity_name: this.entityName,
          new_title: this.fullName.trim(),
        },
        onSuccess(data) {
          this.$store.state.entityInfo[0].title = data.title;
          this.$emit("success", data);
          this.newName = "";
          this.extension = "";
          this.errorMessage = "";
        },
        onError(error) {
          if (error.messages) {
            this.errorMessage = error.messages.join("\n");
          } else {
            this.errorMessage = error.message;
          }
        },
      };
    },
  },
  created() {
    let parsedName = "";
    if (this.entity?.is_group || this.entity?.document) {
      this.newName = this.entity.title;
    } else {
      parsedName = this.entity?.title.split(".").slice(0, -1).join(".");
    }
    this.newName = parsedName?.length > 1 ? parsedName : this.entity?.title;
  },
};
</script>
