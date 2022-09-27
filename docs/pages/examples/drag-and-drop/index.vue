<template lang="html">
  <div class="examples-validation page">
    <div class="content">
      <section>
        <h1>Drag and Drop</h1>
        <vue-tags-input
          v-model="tag"
          :tags="tags"
          :validation="validation"
          @tags-changed="newTags => tags = newTags"
        />
        <div class="data">
          <span>tag</span>
          <el-code :code="tagString" />
          <span>tags</span>
          <el-code :code="tagsString" />
        </div>
        <el-code lang="html" :code="require('./example1.demo.html')" />
        <el-code :code="require('./example1.demo.js')" />
      </section>
    </div>
  </div>
</template>

<script>
import VueTagsInput from '@johmun/vue-tags-input';
import ElCode from '@components/el-code';
import BreakingChanges from '@components/breaking-changes';

export default {
  name: 'ExamplesValidation',
  components: {
    VueTagsInput,
    ElCode,
    BreakingChanges,
  },
  data() {
    return {
      tag: '',
      tags: [],
      validation: [{
        classes: 'min-length',
        rule: tag => tag.text.length < 8,
      }, {
        classes: 'no-numbers',
        rule: '^([^0-9]*)$',
      }, {
        classes: 'avoid-item',
        rule: /^(?!Cannot).*$/,
        disableAdd: true,
      }, {
        classes: 'no-braces',
        rule: ({ text }) => text.indexOf('{') !== -1 || text.indexOf('}') !== -1,
      }],
    };
  },
  computed: {
    tagString() {
      return JSON.stringify(this.tag);
    },
    tagsString() {
      return JSON.stringify(this.tags);
    },
  },
};
</script>

<style lang="scss" scoped>
.data {
  margin-top: 20px;
}
</style>
