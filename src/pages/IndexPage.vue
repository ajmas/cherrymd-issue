<template>
  <q-page class="row items-center justify-evenly">
    <MarkdownEdit
      :content="content"
      class="markdown-edit"
    />
  </q-page>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import MarkdownEdit from 'components/MarkdownEdit.vue';

export default defineComponent({
  name: 'IndexPage',

  components: {
    MarkdownEdit
  },

  setup () {
    return {
      content: ref<string>('')
    };
  }
});
</script>

<style lang="scss" scoped>
.markdown-edit {
  width: 700px;
}
</style>
