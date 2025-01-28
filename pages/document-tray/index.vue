<template>
  <div class="flex flex-col md:flex-row h-screen">
    <div class="w-full md:w-2/5 p-4 flex flex-col bg-white mr-2 rounded-lg h-5/6">
      <!-- Combined Document Tray, Toolbar, and File List -->
      <div class="flex flex-col h-full">
        <!-- Document Tray Title, Searchbar, and File Input -->
        <div class="flex flex-col my-2">
          <div class="flex justify-between items-center mb-2">
            <h3 class="text-lg font-semibold ml-4">Document Tray</h3>
            <input
              type="text"
              v-model="searchQuery"
              placeholder="Search documents..."
              class="p-2 border border-gray-300 rounded-lg mr-4"
            />
          </div>
        </div>
        <!-- Toolbar -->
        <div class="toolbar mb-2 flex flex-wrap justify-between border-2 rounded-md mx-4 py-2">
          <div class="flex space-x-2">
            <button class="flex items-center space-x-2 px-2 py-1 text-gray-700 bg-gray-100 rounded-md hover:bg-gray-200 transition" @click="toggleNewFormModal">
              <Icon name="material-symbols:add" size="1.25rem"></Icon>
              <span>Add New Document</span>
            </button>
          </div>
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
        <!-- Document List -->
        <div class="document-list p-4 flex-grow overflow-y-auto">
          <ul :class="{'grid grid-cols-2 gap-4': isGridView, 'list-none': !isGridView}" class="bg-gray-100 rounded-md py-4 px-1 h-full">
            <li v-for="doc in filteredDocuments" :key="doc.url" :class="{'flex flex-col items-center': isGridView, 'text-left': !isGridView, 'bg-gray-300': doc.url === documentUrl}" draggable="true" @dragstart="handleDragStartDocument(doc)">
              <button @click="selectDocument(doc.url)" class="py-2 px-2 w-full flex justify-between items-center hover:bg-gray-100 transition">
                <div v-if="isGridView" class="flex flex-col items-center justify-center w-full">
                  <Icon :name="getFileIcon(doc.url)" size="3rem"></Icon>
                  <span class="mt-2 text-center">{{ doc.name }}</span>
                </div>
                <div v-else class="flex items-center justify-between w-full">
                  <div class="flex items-center space-x-4">
                    <Icon :name="getFileIcon(doc.url)" size="1.25rem"></Icon>
                    <span>{{ doc.name }}</span>
                  </div>
                  <div class="flex items-center space-x-2">
                    <span class="text-sm text-gray-500 fixed-width">{{ doc.size }} KB</span>
                    <span :class="getBadgeClass(doc.daysLeft)" class="badge fixed-width">{{ doc.daysLeft }} days</span>
                  </div>
                </div>
              </button>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <!-- Document Viewer Section -->
    <div class="document-viewer flex w-full md:w-3/5 h-5/6">
      <div class="w-2/3 border border-gray-300 rounded-lg bg-gray-100">
        <iframe :src="googleDocsViewerUrl" width="100%" class="h-full"></iframe>
      </div>
      <div class="w-1/3 p-4 bg-white rounded-lg shadow-sm ml-2">
        <div class="flex justify-between items-center mb-4">
          <span class="text-lg font-semibold">Cabinet</span>
          <button class="flex items-center space-x-2 px-2 py-1 text-gray-700 bg-gray-100 rounded-md hover:bg-gray-200 transition" @click="toggleAddFileModal">
            <Icon name="mdi:file-plus" size="1.25rem"></Icon>
          </button>
        </div>
        <div class="flex items-center mb-4 space-x-2">
          <select v-model="selectedCabinet" class="w-full p-1 border border-gray-300 rounded-md">
            <option disabled value="">Select a cabinet</option>
            <option v-for="cabinet in cabinets" :key="cabinet" :value="cabinet" class="text-sm">{{ cabinet }}</option>
          </select>
          <button class="flex items-center space-x-2 px-2 py-1 text-gray-700 bg-gray-100 rounded-md hover:bg-gray-200 transition" @click="toggleModal">
            <Icon name="mdi:file-cabinet" size="1.25rem"></Icon>
          </button>
        </div>
        <div class="cabinet-list overflow-y-auto h-5/6">
          <ul :class="{ 'list-none': !isGridView}" class="bg-gray-100 rounded-md py-4 px-1 h-full">
            <li v-for="cabinetDoc in cabinetDocuments[selectedCabinet]" :key="cabinetDoc.url" :class="{'text-left': !isGridView, 'bg-gray-300': cabinetDoc.url === documentUrl}" draggable="true" @dragstart="handleDragStartDocument(cabinetDoc)">
              <button @click="selectDocument(cabinetDoc.url)" class="py-2 w-full flex justify-between items-center hover:bg-gray-100 transition">
                
                <div  class="flex items-center justify-between w-full">
                  <div class="flex items-center space-x-2">
                    <Icon :name="getFileIcon(cabinetDoc.url)" size="1.25rem"></Icon>
                    <span class="whitespace-nowrap overflow-hidden overflow-ellipsis">{{ cabinetDoc.name }}</span>
                  </div>
                </div>
              </button>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal -->
  <div v-if="showModal" class="fixed inset-0 bg-gray-600 bg-opacity-50 flex items-center justify-center">
    <div class="bg-white p-4 rounded-lg shadow-lg w-1/3 relative max-h-screen overflow-y-auto">
      <button @click="toggleModal" class="absolute top-3 right-2 text-gray-500 hover:text-gray-700">
        <Icon name="mdi:close" size="1.5rem"></Icon>
      </button>
      <h2 class="text-lg font-semibold mb-4">Manage Cabinets</h2>
      <div class="cabinet-list overflow-y-auto h-48">
        <ul class="grid grid-cols-1 gap-4">
          <li v-for="(cabinet, index) in cabinets" :key="index" class="flex justify-between items-center bg-white border-b-2 px-2 py-4 rounded-sm hover:bg-gray-100 transition">
            <span>{{ cabinet }}</span>
            <button @click="deleteCabinet(index)" class="text-red-500 hover:text-red-700">
              <Icon name="mdi:delete" size="1.25rem"></Icon>
            </button>
          </li>
        </ul>
      </div>
      <div class="flex items-center my-4">
        <input v-model="newCabinetName" placeholder="New cabinet name" class="p-2 border border-gray-300 rounded-lg w-full mr-2" />
        <rs-button @click="addCabinet" class="bg-blue-500 text-white px-4 py-2 rounded-lg flex items-center">
          <Icon name="mdi:plus" size="1.25rem" class="mr-1"></Icon>
          Add
        </rs-button>
      </div>
    </div>
  </div>

  <!-- Add File Modal -->
  <div v-if="showAddFileModal" class="fixed inset-0 bg-gray-600 bg-opacity-50 flex items-center justify-center">
    <div class="bg-white p-4 rounded-lg shadow-lg w-1/3 relative max-h-screen overflow-y-auto">
      <button @click="toggleAddFileModal" class="absolute top-3 right-2 text-gray-500 hover:text-gray-700">
        <Icon name="mdi:close" size="1.5rem"></Icon>
      </button>
      <h2 class="text-lg font-semibold mb-4">Add Document to Cabinet</h2>
      <div class="document-list p-4 flex-grow overflow-y-auto mb-4">
        <span class="p-2"> Select Document File</span>
        <ul :class="{'grid grid-cols-2 gap-4': isGridView, 'list-none': !isGridView}" class="bg-gray-100 rounded-md py-4 px-1 mt-2">
          <li v-for="doc in documents" :key="doc.url" :class="{'flex flex-col items-center': isGridView, 'text-left': !isGridView, 'bg-gray-300': doc.url === selectedDocument?.url}" @click="selectedDocument = doc">
            <button class="py-2 px-2 w-full flex justify-between items-center hover:bg-gray-100 transition">
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
      <div class="mb-4 p-2">
        <span> Assign to Cabinet</span>
        <select v-model="newFileCabinet" class="w-full p-2 border border-gray-300 rounded-md">
          <option disabled value="">Select a cabinet</option>
          <option v-for="cabinet in cabinets" :key="cabinet" :value="cabinet">{{ cabinet }}</option>
        </select>
      </div>
      <div class="flex justify-end">
        <rs-button @click="addFileToSelectedCabinet" class="text-white px-4 py-2 rounded-lg flex items-center">
          <Icon name="mdi:plus" size="1.25rem" class="mr-1"></Icon>
          Add File
        </rs-button>
      </div>
    </div>
  </div>

  <!-- New Form Modal -->
  <div v-if="showNewFormModal" class="fixed inset-0 bg-gray-600 bg-opacity-50 flex items-center justify-center">
    <div class="bg-white p-4 rounded-lg shadow-lg w-1/3 relative max-h-screen overflow-y-auto">
      <button @click="toggleNewFormModal" class="absolute top-3 right-2 text-gray-500 hover:text-gray-700">
        <Icon name="mdi:close" size="1.5rem"></Icon>
      </button>
      <h2 class="text-lg font-semibold mb-4">Import New File</h2>
      <FormKit
        type="file"
        name="documents"
        label="Drag and drop files here or click to import into document tray"
        help="Supported formats: PDF, DOC, DOCX, XLS, XLSX"
        @input="handleFileInput"
        class="drag-drop-area border-dashed border-2 mt-2 border-gray-300 p-4 rounded-lg bg-gray-50 flex flex-col items-center justify-center h-32"
      />
      <div class="mt-4">
        <span class="text-lg font-semibold mb-2">Metadata</span>
        <div class="grid grid-cols-2 gap-4">
          <FormKit type="text" name="tajuk" label="Tajuk" v-model="metadata.tajuk" />
          <FormKit type="text" name="perkara" label="Perkara" v-model="metadata.perkara" />
          <FormKit type="text" name="negeri" label="Negeri" v-model="metadata.negeri" />
          <FormKit type="date" name="tarikh" label="Tarikh" v-model="metadata.tarikh" />
          <FormKit type="text" name="namaFail" label="Nama Fail" v-model="metadata.namaFail" />
          <FormKit type="text" name="user" label="User" v-model="metadata.user" />
          <FormKit type="text" name="fulltext" label="Fulltext" v-model="metadata.fulltext" />
          <FormKit type="date" name="storeDate" label="Store Date" v-model="metadata.storeDate" />
        </div>
      </div>
      <div class="flex justify-end mt-4">
        <rs-button @click="submitNewForm" class="text-white px-4 py-2 rounded-lg flex items-center">
          <Icon name="mdi:upload" size="1.25rem" class="mr-1"></Icon>
          Import
        </rs-button>
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
          documentUrl: 'https://www.adobe.com/support/products/enterprise/knowledgecenter/media/c4611_sample_explain.pdf', // URL of the Test Document
          documents: [
              { name: 'Test Document', url: 'https://www.adobe.com/support/products/enterprise/knowledgecenter/media/c4611_sample_explain.pdf', size: 512, daysLeft: 25 },
              { name: 'Laporan Pemeriksaan Tapak', url: 'https://www.example.com/sample2.pdf', size: 211, daysLeft: 15 },
              { name: 'Log Penyelenggaraan Peralatan', url: 'https://www.example.com/sample3.pdf', size: 112, daysLeft: 5 },
              { name: 'Protokol Keselamatan', url: 'https://www.example.com/sample4.pdf', size: 343, daysLeft: 0 },
              { name: 'Spesifikasi Bahan', url: 'https://www.example.com/sample6.pdf', size: 878, daysLeft: 28 },
              { name: 'Perjanjian Kontrak', url: 'https://www.example.com/sample7.pdf', size: 78, daysLeft: 4 },
              { name: 'Laporan Bajet', url: 'https://www.example.com/sample8.pdf', size: 66, daysLeft: 25 },
              { name: 'Foto Kemajuan', url: 'https://www.example.com/sample9.pdf', size: 389, daysLeft: 15 }
          ], // List of documents with sizes in KB
          isGridView: false, // Toggle between grid and list view
          sortOrder: 'asc', // Track the current sort order
          searchQuery: '', // Add search query to data
          selectedCabinet: 'Jabatan Ukur', // Set default value to "Jabatan Ukur"
          cabinets: ['Jabatan Ukur', 'Jabatan Infra', 'Jabatan IT'], // Updated list of cabinet names
          cabinetDocuments: {
              'Jabatan Ukur': [
                  { name: 'Laporan Ukur Topografi', url: 'https://www.example.com/cabinet1.pdf', size: 123, daysLeft: 25 },
                  { name: 'Data Ukur Sempadan', url: 'https://www.example.com/cabinet2.pdf', size: 456, daysLeft: 15 },
                  { name: 'Rangkaian Kawalan Geodetik', url: 'https://www.example.com/cabinet3.pdf', size: 789, daysLeft: 5 },
              ],
              'Jabatan Infra': [
                  { name: 'Pelan Reka Bentuk Infrastruktur', url: 'https://www.example.com/cabinet4.pdf', size: 123, daysLeft: 20 },
                  { name: 'Pemetaan Utiliti', url: 'https://www.example.com/cabinet5.pdf', size: 456, daysLeft: 10 },
              ],
              'Jabatan IT': [
                  { name: 'Pelan Infrastruktur IT', url: 'https://www.example.com/cabinet6.pdf', size: 789, daysLeft: 30 },
              ],
          }, // List of documents in the selected cabinet
          showModal: false, // Track modal visibility
          newCabinetName: '', // Track new cabinet name input
          showAddFileModal: false, // Track add file modal visibility
          newFileCabinet: '', // Track selected cabinet for new file
          selectedDocument: null, // Track selected document
          showNewFormModal: false,
          newFormInput: '',
          metadata: {
            tajuk: '',
            perkara: '',
            negeri: '',
            tarikh: '',
            namaFail: '',
            user: '',
            fulltext: '',
            storeDate: ''
          },
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
      },
      toggleModal() {
          this.showModal = !this.showModal;
      },
      addCabinet() {
          if (this.newCabinetName.trim()) {
              this.cabinets.push(this.newCabinetName.trim());
              this.newCabinetName = '';
          }
      },
      deleteCabinet(index) {
          this.cabinets.splice(index, 1);
      },
      addFileToCabinet() {
        // Logic to add a new file to the cabinet
      },
      toggleAddFileModal() {
        this.showAddFileModal = !this.showAddFileModal;
      },
      handleNewFileInput(files) {
        // Logic to handle new file input
      },
      addFileToSelectedCabinet() {
        if (this.selectedDocument && this.newFileCabinet) {
          if (!this.cabinetDocuments[this.newFileCabinet]) {
            this.$set(this.cabinetDocuments, this.newFileCabinet, []);
          }
          this.cabinetDocuments[this.newFileCabinet].push(this.selectedDocument);
          this.toggleAddFileModal();
        }
      },
      toggleNewFormModal() {
        this.showNewFormModal = !this.showNewFormModal;
      },
      submitNewForm() {
        console.log(this.newFormInput);
        this.toggleNewFormModal();
      },
      getBadgeClass(daysLeft) {
        if (daysLeft >= 21 && daysLeft <= 30) {
          return 'bg-green-500 text-white';
        } else if (daysLeft >= 11 && daysLeft <= 20) {
          return 'bg-yellow-500 text-white';
        } else if (daysLeft >= 0 && daysLeft <= 10) {
          return 'bg-red-500 text-white';
        }
        return '';
      },
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
.badge {
  padding: 0.25rem 0.5rem;
  border-radius: 0.25rem;
  font-size: 0.75rem;
  font-weight: 600;
}
.fixed-width {
  width: 80px;
  text-align: center;
}
</style>
