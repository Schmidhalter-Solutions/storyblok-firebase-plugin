<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import { collection, getDocs } from 'firebase/firestore'
import { db } from './firebase'
import { useFieldPlugin } from '@storyblok/field-plugin/vue3'

interface Category {
  id: string;
  name: {
    en: string;
  };
  order?: number;
  slug?: string;
}

const plugin = useFieldPlugin({
  validateContent: (content: unknown) => {
    if (Array.isArray(content)) {
      return { content }
    }
    return { content: [] }
  }
})

const selectedDocuments = ref<string[]>([])
const documents = ref<Category[]>([])
const loading = ref(true)
const error = ref<string | null>(null)
const visibleDocuments = ref<number>(10)
const isLoadingMore = ref(false)

const loadDocuments = async () => {
  try {
    const querySnapshot = await getDocs(collection(db, import.meta.env.VITE_FIREBASE_COLLECTION))
    documents.value = querySnapshot.docs
      .map(doc => ({
        id: doc.id,
        ...doc.data()
      } as Category))
      .sort((a, b) => (a.order || 0) - (b.order || 0))
  } catch (err) {
    error.value = 'Failed to load documents'
    console.error(err)
  } finally {
    loading.value = false
  }
}

const handleScroll = (event: Event) => {
  const target = event.target as HTMLElement
  const { scrollTop, scrollHeight, clientHeight } = target
  
  // Load more when user scrolls to bottom (with 20px threshold)
  if (scrollHeight - scrollTop - clientHeight < 20 && !isLoadingMore.value) {
    loadMore()
  }
}

const loadMore = () => {
  if (visibleDocuments.value < documents.value.length) {
    isLoadingMore.value = true
    // Simulate loading delay for better UX
    setTimeout(() => {
      visibleDocuments.value += 10
      isLoadingMore.value = false
    }, 300)
  }
}

const getDocumentSlug = (docId: string): string => {
  const doc = documents.value.find(d => d.id === docId)
  return doc?.slug || docId
}

const toggleDocument = (docId: string) => {
  const index = selectedDocuments.value.indexOf(docId)
  const newSelection = [...selectedDocuments.value]
  
  if (index === -1) {
    newSelection.push(docId)
  } else {
    newSelection.splice(index, 1)
  }
  
  // Filter out any null values and ensure we only have valid strings
  selectedDocuments.value = newSelection.filter(id => id !== null && typeof id === 'string')
}

// Watch for changes in selectedDocuments and update the plugin content
watch(selectedDocuments, (newValue) => {
  if (plugin.type === 'loaded' && plugin.data) {
    // Convert IDs to slugs before sending to Storyblok
    const filteredContent = newValue
      .filter(id => id !== null && typeof id === 'string')
      .map(id => getDocumentSlug(id))
    plugin.actions.setContent(filteredContent)
  }
}, { deep: true })

// Initialize selectedDocuments from plugin content when loaded
watch(() => plugin.type, (newType) => {
  if (newType === 'loaded' && plugin.data && Array.isArray(plugin.data.content)) {
    // Convert slugs back to IDs when initializing from Storyblok content
    const slugs = (plugin.data.content as string[]).filter(id => id !== null && typeof id === 'string')
    selectedDocuments.value = slugs.map(slug => {
      const doc = documents.value.find(d => d.slug === slug)
      return doc?.id || slug
    })
  }
})

onMounted(() => {
  loadDocuments()
})
</script>

<template>
  <div class="document-selector">
    <div v-if="loading" class="loading">Loading documents...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else class="document-list" @scroll="handleScroll">
      <label
        v-for="doc in documents.slice(0, visibleDocuments)"
        :key="doc.id"
        class="document-item"
        :class="{ selected: selectedDocuments.includes(doc.id) }"
      >
        <input
          type="checkbox"
          :checked="selectedDocuments.includes(doc.id)"
          @change="toggleDocument(doc.id)"
        />
        <span>{{ doc.name['en'].toLowerCase().charAt(0).toUpperCase() + doc.name['en'].toLowerCase().slice(1) || doc.id }}</span>
      </label>
      <div v-if="isLoadingMore" class="loading-more">Loading more...</div>
    </div>
  </div>
</template>

<style scoped>
.document-selector {
  padding: 1rem;
}

.loading, .error {
  padding: 1rem;
  text-align: center;
}

.error {
  color: red;
}

.document-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  max-height: 400px;
  overflow-y: auto;
  padding-right: 0.5rem;
}

.document-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  user-select: none;
}

.document-item:hover {
  background-color: #f5f5f5;
}

.document-item.selected {
  background-color: #e3f2fd;
  border-color: #2196f3;
}

input[type="checkbox"] {
  margin: 0;
  cursor: pointer;
}

.loading-more {
  text-align: center;
  padding: 0.5rem;
  color: #666;
}
</style>
