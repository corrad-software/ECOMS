<template>
  <div class="flex flex-col md:flex-row h-screen">
      <div class="w-full md:w-1/2 p-4 flex flex-col h-full">
          <!-- Toolbar -->
          <div class="toolbar mb-4 flex flex-wrap justify-between bg-white p-2 rounded-lg shadow-sm">
              <button class="flex items-center space-x-2 px-4 py-2 text-gray-700 bg-gray-100 rounded-lg hover:bg-gray-200 transition">
                  <Icon name="mdi:scanner" size="1.25rem"></Icon>
                  <span>Scan</span>
              </button>
              <div class="flex space-x-2">
                  <button class="flex items-center space-x-2 px-4 py-2 text-gray-700 bg-gray-100 rounded-lg hover:bg-gray-200 transition" @click="sortDocuments">
                      <Icon name="mdi:sort" size="1.25rem"></Icon>
                      <span>Sort</span>
                  </button>
                  <button class="flex items-center space-x-2 px-4 py-2 text-gray-700 bg-gray-100 rounded-lg hover:bg-gray-200 transition" @click="toggleView">
                      <Icon :name="isGridView ? 'mdi:view-list' : 'mdi:view-grid'" size="1.25rem"></Icon>
                      <span>{{ isGridView ? 'List View' : 'Grid View' }}</span>
                  </button>
              </div>
          </div>
          <!-- Drag and Drop Area -->
          <div class=" bg-white rounded-lg shadow-sm  p-4">
            <FormKit
              type="file"
              name="documents"
              label="Drag and drop files here or click to import into document tray"
              help="Supported formats: PDF, DOC, DOCX, XLS, XLSX"
              @input="handleFileInput"
              class="drag-drop-area border-dashed border-2 border-gray-300 p-4 rounded-lg bg-gray-50 flex flex-col items-center justify-center h-32"
          />
          </div>
          <div class="flex justify-between items-center my-2 p-2 bg-white rounded-md">
                  <h3 class="text-lg font-semibold">Document Tray</h3>
                  <input
                      type="text"
                      v-model="searchQuery"
                      placeholder="Search documents..."
                      class="p-2 border border-gray-300 rounded-lg"
                  />
              </div>
          <div class="document-list p-4 bg-white rounded-lg shadow-sm flex-grow overflow-y-auto h-5/6">
              
              <ul :class="{'grid grid-cols-2 gap-4': isGridView, 'list-none': !isGridView}">
                  <li v-for="doc in filteredDocuments" :key="doc.url" class="mb-2" :class="{'flex flex-col items-center': isGridView, 'text-left': !isGridView}" draggable="true" @dragstart="handleDragStartDocument(doc)">
                      <button @click="selectDocument(doc.url)" class="bg-white border-b-2 px-2 py-4 rounded-md w-full flex justify-between items-center hover:bg-gray-100 transition">
                          <div v-if="isGridView" class="flex flex-col items-center justify-center w-full">
                              <Icon :name="getFileIcon(doc.url)" size="3rem"></Icon>
                              <span class="mt-2 text-center">{{ doc.name }}</span>
                          </div>
                          <div v-else class="flex items-center justify-between w-full">
                              <div class="flex items-center space-x-4">
                                  <Icon :name="getFileIcon(doc.url)" size="1.25rem"></Icon>
                                  <span>{{ doc.name }}</span>
                              </div>
                              <span class="text-sm text-gray-500">{{ doc.size }} KB</span>
                          </div>
                      </button>
                  </li>
              </ul>
          </div>
      </div>
      <!-- Document Viewer Section -->
      <div class="document-viewer flex w-full md:w-1/2 h-full">
          <div class="w-1/3 p-4 bg-white rounded-lg shadow-sm mr-2">
              <div class="flex justify-between items-center mb-4">
                  <span class="text-xs font-semibold">Cabinet</span>
                  <button class="flex items-center space-x-2 px-2 py-1 text-gray-700 bg-gray-100 rounded-lg hover:bg-gray-200 transition">
                      <Icon name="mdi:plus" size="1.25rem"></Icon>
                  </button>
              </div>
              <div class="flex items-center mb-4">
                  <select v-model="selectedCabinet" class="w-full p-1 border border-gray-300 rounded-sm">
                      <option v-for="cabinet in cabinets" :key="cabinet" :value="cabinet" class="text-sm">{{ cabinet }}</option>
                  </select>
              </div>
              <div class="cabinet-list overflow-y-auto h-5/6">
                  <ul class="grid grid-cols-1 gap-4">
                      <li v-for="cabinetDoc in cabinetDocuments" :key="cabinetDoc.url" class="mb-2 flex flex-col items-center" draggable="true" @dragstart="handleDragStartDocument(cabinetDoc)">
                          <button @click="selectDocument(cabinetDoc.url)" class="bg-white border-b-2 px-2 py-4 rounded-md w-full flex flex-col items-center hover:bg-gray-100 transition">
                              <Icon :name="getFileIcon(cabinetDoc.url)" size="3rem"></Icon>
                              <span class="mt-2 text-center">{{ cabinetDoc.name }}</span>
                          </button>
                      </li>
                  </ul>
              </div>
          </div>
          <div class="w-2/3 border border-gray-300 rounded-lg bg-gray-100">
              <iframe :src="googleDocsViewerUrl" width="100%" class="h-full"></iframe>
          </div>
      </div>
  </div>
</template>

<script>
import { FormKit } from '@formkit/vue';

export default {
  components: {
    FormKit
  },
  data() {
      return {
          documentUrl: 'https://www.adobe.com/support/products/enterprise/knowledgecenter/media/c4611_sample_explain.pdf',
          documents: [
              { name: 'Road Construction Plan', url: 'https://www.adobe.com/support/products/enterprise/knowledgecenter/media/c4611_sample_explain.pdf', size: 512 },
              { name: 'Site Inspection Report', url: 'https://www.example.com/sample2.pdf', size: 211 },
              { name: 'Equipment Maintenance Log', url: 'https://www.example.com/sample3.pdf', size: 112 },
              { name: 'Safety Protocols', url: 'https://www.example.com/sample4.pdf', size: 343 },
              { name: 'Project Timeline', url: 'https://www.example.com/sample5.pdf', size: 332 },
              { name: 'Material Specifications', url: 'https://www.example.com/sample6.pdf', size: 878 },
              { name: 'Contract Agreement', url: 'https://www.example.com/sample7.pdf', size: 78 },
              { name: 'Budget Report', url: 'https://www.example.com/sample8.pdf', size: 66 },
              { name: 'Progress Photos', url: 'https://www.example.com/sample9.pdf', size: 389 }
          ], // List of documents with sizes in KB
          isGridView: false, // Toggle between grid and list view
          sortOrder: 'asc', // Track the current sort order
          searchQuery: '', // Add search query to data
          selectedCabinet: '', // Track the selected cabinet
          cabinets: ['Cabinet 1', 'Cabinet 2', 'Cabinet 3'], // List of cabinet names
          cabinetDocuments: [
              { name: 'Cabinet Document 1', url: 'https://www.example.com/cabinet1.pdf', size: 123 },
              { name: 'Cabinet Document 2', url: 'https://www.example.com/cabinet2.pdf', size: 456 },
              { name: 'Cabinet Document 3', url: 'https://www.example.com/cabinet3.pdf', size: 789 },
              { name: 'Cabinet Document 4', url: 'https://www.example.com/cabinet1.pdf', size: 123 },
              { name: 'Cabinet Document 5', url: 'https://www.example.com/cabinet2.pdf', size: 456 },
              { name: 'Cabinet Document 6', url: 'https://www.example.com/cabinet3.pdf', size: 789 },
          ] // List of documents in the selected cabinet
      };
  },
  computed: {
      googleDocsViewerUrl() {
          return `https://docs.google.com/gview?url=${this.documentUrl}&embedded=true`;
      },
      sortedDocuments() {
          return [...this.documents].sort((a, b) => {
              if (this.sortOrder === 'asc') {
                  return a.name.localeCompare(b.name);
              } else {
                  return b.name.localeCompare(a.name);
              }
          });
      },
      filteredDocuments() {
          return this.sortedDocuments.filter(doc => 
              doc.name.toLowerCase().includes(this.searchQuery.toLowerCase())
          );
      }
  },
  methods: {
      downloadDocument() {
          const link = document.createElement('a');
          link.href = this.documentUrl;
          link.download = 'document.pdf'; // Replace with actual document name
          link.click();
      },
      updateDocumentVersion() {
          this.documentVersion = this.selectedVersion;
          // Update documentUrl based on the selected version if needed
      },
      handleDrop(event) {
          const file = event.dataTransfer.files[0];
          if (file) {
              const reader = new FileReader();
              reader.onload = (e) => {
                  this.documentUrl = e.target.result;
              };
              reader.readAsDataURL(file);
          }
      },
      handleDragStartDocument(doc) {
          const file = new Blob([doc.name], { type: 'text/plain' });
          const event = new DragEvent('dragstart', {
              dataTransfer: new DataTransfer()
          });
          event.dataTransfer.items.add(file);
          this.handleDrop(event);
      },
      selectDocument(url) {
          this.documentUrl = url;
      },
      toggleView() {
          this.isGridView = !this.isGridView;
      },
      getFileIcon(url) {
          const extension = url.split('.').pop();
          switch (extension) {
              case 'pdf':
                  return 'mdi:file-pdf';
              case 'doc':
              case 'docx':
                  return 'mdi:file-word';
              case 'xls':
              case 'xlsx':
                  return 'mdi:file-excel';
              default:
                  return 'mdi:file';
          }
      },
      getFileExtension(url) {
          return url.split('.').pop();
      },
      sortDocuments() {
          this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
      },
      handleFileInput(files) {
          const file = files[0];
          if (file) {
              const reader = new FileReader();
              reader.onload = (e) => {
                  this.documentUrl = e.target.result;
              };
              reader.readAsDataURL(file);
          }
      }
  },
};

definePageMeta({
  title: "Document Viewer",
  layout: "default",
  middleware: ["auth"], // Ensure only authenticated users can access
});
</script>

<style scoped>
/* Add your scoped styles here */
border: 1px solid #ccc; /* Add the missing semicolon here */
border-radius: 4px;
/* Add styles for grid view */
.grid-cols-2 {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1rem;
}
</style>
