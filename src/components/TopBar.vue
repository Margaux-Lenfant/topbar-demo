<script setup>
import { ref, computed, watch, nextTick } from 'vue'

defineProps({
  compactDate: {
    type: Boolean,
    default: false
  }
})

// Corpus
const selectedCorpus = ref('France')
const corpusOptions = ['Argentina', 'Bulgaria', 'Colombia', 'Czechia', 'Denmark', 'EU', 'France']
const showCorpusDropdown = ref(false)
const corpusSearch = ref('')

const filteredCorpusOptions = () => {
  if (!corpusSearch.value) return corpusOptions
  return corpusOptions.filter(option =>
    option.toLowerCase().includes(corpusSearch.value.toLowerCase())
  )
}

const selectCorpus = (corpus) => {
  selectedCorpus.value = corpus
  showCorpusDropdown.value = false
  corpusSearch.value = ''
}

// Sources
const sourceSearch = ref('')
const showSourcesDropdown = ref(false)
const selectedItem = ref(null) // Track selected item (single selection for pre-registered)
const sourceSearchInput = ref(null)

// Entity search - multiple selection
const selectedSearchEntities = ref([]) // Track multiple selected entities from search
const validatedSearchEntities = ref([]) // Track validated entities

// Mock entity database for search
const allEntities = [
  { id: 'entity-1', label: 'Ile de France - Stakeholders' },
  { id: 'entity-2', label: 'Leeanna Mac Vacaron' },
  { id: 'entity-3', label: 'Data Annuel France - Stakeholder' },
  { id: 'entity-4', label: 'Italy Stakeholders' },
  { id: 'entity-5', label: 'France Corporate' },
  { id: 'entity-6', label: 'France Government' },
  { id: 'entity-7', label: 'EU Parliament Members' },
  { id: 'entity-8', label: 'Bulgaria Officials' },
  { id: 'entity-9', label: 'Denmark Stakeholders' },
  { id: 'entity-10', label: 'Colombia Partners' }
]

// Filtered entities based on search input
const filteredSearchEntities = computed(() => {
  if (!sourceSearch.value || sourceSearch.value.length < 1) return []
  const searchTerm = sourceSearch.value.toLowerCase()
  return allEntities.filter(entity =>
    entity.label.toLowerCase().includes(searchTerm)
  )
})

// Check if entity is selected in search results
const isEntitySelected = (entityId) => {
  return selectedSearchEntities.value.includes(entityId)
}

// Toggle entity selection in search results
const toggleEntitySelection = (entityId) => {
  const index = selectedSearchEntities.value.indexOf(entityId)
  if (index === -1) {
    selectedSearchEntities.value.push(entityId)
  } else {
    selectedSearchEntities.value.splice(index, 1)
  }
}

// Validate selected entities
const validateSearchSelection = () => {
  validatedSearchEntities.value = [...selectedSearchEntities.value]
  showSourcesDropdown.value = false
  sourceSearch.value = ''
  // Clear single selection when validating multiple entities
  selectedItem.value = null
}

// Clear search selection
const clearSearchSelection = () => {
  selectedSearchEntities.value = []
}

// Save the list (when more than 1 entity selected)
const saveEntityList = () => {
  // This would typically save to backend or local storage
  console.log('Saving entity list:', selectedSearchEntities.value)
  alert('Liste sauvegardée avec ' + selectedSearchEntities.value.length + ' entités')
}

// Show action buttons when entities are selected in search
const showSearchActionButtons = computed(() => {
  return selectedSearchEntities.value.length > 0
})

// Displayed entities: show when searching OR when there are selected entities
const displayedEntities = computed(() => {
  if (sourceSearch.value.length > 0) {
    return filteredSearchEntities.value
  }
  // When not searching, show selected entities so user can uncheck them
  if (selectedSearchEntities.value.length > 0) {
    return allEntities.filter(entity =>
      selectedSearchEntities.value.includes(entity.id)
    )
  }
  return []
})

// Check if entities section should be visible
const showEntitiesSection = computed(() => {
  return displayedEntities.value.length > 0
})

// Get selected entities as objects (for displaying tags)
const selectedEntitiesObjects = computed(() => {
  return allEntities.filter(entity =>
    selectedSearchEntities.value.includes(entity.id)
  )
})

// Get filtered entities excluding already selected ones (to avoid duplicates in search results)
const nonSelectedFilteredEntities = computed(() => {
  return filteredSearchEntities.value.filter(entity =>
    !selectedSearchEntities.value.includes(entity.id)
  )
})

// Remove entity from selection (for tag X button)
const removeEntityFromSelection = (entityId) => {
  const index = selectedSearchEntities.value.indexOf(entityId)
  if (index !== -1) {
    selectedSearchEntities.value.splice(index, 1)
  }
}

// Filter category items based on search
const getFilteredCategoryItems = (category) => {
  if (!sourceSearch.value) return category.items
  const searchTerm = sourceSearch.value.toLowerCase()
  return category.items.filter(item =>
    item.label.toLowerCase().includes(searchTerm)
  )
}

// Check if category should show its items
const shouldShowCategoryItems = (category) => {
  const searchTerm = sourceSearch.value?.trim() || ''
  if (searchTerm.length > 0) {
    // When searching: auto-expand ONLY if has matching items
    const matchingItems = category.items.filter(item =>
      item.label.toLowerCase().includes(searchTerm.toLowerCase())
    )
    return matchingItems.length > 0
  }
  // When not searching: use manual expanded state
  return category.expanded
}

// Highlight matching text with bold
const highlightMatch = (text) => {
  const searchTerm = sourceSearch.value?.trim() || ''
  if (!searchTerm) return text

  const regex = new RegExp(`(${searchTerm.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})`, 'gi')
  return text.replace(regex, '<span class="highlight-match">$1</span>')
}

watch(showSourcesDropdown, (isOpen) => {
  if (isOpen) {
    nextTick(() => {
      sourceSearchInput.value?.focus()
    })
    // Keep current selection when reopening
  }
})

const sourceCategories = ref([
  {
    id: 'entities',
    label: "Listes d'entités",
    expanded: false,
    items: [
      { id: 'sub-entities-paps', label: 'Sub-entities PAPs' },
      { id: 'czechia-paps', label: 'Czechia PAPs' },
      { id: 'italy', label: 'Italy' }
    ]
  },
  {
    id: 'communities',
    label: 'Communautés',
    expanded: false,
    items: [
      { id: 'community-1', label: 'Community 1' },
      { id: 'community-2', label: 'Community 2' }
    ]
  },
  {
    id: 'subcommunities',
    label: 'Sous-communautés',
    expanded: false,
    items: [
      { id: 'subcommunity-1', label: 'Subcommunity 1' },
      { id: 'subcommunity-2', label: 'Subcommunity 2' }
    ]
  },
  {
    id: 'stakeholders',
    label: 'Stakeholders',
    expanded: false,
    items: [
      { id: 'stakeholder-1', label: 'Stakeholder 1' },
      { id: 'stakeholder-2', label: 'Stakeholder 2' }
    ]
  }
])

const toggleSourceCategory = (category) => {
  category.expanded = !category.expanded
}

const selectItem = (itemId) => {
  selectedItem.value = itemId
  showSourcesDropdown.value = false
  // Clear multiple entity selection when selecting a pre-registered item
  selectedSearchEntities.value = []
  validatedSearchEntities.value = []
  sourceSearch.value = ''
}

const clearSelection = () => {
  selectedItem.value = null
  validatedSearchEntities.value = []
  selectedSearchEntities.value = []
}

const selectedItemLabel = computed(() => {
  if (!selectedItem.value) return null
  for (const category of sourceCategories.value) {
    const item = category.items.find(i => i.id === selectedItem.value)
    if (item) return item.label
  }
  return null
})

// Sujets
const subjectSearch = ref('')
const showSujetsDropdown = ref(false)
const subjectSearchInput = ref(null)
const selectedSubject = ref(null) // Single selection only
const selectedKeywords = ref([]) // Tableau pour stocker les mots-clés personnalisés
const recentSearches = ref([]) // Historique des recherches récentes

// Computed pour afficher les tags de mots-clés
const keywordTags = computed(() => {
  return selectedKeywords.value.map((keyword, index) => ({
    id: `keyword-${index}`,
    label: keyword
  }))
})

// Liste simple de queries (pas de catégories)
const subjectItems = ref([
  { id: 'query-1', label: 'France :: 20225 (VW Transverses et Pays)' },
  { id: 'query-2', label: 'France :: Mobilité électrique' },
  { id: 'query-3', label: 'EU :: Réglementation environnementale' },
  { id: 'query-4', label: 'Bulgaria :: Automotive Industry' },
  { id: 'query-5', label: 'Denmark :: Green Transition' }
])

const selectSubject = (itemId) => {
  selectedSubject.value = itemId
  showSujetsDropdown.value = false
  subjectSearch.value = ''
}

const clearSubjectSelection = () => {
  selectedSubject.value = null
}

const selectedSubjectLabel = computed(() => {
  if (!selectedSubject.value) return null
  const item = subjectItems.value.find(i => i.id === selectedSubject.value)
  return item ? item.label : null
})

// Filter subject items based on search
const filteredSubjectItems = computed(() => {
  if (!subjectSearch.value) return subjectItems.value
  const searchTerm = subjectSearch.value.toLowerCase()
  return subjectItems.value.filter(item =>
    item.label.toLowerCase().includes(searchTerm)
  )
})

// Highlight matching text for subjects
const highlightSubjectMatch = (text) => {
  const searchTerm = subjectSearch.value?.trim() || ''
  if (!searchTerm) return text

  const regex = new RegExp(`(${searchTerm.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})`, 'gi')
  return text.replace(regex, '<span class="highlight-match">$1</span>')
}

// Ajouter un mot-clé
const addKeyword = (keyword = null) => {
  const keywordToAdd = keyword || subjectSearch.value.trim()
  if (keywordToAdd && !selectedKeywords.value.includes(keywordToAdd)) {
    selectedKeywords.value.push(keywordToAdd)
    // Ajouter aux recherches récentes (max 10, éviter les doublons)
    if (!recentSearches.value.includes(keywordToAdd)) {
      recentSearches.value.unshift(keywordToAdd)
      if (recentSearches.value.length > 10) {
        recentSearches.value.pop()
      }
    }
    subjectSearch.value = ''
    showSujetsDropdown.value = false
  }
}

// Supprimer un mot-clé
const removeKeyword = (index) => {
  selectedKeywords.value.splice(index, 1)
}

// Gérer l'événement keydown sur l'input
const handleSubjectKeydown = (event) => {
  if (event.key === 'Enter' && subjectSearch.value.trim()) {
    event.preventDefault()
    addKeyword()
  }
}

watch(showSujetsDropdown, (isOpen) => {
  if (isOpen) {
    nextTick(() => {
      subjectSearchInput.value?.focus()
    })
  }
})

// Date Picker
const showDatePicker = ref(false)
const startDate = ref('2026-01-01')
const endDate = ref('2026-01-16')
const tempStartDate = ref(null)
const tempEndDate = ref(null)
const selectedPeriod = ref('personnalise')

const periodOptions = [
  { id: '7jours', label: '7 jours', days: 7 },
  { id: '14jours', label: '14 jours', days: 14 },
  { id: '1mois', label: '1 mois', months: 1 },
  { id: '3mois', label: '3 mois', months: 3 },
  { id: '6mois', label: '6 mois', months: 6 },
  { id: '1an', label: '1 an', years: 1 },
  { id: 'personnalise', label: 'Personnalisé', custom: true }
]

// Calendar state
const leftMonth = ref(new Date(2026, 0, 1)) // January 2026
const rightMonth = ref(new Date(2026, 1, 1)) // February 2026
const hoveredDate = ref(null)

const formatDateRange = () => {
  const format = (dateStr) => new Date(dateStr).toLocaleDateString('fr-FR')
  return `${format(startDate.value)} - ${format(endDate.value)}`
}

const formatDateInput = (dateStr) => {
  if (!dateStr) return ''
  const date = new Date(dateStr)
  const day = String(date.getDate()).padStart(2, '0')
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const year = date.getFullYear()
  return `${day}/${month}/${year}`
}

// Format date to YYYY-MM-DD without timezone conversion
const toDateString = (date) => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

const openDatePicker = () => {
  tempStartDate.value = startDate.value
  tempEndDate.value = endDate.value
  showDatePicker.value = true
}

const closeDatePicker = () => {
  showDatePicker.value = false
}

const clearDates = () => {
  tempStartDate.value = null
  tempEndDate.value = null
  selectedPeriod.value = 'personnalise'
}

const cancelDatePicker = () => {
  showDatePicker.value = false
}

const validateDates = () => {
  if (tempStartDate.value && tempEndDate.value) {
    startDate.value = tempStartDate.value
    endDate.value = tempEndDate.value
  }
  showDatePicker.value = false
}

const selectPeriod = (period) => {
  selectedPeriod.value = period.id
  if (!period.custom) {
    const end = new Date()
    const start = new Date()

    if (period.days) {
      start.setDate(end.getDate() - period.days + 1)
    } else if (period.months) {
      start.setMonth(end.getMonth() - period.months)
      start.setDate(start.getDate() + 1)
    } else if (period.years) {
      start.setFullYear(end.getFullYear() - period.years)
      start.setDate(start.getDate() + 1)
    }

    tempStartDate.value = toDateString(start)
    tempEndDate.value = toDateString(end)

    // Update calendar view to show the selected range
    leftMonth.value = new Date(start.getFullYear(), start.getMonth(), 1)
    rightMonth.value = new Date(leftMonth.value.getFullYear(), leftMonth.value.getMonth() + 1, 1)
  }
}

// Calendar navigation
const prevMonth = () => {
  leftMonth.value = new Date(leftMonth.value.getFullYear(), leftMonth.value.getMonth() - 1, 1)
  rightMonth.value = new Date(rightMonth.value.getFullYear(), rightMonth.value.getMonth() - 1, 1)
}

const nextMonth = () => {
  leftMonth.value = new Date(leftMonth.value.getFullYear(), leftMonth.value.getMonth() + 1, 1)
  rightMonth.value = new Date(rightMonth.value.getFullYear(), rightMonth.value.getMonth() + 1, 1)
}

const getMonthName = (date) => {
  return date.toLocaleDateString('fr-FR', { month: 'long', year: 'numeric' })
}

const getDaysInMonth = (date) => {
  const year = date.getFullYear()
  const month = date.getMonth()
  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)
  const days = []

  // Add empty slots for days before the first day of the month
  const startDay = firstDay.getDay() === 0 ? 6 : firstDay.getDay() - 1 // Monday = 0
  for (let i = 0; i < startDay; i++) {
    days.push(null)
  }

  // Add all days of the month
  for (let i = 1; i <= lastDay.getDate(); i++) {
    days.push(new Date(year, month, i))
  }

  return days
}

const isDateInRange = (date) => {
  if (!date || !tempStartDate.value) return false
  const d = new Date(date)
  const start = new Date(tempStartDate.value)

  // If we have an end date, use it
  if (tempEndDate.value) {
    const end = new Date(tempEndDate.value)
    return d > start && d < end
  }

  // If hovering while selecting, show preview range
  if (hoveredDate.value && !tempEndDate.value) {
    const hovered = new Date(hoveredDate.value)
    if (hovered > start) {
      return d > start && d < hovered
    } else if (hovered < start) {
      return d > hovered && d < start
    }
  }

  return false
}

const isDateInRangePreview = (date) => {
  if (!date || !tempStartDate.value || tempEndDate.value || !hoveredDate.value) return false
  const d = new Date(date)
  const start = new Date(tempStartDate.value)
  const hovered = new Date(hoveredDate.value)

  if (hovered > start) {
    return d > start && d <= hovered
  } else if (hovered < start) {
    return d >= hovered && d < start
  }
  return false
}

const onDateHover = (date) => {
  if (date && tempStartDate.value && !tempEndDate.value) {
    hoveredDate.value = toDateString(date)
  }
}

const onDateLeave = () => {
  hoveredDate.value = null
}

const isStartDate = (date) => {
  if (!date || !tempStartDate.value) return false
  return new Date(date).toDateString() === new Date(tempStartDate.value).toDateString()
}

const isEndDate = (date) => {
  if (!date || !tempEndDate.value) return false
  return new Date(date).toDateString() === new Date(tempEndDate.value).toDateString()
}

const selectDate = (date) => {
  if (!date) return
  const dateStr = toDateString(date)

  selectedPeriod.value = 'personnalise'
  hoveredDate.value = null

  if (!tempStartDate.value || (tempStartDate.value && tempEndDate.value)) {
    // Start a new selection
    tempStartDate.value = dateStr
    tempEndDate.value = null
  } else {
    // Complete the selection
    if (new Date(dateStr) < new Date(tempStartDate.value)) {
      tempEndDate.value = tempStartDate.value
      tempStartDate.value = dateStr
    } else {
      tempEndDate.value = dateStr
    }
  }
}

// Check if we're in selection mode (start selected, no end yet)
const isSelectingRange = computed(() => {
  return tempStartDate.value && !tempEndDate.value
})

// Check if date should show the range band (for start date only when there's an end or preview)
const hasRangeBand = (date) => {
  if (!date) return false
  const isStart = isStartDate(date)
  const isEnd = isEndDate(date)

  if (isStart && !isEnd) {
    return tempEndDate.value || (hoveredDate.value && new Date(hoveredDate.value) > new Date(tempStartDate.value))
  }
  if (isEnd && !isStart) {
    return true
  }
  return false
}

const weekDays = ['Lu', 'Ma', 'Me', 'Je', 'Ve', 'Sa', 'Di']
</script>

<template>
  <div class="topbar">
    <!-- Corpus Dropdown -->
    <div class="topbar-item corpus-dropdown">
      <button class="dropdown-trigger" @click="showCorpusDropdown = !showCorpusDropdown">
        <span>{{ selectedCorpus }}</span>
        <svg class="chevron" :class="{ rotated: showCorpusDropdown }" width="18" height="18" viewBox="0 0 18 18" fill="none">
          <path d="M4.5 6.75L9 11.25L13.5 6.75" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </button>
      <div v-if="showCorpusDropdown" class="dropdown-menu corpus-menu">
        <div class="corpus-search">
          <input type="text" v-model="corpusSearch" placeholder="" />
          <svg width="12" height="12" viewBox="0 0 15 15" fill="none">
            <path d="M13.125 13.125L10.4062 10.4062M11.875 6.875C11.875 9.63642 9.63642 11.875 6.875 11.875C4.11358 11.875 1.875 9.63642 1.875 6.875C1.875 4.11358 4.11358 1.875 6.875 1.875C9.63642 1.875 11.875 4.11358 11.875 6.875Z" stroke="#595959" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
        <div class="corpus-options">
          <button
            v-for="option in filteredCorpusOptions()"
            :key="option"
            class="dropdown-option"
            :class="{ active: option === selectedCorpus }"
            @click="selectCorpus(option)"
          >
            {{ option }}
          </button>
        </div>
      </div>
    </div>

    <!-- Sources Search -->
    <div class="topbar-item search-group sources-dropdown" :class="{ active: showSourcesDropdown }">
      <span
        class="sources-floating-label"
        :class="{ floating: showSourcesDropdown || selectedItem || selectedEntitiesObjects.length > 0, 'floating-active': showSourcesDropdown }"
        @click="showSourcesDropdown = !showSourcesDropdown"
      >Sources</span>

      <div class="search-input-wrapper" @click="showSourcesDropdown = true">
        <svg class="search-icon" width="15" height="15" viewBox="0 0 15 15" fill="none">
          <path d="M13.125 13.125L10.4062 10.4062M11.875 6.875C11.875 9.63642 9.63642 11.875 6.875 11.875C4.11358 11.875 1.875 9.63642 1.875 6.875C1.875 4.11358 4.11358 1.875 6.875 1.875C9.63642 1.875 11.875 4.11358 11.875 6.875Z" stroke="#595959" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>

        <!-- Entity Tags in search bar -->
        <div v-if="selectedEntitiesObjects.length > 0 && !showSourcesDropdown" class="search-bar-tags">
          <div
            v-for="entity in selectedEntitiesObjects"
            :key="entity.id"
            class="entity-tag"
          >
            <span class="entity-tag-label">{{ entity.label }}</span>
            <button class="entity-tag-remove" @click.stop="removeEntityFromSelection(entity.id)">
              <svg width="8" height="8" viewBox="0 0 8 8" fill="none">
                <path d="M7 1L1 7" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M1 1L7 7" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </button>
          </div>
        </div>

        <!-- Selected item label (for category items) -->
        <span v-if="selectedItem && !showSourcesDropdown && selectedEntitiesObjects.length === 0" class="selected-item-label">{{ selectedItemLabel }}</span>

        <input
          v-show="(selectedEntitiesObjects.length === 0 && !selectedItem) || showSourcesDropdown"
          ref="sourceSearchInput"
          type="text"
          v-model="sourceSearch"
          :placeholder="showSourcesDropdown ? '' : 'Chercher un auteur, une communauté ...'"
          @focus="showSourcesDropdown = true"
        />
      </div>

      <button v-if="selectedItem && !showSourcesDropdown" class="clear-selection-btn" @click.stop="clearSelection">
        <svg width="15" height="15" viewBox="0 0 15 15" fill="none">
          <path d="M11.25 3.75L3.75 11.25" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M3.75 3.75L11.25 11.25" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </button>

      <div v-if="showSourcesDropdown" class="sources-categories-panel" :class="{ 'has-action-buttons': showSearchActionButtons }">
        <!-- Scrollable content -->
        <div class="panel-scrollable-content">
          <!-- Entités Section (when searching with results OR when entities are selected) -->
          <div v-if="showEntitiesSection || selectedEntitiesObjects.length > 0" class="entities-section">
            <div class="section-header">
              <span class="section-title">Entités</span>
              <button class="save-list-link" @click.stop="saveEntityList">
                <svg width="14" height="14" viewBox="0 0 14 14" fill="none">
                  <path d="M11.0833 12.25V7.58333H2.91667V12.25M2.91667 1.75V4.66667H9.33333M12.25 12.25H1.75C1.42783 12.25 1.11889 12.1222 0.890524 11.8938C0.662159 11.6654 0.534167 11.3565 0.534167 11.0343V2.96667C0.534167 2.6445 0.662159 2.33556 0.890524 2.1072C1.11889 1.87883 1.42783 1.75083 1.75 1.75083H9.91667L13.4167 5.25083V11.0343C13.4167 11.3565 13.2887 11.6654 13.0603 11.8938C12.832 12.1222 12.5055 12.25 12.25 12.25Z" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                <span>Sauvegarder en liste</span>
              </button>
            </div>

            <!-- Selected Entities with Checkboxes (always visible in dropdown) -->
            <div v-if="selectedEntitiesObjects.length > 0" class="entities-list">
              <div
                v-for="entity in selectedEntitiesObjects"
                :key="'selected-' + entity.id"
                class="search-result-item selected"
                @click="toggleEntitySelection(entity.id)"
              >
                <div class="checkbox-wrapper">
                  <div class="custom-checkbox checked">
                    <svg width="12" height="12" viewBox="0 0 12 12" fill="none">
                      <path d="M10 3L4.5 8.5L2 6" stroke="white" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                  </div>
                </div>
                <span class="entity-label" v-html="highlightMatch(entity.label)"></span>
              </div>
            </div>

            <!-- Search Results with Checkboxes (when searching, excluding already selected) -->
            <div v-if="sourceSearch.length > 0 && nonSelectedFilteredEntities.length > 0" class="entities-list">
              <div
                v-for="entity in nonSelectedFilteredEntities"
                :key="entity.id"
                class="search-result-item"
                @click="toggleEntitySelection(entity.id)"
              >
                <div class="checkbox-wrapper">
                  <div class="custom-checkbox">
                  </div>
                </div>
                <span class="entity-label" v-html="highlightMatch(entity.label)"></span>
              </div>
            </div>
          </div>

          <!-- Categories Section (always visible, filtered when searching) -->
          <div class="categories-section">
            <template v-for="category in sourceCategories" :key="category.id">
              <div class="source-category-wrapper">
                <button
                  class="source-category"
                  @click="toggleSourceCategory(category)"
                >
                  <span>{{ category.label }}</span>
                  <svg class="category-chevron" :class="{ expanded: shouldShowCategoryItems(category) }" width="18" height="18" viewBox="0 0 18 18" fill="none">
                    <path d="M4.5 6.75L9 11.25L13.5 6.75" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                  </svg>
                </button>
                <div v-if="shouldShowCategoryItems(category)" class="category-items">
                  <button
                    v-for="item in getFilteredCategoryItems(category)"
                    :key="item.id"
                    class="category-item"
                    :class="{ selected: selectedItem === item.id }"
                    @click="selectItem(item.id)"
                    v-html="highlightMatch(item.label)"
                  ></button>
                </div>
              </div>
            </template>
          </div>
        </div>

        <!-- Action Buttons (fixed at bottom) -->
        <div v-if="showSearchActionButtons" class="search-action-buttons">
          <button class="btn-clear" @click="clearSearchSelection">Effacer</button>
          <button class="btn-validate" @click="validateSearchSelection">Valider</button>
        </div>
      </div>

      <div v-if="showSourcesDropdown" class="dropdown-overlay" @click="showSourcesDropdown = false"></div>
    </div>

    <!-- Sujets Search -->
    <div class="topbar-item search-group sujets-dropdown" :class="{ active: showSujetsDropdown }">
      <span
        class="sources-floating-label"
        :class="{ floating: showSujetsDropdown || selectedSubject || keywordTags.length > 0, 'floating-active': showSujetsDropdown }"
        @click="showSujetsDropdown = !showSujetsDropdown"
      >Sujets</span>

      <div class="search-input-wrapper" @click="showSujetsDropdown = true">
        <svg class="search-icon" width="15" height="15" viewBox="0 0 15 15" fill="none">
          <path d="M13.125 13.125L10.4062 10.4062M11.875 6.875C11.875 9.63642 9.63642 11.875 6.875 11.875C4.11358 11.875 1.875 9.63642 1.875 6.875C1.875 4.11358 4.11358 1.875 6.875 1.875C9.63642 1.875 11.875 4.11358 11.875 6.875Z" stroke="#595959" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>

        <!-- Keyword Tags in search bar -->
        <div v-if="keywordTags.length > 0 && !showSujetsDropdown" class="search-bar-tags">
          <div
            v-for="(tag, index) in keywordTags"
            :key="tag.id"
            class="entity-tag"
          >
            <span class="entity-tag-label">{{ tag.label }}</span>
            <button class="entity-tag-remove" @click.stop="removeKeyword(index)">
              <svg width="8" height="8" viewBox="0 0 8 8" fill="none">
                <path d="M7 1L1 7" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M1 1L7 7" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </button>
          </div>
        </div>

        <!-- Selected subject label -->
        <span v-if="selectedSubject && !showSujetsDropdown && keywordTags.length === 0" class="selected-item-label">{{ selectedSubjectLabel }}</span>

        <input
          v-show="(keywordTags.length === 0 && !selectedSubject) || showSujetsDropdown"
          ref="subjectSearchInput"
          type="text"
          v-model="subjectSearch"
          :placeholder="showSujetsDropdown ? '' : 'Chercher un mot-clé, une thématique ...'"
          @focus="showSujetsDropdown = true"
          @keydown="handleSubjectKeydown"
        />
      </div>

      <button v-if="selectedSubject && !showSujetsDropdown" class="clear-selection-btn" @click.stop="clearSubjectSelection">
        <svg width="15" height="15" viewBox="0 0 15 15" fill="none">
          <path d="M11.25 3.75L3.75 11.25" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M3.75 3.75L11.25 11.25" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </button>

      <!-- Sujets Panel - Liste simple -->
      <div v-if="showSujetsDropdown" class="sources-categories-panel">
        <div class="panel-scrollable-content">
          <!-- Recherches récentes -->
          <div v-if="recentSearches.length > 0 && !subjectSearch" class="subject-items-list recent-searches-section">
            <div class="subject-category-header">Recherches récentes</div>
            <button
              v-for="(search, index) in recentSearches"
              :key="'recent-' + index"
              class="category-item recent-search-item"
              @click="addKeyword(search)"
            >
              <svg class="recent-icon" width="14" height="14" viewBox="0 0 14 14" fill="none">
                <path d="M7 3.5V7L9.33333 8.16667M12.8333 7C12.8333 10.2217 10.2217 12.8333 7 12.8333C3.77834 12.8333 1.16667 10.2217 1.16667 7C1.16667 3.77834 3.77834 1.16667 7 1.16667C10.2217 1.16667 12.8333 3.77834 12.8333 7Z" stroke="#8C8C8C" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              {{ search }}
            </button>
          </div>

          <div class="subject-items-list">
            <div class="subject-category-header">Queries</div>
            <template v-if="filteredSubjectItems.length > 0">
              <button
                v-for="item in filteredSubjectItems"
                :key="item.id"
                class="category-item"
                :class="{ selected: selectedSubject === item.id }"
                @click="selectSubject(item.id)"
                v-html="highlightSubjectMatch(item.label)"
              ></button>
            </template>
            <div v-else class="no-results">
              <span>Aucune query</span>
            </div>
          </div>
        </div>
      </div>

      <div v-if="showSujetsDropdown" class="dropdown-overlay" @click="showSujetsDropdown = false"></div>
    </div>

    <!-- Date Picker -->
    <div class="topbar-item date-picker-wrapper" :class="{ compact: compactDate, active: showDatePicker }">
      <button class="date-picker-trigger" @click="openDatePicker">
        <span v-if="!compactDate" class="date-range">{{ formatDateRange() }}</span>
        <svg width="15" height="15" viewBox="0 0 15 15" fill="none">
          <path d="M10 1.25V3.75M5 1.25V3.75M1.875 6.25H13.125M3.125 2.5H11.875C12.5654 2.5 13.125 3.05964 13.125 3.75V12.5C13.125 13.1904 12.5654 13.75 11.875 13.75H3.125C2.43464 13.75 1.875 13.1904 1.875 12.5V3.75C1.875 3.05964 2.43464 2.5 3.125 2.5Z" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </button>

      <!-- Date Picker Dropdown Panel -->
      <div v-if="showDatePicker" class="date-picker-panel">
        <!-- Left Sidebar - Pre-defined periods -->
        <div class="date-picker-sidebar">
          <button
            v-for="period in periodOptions"
            :key="period.id"
            class="period-option"
            :class="{ active: selectedPeriod === period.id }"
            @click="selectPeriod(period)"
          >
            {{ period.label }}
          </button>
        </div>

        <!-- Right Content - Calendar -->
        <div class="date-picker-content">

          <!-- Calendar Navigation -->
          <div class="calendar-header">
            <button class="nav-btn" @click="prevMonth">
              <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
                <path d="M11.25 13.5L6.75 9L11.25 4.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </button>
            <div class="months-display">
              <span class="month-name">{{ getMonthName(leftMonth) }}</span>
              <span class="month-name">{{ getMonthName(rightMonth) }}</span>
            </div>
            <button class="nav-btn" @click="nextMonth">
              <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
                <path d="M6.75 13.5L11.25 9L6.75 4.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </button>
          </div>

          <!-- Calendars Grid -->
          <div class="calendars-container">
            <!-- Left Calendar -->
            <div class="calendar" @mouseleave="onDateLeave">
              <div class="calendar-weekdays">
                <span v-for="day in weekDays" :key="day" class="weekday">{{ day }}</span>
              </div>
              <div class="calendar-days">
                <button
                  v-for="(date, index) in getDaysInMonth(leftMonth)"
                  :key="'left-' + index"
                  class="calendar-day"
                  :class="{
                    empty: !date,
                    'in-range': isDateInRange(date),
                    'in-range-preview': isDateInRangePreview(date),
                    'start-date': isStartDate(date),
                    'end-date': isEndDate(date),
                    'has-range': hasRangeBand(date),
                    'selecting': isSelectingRange
                  }"
                  :disabled="!date"
                  @click="selectDate(date)"
                  @mouseenter="onDateHover(date)"
                >
                  {{ date ? date.getDate() : '' }}
                </button>
              </div>
            </div>

            <!-- Right Calendar -->
            <div class="calendar" @mouseleave="onDateLeave">
              <div class="calendar-weekdays">
                <span v-for="day in weekDays" :key="day" class="weekday">{{ day }}</span>
              </div>
              <div class="calendar-days">
                <button
                  v-for="(date, index) in getDaysInMonth(rightMonth)"
                  :key="'right-' + index"
                  class="calendar-day"
                  :class="{
                    empty: !date,
                    'in-range': isDateInRange(date),
                    'in-range-preview': isDateInRangePreview(date),
                    'start-date': isStartDate(date),
                    'end-date': isEndDate(date),
                    'has-range': hasRangeBand(date),
                    'selecting': isSelectingRange
                  }"
                  :disabled="!date"
                  @click="selectDate(date)"
                  @mouseenter="onDateHover(date)"
                >
                  {{ date ? date.getDate() : '' }}
                </button>
              </div>
            </div>
          </div>

          <!-- Action Buttons -->
          <div class="date-picker-actions">
            <button class="btn-effacer" @click="clearDates">Effacer</button>
            <div class="action-buttons-right">
              <button class="btn-annuler" @click="cancelDatePicker">Annuler</button>
              <button class="btn-valider" @click="validateDates">Valider</button>
            </div>
          </div>
        </div>
      </div>

      <div v-if="showDatePicker" class="dropdown-overlay" @click="closeDatePicker"></div>
    </div>
  </div>
</template>

<style scoped>
.topbar {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 20px 30px;
}

.topbar-item {
  background: var(--color-white);
  border-radius: 8px;
  height: 40px;
  display: flex;
  align-items: center;
}

/* Corpus Dropdown */
.corpus-dropdown {
  position: relative;
  min-width: 160px;
  border: 1px solid #EAEAEA;
}

.dropdown-trigger {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  height: 100%;
  padding: 10px;
  border: none;
  background: transparent;
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
  font-family: 'Roboto', sans-serif;
  color: var(--color-black);
}

.chevron {
  transition: transform 0.2s ease;
  color: var(--color-black);
}

.chevron.rotated {
  transform: rotate(180deg);
}

.dropdown-menu {
  position: absolute;
  top: calc(100% + 5px);
  left: 0;
  width: 160px;
  background: var(--color-white);
  border-radius: 8px;
  border: 1px solid #E5E5E5;
  z-index: 100;
  overflow: hidden;
}

.corpus-menu {
  padding: 5px;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.corpus-search {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 5px;
  height: 28px;
  padding: 0 8px;
  border: 1px solid #EAEAEA;
  border-radius: 6px;
}

.corpus-search input {
  width: 100px;
  min-width: 0;
  border: none;
  outline: none;
  font-size: 14px;
  color: var(--color-black);
  background: transparent;
  font-family: 'Roboto', sans-serif;
}

.corpus-search svg {
  flex-shrink: 0;
}

.corpus-options {
  max-height: 200px;
  overflow-y: auto;
}

.dropdown-option {
  display: block;
  width: 100%;
  height: 30px;
  padding: 0 8px;
  border: none;
  background: var(--color-white);
  text-align: left;
  cursor: pointer;
  font-size: 14px;
  color: var(--color-black);
  line-height: 30px;
  border-radius: 4px;
  transition: background-color 0.15s ease;
}

.dropdown-option:hover {
  background-color: var(--color-background);
}

.dropdown-option.active {
  background-color: var(--color-blue);
  color: white;
}

/* Search Groups */
.search-group {
  position: relative;
  flex: 1;
  min-width: 280px;
  border: 1px solid #EAEAEA;
  transition: border-color 0.2s ease;
  padding: 5px 10px;
}

.search-group.active {
  border-color: var(--color-blue);
}

.search-input-wrapper {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 10px;
  overflow-x: auto;
  scrollbar-width: none; /* Firefox */
  -ms-overflow-style: none; /* IE/Edge */
}

.search-input-wrapper::-webkit-scrollbar {
  display: none; /* Chrome, Safari, Opera */
}

.search-icon {
  flex-shrink: 0;
  opacity: 0.5;
}

.search-input-wrapper input {
  flex: 1;
  border: none;
  outline: none;
  font-size: 16px;
  color: #595959;
  background: transparent;
  font-family: 'Roboto', sans-serif;
}

.search-input-wrapper input::placeholder {
  color: var(--color-black);
  opacity: 0.5;
}

.search-separator {
  width: 1px;
  height: 20px;
  background: #595959;
  opacity: 0.3;
  flex-shrink: 0;
}

.filter-button {
  height: 32px;
  padding: 0 16px;
  border: none;
  background: transparent;
  color: var(--color-black);
  font-size: 16px;
  font-family: 'Roboto', sans-serif;
  cursor: pointer;
  border-left: 1px solid #e0e0e0;
  transition: color 0.15s ease;
}

.filter-button:hover {
  color: var(--color-blue);
}

/* Sources Floating Label */
.sources-floating-label {
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  padding: 0 10px;
  height: 20px;
  display: flex;
  align-items: center;
  font-size: 16px;
  font-family: 'Roboto', sans-serif;
  color: #595959;
  border-left: 1px solid #8D8D8D;
  background: transparent;
  transition: all 0.2s ease;
  z-index: 102;
}

.sources-floating-label.floating {
  right: auto;
  left: 10px;
  top: 0;
  padding: 0;
  height: auto;
  font-size: 12px;
  font-weight: 500;
  color: #8D8D8D;
  background: #ffffff;
  border-left: none;
}

.sources-floating-label.floating-active {
  color: var(--color-blue);
}

.selected-item-label {
  flex: 1;
  font-size: 16px;
  font-family: 'Roboto', sans-serif;
  color: var(--color-black);
}

.clear-selection-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  border: none;
  background: transparent;
  cursor: pointer;
  color: var(--color-black);
}

.clear-selection-btn:hover {
  color: var(--color-blue);
}

/* Sources Categories Panel */
.sources-categories-panel {
  position: absolute;
  top: calc(100% + 5px);
  left: 0;
  right: 0;
  background: var(--color-white);
  border: 1px solid #EAEAEA;
  border-radius: 8px;
  padding: 5px;
  z-index: 101;
  max-height: 350px;
  display: flex;
  flex-direction: column;
}

.sources-categories-panel.has-action-buttons {
  padding-bottom: 60px;
}

.panel-scrollable-content {
  flex: 1;
  overflow-y: auto;
  max-height: 280px;
}

.source-category {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  padding: 8px;
  border: none;
  background: var(--color-white);
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  color: var(--color-blue);
  font-family: 'Roboto', sans-serif;
  text-align: left;
  transition: background-color 0.15s ease;
}

.source-category:hover {
  background-color: var(--color-background);
  border-radius: 4px;
}

.category-chevron {
  color: var(--color-blue);
  transition: transform 0.2s ease;
}

.category-chevron.expanded {
  transform: rotate(180deg);
}

.source-category-wrapper {
  display: flex;
  flex-direction: column;
}

.category-items {
  display: flex;
  flex-direction: column;
}

.category-item {
  display: block;
  width: 100%;
  padding: 8px 8px;
  border: none;
  background: transparent;
  text-align: left;
  cursor: pointer;
  font-size: 14px;
  color: var(--color-black);
  font-family: 'Roboto', sans-serif;
  border-radius: 4px;
  transition: background-color 0.15s ease;
}

.category-item:hover {
  background-color: var(--color-background);
}

.category-item.selected {
  background-color: var(--color-blue);
  color: var(--color-white);
}

/* Entities Section */
.entities-section {
  border-bottom: 1px solid #EAEAEA;
  padding-bottom: 5px;
  margin-bottom: 5px;
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px;
  background: var(--color-white);
}

.section-title {
  font-size: 14px;
  font-weight: 500;
  color: var(--color-blue);
  font-family: 'Roboto', sans-serif;
}

.save-list-link {
  display: flex;
  align-items: center;
  gap: 6px;
  border: none;
  background: transparent;
  cursor: pointer;
  color: #8D8D8D;
  font-size: 13px;
  font-family: 'Roboto', sans-serif;
  padding: 4px 8px;
  border-radius: 4px;
  transition: all 0.15s ease;
}

.save-list-link:hover {
  color: var(--color-blue);
  background-color: #EAEFFD;
}

/* Entity Tags in Search Bar */
.search-bar-tags {
  display: flex;
  flex-wrap: nowrap;
  gap: 6px;
  flex: 1;
  min-width: 0;
  overflow-x: auto;
}

/* Selected Entities Tags in Dropdown */
.selected-entities-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  padding: 8px;
}

.entity-tag {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 4px 8px;
  background: #F2F2F2;
  border-radius: 4px;
  font-size: 14px;
  font-family: 'Roboto', sans-serif;
  color: var(--color-black);
  white-space: nowrap;
}

.entity-tag-label {
  max-width: 150px;
  overflow: hidden;
  text-overflow: ellipsis;
}

.entity-tag-remove {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 14px;
  height: 14px;
  padding: 0;
  border: none;
  background: transparent;
  cursor: pointer;
  color: #595959;
  transition: color 0.15s ease;
  flex-shrink: 0;
}

.entity-tag-remove:hover {
  color: var(--color-black);
}

.entities-list {
  display: flex;
  flex-direction: column;
  max-height: 200px;
  overflow-y: auto;
}

/* Categories Section */
.categories-section {
  padding-top: 5px;
}

.search-result-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 8px;
  cursor: pointer;
  border-radius: 4px;
  transition: background-color 0.15s ease;
}

.search-result-item:hover {
  background-color: #EAEFFD;
}

.search-result-item.selected {
  background-color: #EAEFFD;
}

.checkbox-wrapper {
  flex-shrink: 0;
}

.custom-checkbox {
  width: 14px;
  height: 14px;
  border: 1.5px solid #8D8D8D;
  border-radius: 3px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.15s ease;
}

.custom-checkbox.checked {
  background-color: var(--color-blue);
  border-color: var(--color-blue);
}

.entity-label {
  font-size: 14px;
  color: var(--color-black);
  font-family: 'Roboto', sans-serif;
}

/* Highlight matching text */
:deep(.highlight-match) {
  font-weight: 600;
}

/* Search Action Buttons */
.search-action-buttons {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  gap: 10px;
  padding: 10px;
  background: var(--color-white);
  border-top: 1px solid #EAEAEA;
  border-radius: 0 0 8px 8px;
}

.btn-clear,
.btn-validate {
  flex: 1;
  height: 36px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  font-family: 'Roboto', sans-serif;
  cursor: pointer;
  transition: all 0.15s ease;
}

.btn-clear {
  background: transparent;
  border: 1px solid var(--color-blue);
  color: var(--color-blue);
}

.btn-clear:hover {
  background-color: #EAEFFD;
}

.btn-validate {
  background: var(--color-blue);
  border: none;
  color: white;
}

.btn-validate:hover {
  background: #1a4fd6;
}

.dropdown-overlay {
  position: fixed;
  inset: 0;
  z-index: 100;
}

/* Date Picker */
.date-picker-wrapper {
  position: relative;
  border: 1px solid #EAEAEA;
  transition: border-color 0.2s ease;
}

.date-picker-wrapper.active {
  border-color: var(--color-blue);
}

.date-picker-trigger {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0 16px ;
  height: 100%;
  border: none;
  background: transparent;
  cursor: pointer;
  font-family: 'Roboto', sans-serif;
  font-weight: 500;
}

.date-picker-wrapper.compact .date-picker-trigger {
  padding: 0 12px;
  justify-content: center;
}

.date-range {
  font-size: 16px;
  color: var(--color-black);
  white-space: nowrap;
}

.date-picker-trigger svg {
  color: var(--color-black);
}

/* Date Picker Panel */
.date-picker-panel {
  position: absolute;
  top: calc(100% + 5px);
  right: 0;
  display: flex;
  background: var(--color-white);
  border: 1px solid #EAEAEA;
  border-radius: 8px;
  z-index: 101;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

/* Sidebar - Pre-defined periods */
.date-picker-sidebar {
  display: flex;
  flex-direction: column;
  padding: 10px;
  border-right: 1px solid #EAEAEA;
  min-width: 160px;
}

.period-option {
  display: flex;
  align-items: center;
  height: 36px;
  padding: 12px;
  border: none;
  background: transparent;
  font-size: 14px;
  font-family: 'Roboto', sans-serif;
  color: var(--color-black);
  cursor: pointer;
  border-radius: 6px;
  transition: all 0.15s ease;
  text-align: left;
}

.period-option:hover {
  background-color: #F5F5F5;
}

.period-option.active {
  background-color: var(--color-blue);
  color: white;
}

/* Date Picker Content */
.date-picker-content {
  display: flex;
  flex-direction: column;
  padding: 20px 30px;
  min-width: 500px;
}

/* Date Range Display */
.date-range-display {
  display: flex;
  justify-content: center;
  margin-bottom: 15px;
}

.date-input-group {
  display: flex;
  align-items: center;
  gap: 10px;
}

.date-input {
  width: 110px;
  height: 36px;
  padding: 0 12px;
  border: 1px solid #EAEAEA;
  border-radius: 6px;
  font-size: 14px;
  font-family: 'Roboto', sans-serif;
  color: var(--color-black);
  text-align: center;
  background: var(--color-white);
}

.date-input:focus {
  outline: none;
  border-color: var(--color-blue);
}

.date-separator {
  color: #8D8D8D;
  font-size: 14px;
}

/* Calendar Header */
.calendar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 15px;
}

.nav-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border: none;
  background: transparent;
  cursor: pointer;
  color: var(--color-black);
  border-radius: 6px;
  transition: background-color 0.15s ease;
}

.nav-btn:hover {
  background-color: #F5F5F5;
}

.months-display {
  display: flex;
  gap: 80px;
}

.month-name {
  font-size: 14px;
  font-weight: 500;
  font-family: 'Roboto', sans-serif;
  color: var(--color-black);
  text-transform: capitalize;
}

/* Calendars Container */
.calendars-container {
  display: flex;
  gap: 30px;
}

.calendar {
  flex: 1;
}

.calendar-weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  margin-bottom: 8px;
}

.weekday {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 30px;
  font-size: 12px;
  font-weight: 500;
  font-family: 'Roboto', sans-serif;
  color: #8D8D8D;
}

.calendar-days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 0;
}

.calendar-day {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  border: none;
  background: transparent;
  font-size: 14px;
  font-family: 'Roboto', sans-serif;
  color: var(--color-black);
  cursor: pointer;
  transition: color 0.15s ease;
}

/* Circle behind the date number */
.calendar-day::before {
  content: '';
  position: absolute;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: transparent;
  transition: background-color 0.15s ease, transform 0.15s ease;
  z-index: 0;
}

/* Range background - full width rectangle */
.calendar-day::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  height: 32px;
  transform: translateY(-50%);
  background: transparent;
  transition: background-color 0.15s ease;
  z-index: -1;
}

.calendar-day:hover:not(.empty):not(:disabled)::before {
  background-color: #F0F0F0;
  transform: scale(1.05);
}

.calendar-day.empty {
  cursor: default;
}

.calendar-day.empty::before,
.calendar-day.empty::after {
  display: none;
}

/* In range - light blue background band */
.calendar-day.in-range::after {
  background-color: #EAEFFD;
}

.calendar-day.in-range:not(.start-date):not(.end-date)::before {
  background-color: transparent;
}

/* Start date - blue circle */
.calendar-day.start-date::before {
  background-color: var(--color-blue);
}

.calendar-day.start-date {
  color: white;
}

/* Start date with range - add right half band */
.calendar-day.start-date.has-range::after {
  left: 50%;
  background-color: #EAEFFD;
}

/* End date - blue circle */
.calendar-day.end-date::before {
  background-color: var(--color-blue);
}

.calendar-day.end-date {
  color: white;
}

/* End date with range - add left half band */
.calendar-day.end-date.has-range::after {
  right: 50%;
  left: 0;
  background-color: #EAEFFD;
}

/* When start and end are the same - just circle, no band */
.calendar-day.start-date.end-date::after {
  background-color: transparent;
}

/* Selecting mode - pulsing effect on start date */
.calendar-day.start-date.selecting::before {
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(42, 96, 232, 0.4);
  }
  50% {
    transform: scale(1.05);
    box-shadow: 0 0 0 6px rgba(42, 96, 232, 0);
  }
}

/* Hover on start/end dates */
.calendar-day.start-date:hover::before,
.calendar-day.end-date:hover::before {
  transform: scale(1.08);
  background-color: #1a4fd6;
}

/* Preview range while selecting */
.calendar-day.in-range-preview::after {
  background-color: #F0F5FF;
}

.calendar-day.in-range-preview:not(.start-date)::before {
  background-color: rgba(42, 96, 232, 0.1);
}

/* Date Picker Actions */
.date-picker-actions {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 30px;
}

.action-buttons-right {
  display: flex;
  gap: 10px;
}

.btn-effacer,
.btn-annuler,
.btn-valider {
  height: 36px;
  padding: 0 20px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  font-family: 'Roboto', sans-serif;
  cursor: pointer;
  transition: all 0.15s ease;
}

.btn-effacer {
  background: transparent;
  border: none;
  color: #8D8D8D;
}

.btn-effacer:hover {
  color: var(--color-black);
}

.btn-annuler {
  background: transparent;
  border: 1px solid #EAEAEA;
  color: var(--color-black);
}

.btn-annuler:hover {
  border-color: #CCCCCC;
}

.btn-valider {
  background: var(--color-blue);
  border: none;
  color: white;
}

.btn-valider:hover {
  background: #1a4fd6;
}

/* Subject Category Header */
.subject-category-header {
  padding: 8px;
  font-size: 14px;
  font-weight: 500;
  color: var(--color-blue);
  font-family: 'Roboto', sans-serif;
}

.subject-items-list {
  display: flex;
  flex-direction: column;
}

.no-results {
  padding: 12px 8px;
  font-size: 14px;
  color: #8D8D8D;
  font-family: 'Roboto', sans-serif;
}

.no-results span {
  font-style: italic;
}

.add-keyword-btn {
  display: block;
  width: 100%;
  margin-top: 8px;
  padding: 8px 12px;
  border: 1px dashed var(--color-blue);
  background: transparent;
  color: var(--color-blue);
  font-size: 14px;
  font-family: 'Roboto', sans-serif;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.15s ease;
  font-style: normal;
}

.add-keyword-btn:hover {
  background: #EAEFFD;
}

/* Recent Searches Section */
.recent-searches-section {
  margin-bottom: 10px;
}

.recent-search-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.recent-icon {
  flex-shrink: 0;
}
</style>
