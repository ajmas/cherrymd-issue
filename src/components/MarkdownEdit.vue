<template>
  <q-card class="markdown-edit" outlined>
    <q-toolbar dense>
      <q-tabs v-model="mode" align="left" dense >
        <q-tab name="edit" :label="$t('label.write')"/>
        <q-tab name="preview" :label="$t('label.preview')"/>
      </q-tabs>
      <q-space />
      <q-toolbar v-if="mode === 'edit'">
        <q-space />
        <q-btn v-for="button in buttons" :key="button.icon" flat round dense @click="onButton(button.action as string)">
          <q-icon :name="button.icon" />
        </q-btn>
      </q-toolbar>
    </q-toolbar>

    <div
      :id="editorId"
      :ref="editorId"
    >
    </div>

    <q-toolbar>
      <q-space />
      <div
        v-if="maxChars && mode === 'edit'"
        :class="{ charover: (content || '').length > maxChars }"
        class="charcount q-pa-sm"
      >
        {{ content.length }} / {{ maxChars }}
      </div>
    </q-toolbar>
  </q-card>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import 'cherry-markdown/dist/cherry-markdown.css';
import Cherry from 'cherry-markdown';

export default defineComponent({
  props: {
    value: {
      type: String,
      default: ''
    },
    maxChars: {
      type: Number,
      default: undefined
    }
  },
  setup (props) {
    return {
      editorId: 'mdEditor',
      editor: ref<Cherry | undefined>(),
      content: ref<string>(props.value),
      previewMode: false,
      language: 'en-US',
      mode: ref<string>('edit'),
      buttons: [{
        icon: 'format_heading'
      }, {
        icon: 'format_bold',
        action: 'bold'
      }, {
        icon: 'format_italic',
        action: 'italic'
      }, {
        icon: 'strikethrough_s',
        action: 'strikethrough'
      }, {
        icon: 'link',
        action: 'link'
      }, {
        icon: 'code',
        action: 'code'
      }, {
        icon: 'format_list_bulleted',
        action: 'ul'
      }, {
        icon: 'format_list_numbered',
        action: 'ol'
      }]
    };
  },
  watch: {
    value (newValue: string | undefined) {
      this.content = (newValue || '');
      if (this.maxChars) {
        this.content = this.content.substring(0, this.maxChars);
      }
      if (this.editor) {
        if (this.content !== this.editor.getValue()) {
          this.editor?.setValue(this.content, true);
        }
      }
    },
    mode () {
      if (this.mode === 'edit') {
        this.editor?.switchModel('editOnly', false);
      } else {
        this.editor?.switchModel('previewOnly', false);
      }
    }
  },
  async mounted () {
    await this.$nextTick();
    const editorElem = this.$refs[this.editorId] as HTMLElement;
    if (editorElem) {
      this.editor = new Cherry({
        el: editorElem,
        value: this.content,
        locale: 'en_US',
        editor: {
          defaultModel: 'editOnly',
          height: '200px'
        },
        toolbars: {
          theme: 'light',
          showToolbar: false,
          toolbar: [
            'bold',
            'italic',
            'strikethrough',
            '|',
            'header',
            '|',
            'list',
            {
              insert: [
                'link',
                'code',
                'table',
                'line-table',
                'bar-table'
              ]
            }
          ],
          bubble: ['bold', 'italic', 'underline', 'strikethrough', 'sub', 'sup', 'quote']
        }
      });

      if (this.editor?.options?.callback) {
        this.editor.options.callback.afterChange = this.onChange.bind(this);
      }
    }
  },
  methods: {
    onChange (text: string) {
      this.content = text;
      this.$emit('update:value', this.content);
    },
    onButton (action: string) {
      const editor = this.editor; // this.$refs.mdEditor as any;
      if (editor?.toolbar?.toolbarHandlers) {
        const toolbarHandlers = editor.toolbar.toolbarHandlers;
        const handler = toolbarHandlers[action];
        if (handler) {
          handler();
        } else if (action === 'ol' && toolbarHandlers.list) {
          toolbarHandlers.list(1);
        } else if (action === 'ul' && toolbarHandlers.list) {
          toolbarHandlers.list(2);
        } else if (action === 'link' && toolbarHandlers.insert) {
          toolbarHandlers.insert('link');
        }
      }
    }
  }
});
</script>

<style lang="scss" scoped>
.charcount {
  justify-content: right;
  &.charover {
    color: red;
  }
}
</style>
