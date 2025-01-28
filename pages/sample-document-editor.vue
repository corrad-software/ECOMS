<template>
  <div class="sample-document-editor">
    <h1>Sample Document Editor</h1>
    <DocumentEditor :src="documentSrc" />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import DocumentEditor from 'vue-document-editor';

const documentSrc = ref('/documents/sample-document.pdf');
</script>

<style scoped>
.sample-document-editor {
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h1 {
  font-size: 2rem;
  margin-bottom: 1rem;
}
</style>
