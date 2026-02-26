<script setup>
import { ref, computed, nextTick, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  sidebarExpanded: {
    type: Boolean,
    default: false
  }
})

// Date picker open state (declared early for closeAllDropdowns)
const dpIsOpen = ref(false)

// Close all dropdowns
const closeAllDropdowns = (except) => {
  const all = { corpus: showCorpusDropdown, actors: showActorsDropdown, query: showQueryDropdown, language: showLanguageDropdown }
  Object.entries(all).forEach(([key, r]) => {
    if (key !== except) r.value = false
  })
  if (except !== 'datepicker') dpIsOpen.value = false
}

// Toggle dropdowns (close others before opening)
const toggleDropdown = (name) => {
  const refs = { corpus: showCorpusDropdown, actors: showActorsDropdown }
  const isOpen = refs[name].value
  closeAllDropdowns(name)
  refs[name].value = !isOpen

  if (name === 'corpus' && !isOpen) {
    nextTick(() => {
      const el = document.querySelector('.menu-option.active')
      if (el) el.scrollIntoView({ block: 'nearest' })
    })
  }

  if (name === 'actors' && !isOpen) {
    if (selectedActor.value) {
      actorsCategories.value.forEach(cat => {
        cat.expanded = cat.items.some(item => getItemName(item) === selectedActor.value)
      })
      nextTick(() => {
        const el = document.querySelector('.menu-option.active')
        if (el) el.scrollIntoView({ block: 'nearest' })
      })
    }
  }
}

// Corpus
const selectedCorpus = ref('France')
const corpusOptions = ['Argentina', 'Bulgaria', 'Colombia', 'Czechia', 'Denmark', 'EU', 'France', 'Germany', 'Greece', 'Italy', 'Japan']
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

// Highlight matching text
const highlightMatch = (text, search) => {
  if (!search) return text
  const regex = new RegExp(`(${search.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})`, 'gi')
  return text.replace(regex, '<span class="highlight-match">$1</span>')
}

// Actors
const showActorsDropdown = ref(false)
const actorsSearch = ref('')

const actorsEntities = ['Entité 1', 'Entité 2', 'Entité 3', 'Entité 4', 'Entité 5', 'Entité 6', 'Entité 7', 'Entité 8', 'Entité 9', 'Entité 10']

const filteredEntities = () => {
  if (!actorsSearch.value) return actorsEntities.filter(e => checkedEntities.value.includes(e))
  return actorsEntities.filter(item =>
    item.toLowerCase().includes(actorsSearch.value.toLowerCase())
  )
}

const actorsCategories = ref([
  { id: 'entities', label: "Listes d'entités", expanded: false, items: [
    'Liste 1', 'Liste 2', 'Liste 3', 'Liste 4', 'Liste 5', 'Liste 6', 'Liste 7', 'Liste 8', 'Liste 9', 'Liste 10'
  ]},
  { id: 'communities', label: 'Communautés', expanded: false, items: [
    { name: 'Communauté 1', color: '#01A5F8' },
    { name: 'Communauté 2', color: '#BA77BA' },
    { name: 'Communauté 3', color: '#FF7C80' },
    { name: 'Communauté 4', color: '#3AAED1' },
    { name: 'Communauté 5', color: '#000000' },
    { name: 'Communauté 6', color: '#1B4B7A' },
    { name: 'Communauté 7', color: '#EBCB81' },
    { name: 'Communauté 8', color: '#5890C4' },
    { name: 'Communauté 9', color: '#A8DFEA' },
    { name: 'Communauté 10', color: '#70E1C4' }
  ]},
  { id: 'subcommunities', label: 'Sous-communautés', expanded: false, items: [
    'Sous-communauté 1', 'Sous-communauté 2', 'Sous-communauté 3', 'Sous-communauté 4', 'Sous-communauté 5', 'Sous-communauté 6', 'Sous-communauté 7', 'Sous-communauté 8', 'Sous-communauté 9', 'Sous-communauté 10'
  ]},
  { id: 'stakeholders', label: 'Stakeholders', expanded: false, items: [
    'Stakeholder 1', 'Stakeholder 2', 'Stakeholder 3', 'Stakeholder 4', 'Stakeholder 5', 'Stakeholder 6', 'Stakeholder 7', 'Stakeholder 8', 'Stakeholder 9', 'Stakeholder 10'
  ]}
])

const selectedActor = ref(null)
const checkedEntities = ref([])

const getItemName = (item) => typeof item === 'string' ? item : item.name

const selectActor = (item) => {
  checkedEntities.value = []
  selectedActor.value = getItemName(item)
  showActorsDropdown.value = false
}

const toggleEntity = (entity) => {
  selectedActor.value = null
  const idx = checkedEntities.value.indexOf(entity)
  if (idx === -1) {
    checkedEntities.value.push(entity)
  } else {
    checkedEntities.value.splice(idx, 1)
  }
}

const removeEntity = (entity) => {
  checkedEntities.value = checkedEntities.value.filter(e => e !== entity)
}

const resetEntities = () => {
  checkedEntities.value = []
}

const clearActor = () => {
  selectedActor.value = null
  checkedEntities.value = []
}

const hasSelection = () => selectedActor.value || checkedEntities.value.length > 0

const filteredActorsItems = (items) => {
  if (!actorsSearch.value) return items
  return items.filter(item =>
    getItemName(item).toLowerCase().includes(actorsSearch.value.toLowerCase())
  )
}

const isCategoryVisible = (category) => {
  if (actorsSearch.value) return filteredActorsItems(category.items).length > 0
  return category.expanded
}

const toggleActorsCategory = (category) => {
  category.expanded = !category.expanded
}

// Topic bar
const topicSearch = ref('')
const topicSearchFocused = ref(false)

const submitTopicSearch = (e) => {
  e.target.blur()
}

// Query dropdown (inside Topic bar)
const showQueryDropdown = ref(false)
const selectedQuery = ref(null)
const querySearch = ref('')
const queryOptions = ['Query 1', 'Query 2', 'Query 3', 'Query 4', 'Query 5', 'Query 6', 'Query 7', 'Query 8', 'Query 9', 'Query 10']

const filteredQueryOptions = () => {
  if (!querySearch.value) return queryOptions
  return queryOptions.filter(option =>
    option.toLowerCase().includes(querySearch.value.toLowerCase())
  )
}

const selectQuery = (query) => {
  selectedQuery.value = query
  showQueryDropdown.value = false
  querySearch.value = ''
}

const clearQuery = () => {
  selectedQuery.value = null
}

const toggleQueryDropdown = () => {
  const isOpen = showQueryDropdown.value
  closeAllDropdowns('query')
  showQueryDropdown.value = !isOpen

  if (showQueryDropdown.value && selectedQuery.value) {
    nextTick(() => {
      const el = document.querySelector('.topic-query-menu .menu-option.active')
      if (el) el.scrollIntoView({ block: 'nearest' })
    })
  }
}

// Language
const showLanguageDropdown = ref(false)
const selectedLanguage = ref('FR')
const languageOptions = [
  { code: 'EN', label: 'Anglais', flag: new URL('../../assets/flags/Anglais.svg', import.meta.url).href },
  { code: 'FR', label: 'Français', flag: new URL('../../assets/flags/Français.svg', import.meta.url).href },
  { code: 'DE', label: 'Allemand', flag: new URL('../../assets/flags/Allemand.svg', import.meta.url).href },
  { code: 'IT', label: 'Italien', flag: new URL('../../assets/flags/Italien.svg', import.meta.url).href },
  { code: 'JP', label: 'Japonnais', flag: new URL('../../assets/flags/Japonais.svg', import.meta.url).href },
  { code: 'ES', label: 'Espagnol', flag: new URL('../../assets/flags/Espagnol.svg', import.meta.url).href }
]

const toggleLanguageDropdown = () => {
  const isOpen = showLanguageDropdown.value
  closeAllDropdowns('language')
  showLanguageDropdown.value = !isOpen
}

const selectedLanguageOption = computed(() => languageOptions.find(l => l.code === selectedLanguage.value))

const selectLanguage = (lang) => {
  selectedLanguage.value = lang.code
  showLanguageDropdown.value = false
}

const clearLanguage = () => {
  selectedLanguage.value = null
}

// Date Picker
const padTwo = (n) => String(n).padStart(2, '0')
const toDateStr = (d) => `${padTwo(d.getDate())}/${padTwo(d.getMonth() + 1)}/${d.getFullYear()}`
const now = new Date()
const threeMonthsAgo = new Date(now.getFullYear(), now.getMonth() - 3, now.getDate())
const dateStart = ref(toDateStr(threeMonthsAgo))
const dateEnd = ref(toDateStr(now))

const dpHoverDate = ref(null)
const dpSelecting = ref(false)
const dpActiveShortcut = ref(null)

const dpInternalStart = ref(null)
const dpInternalEnd = ref(null)
const dpTempStart = ref(null)
const dpTempEnd = ref(null)

const dpLeftMonth = ref(new Date().getMonth())
const dpLeftYear = ref(new Date().getFullYear())
const dpRightMonth = ref((new Date().getMonth() + 1) % 12)
const dpRightYear = ref(new Date().getMonth() === 11 ? new Date().getFullYear() + 1 : new Date().getFullYear())

const dpMonthKey = (year, month) => year * 12 + month

const dpDayNames = ['Lun', 'Mar', 'Mer', 'Jeu', 'Ven', 'Sam', 'Dim']
const dpMonthNames = [
  'Janvier', 'Février', 'Mars', 'Avril', 'Mai', 'Juin',
  'Juillet', 'Août', 'Septembre', 'Octobre', 'Novembre', 'Décembre'
]

const dpParseDate = (str) => {
  if (!str) return null
  const parts = str.split('/')
  if (parts.length !== 3) return null
  const [d, m, y] = parts.map(Number)
  return new Date(y, m - 1, d)
}

const dpFormatDate = (date) => {
  if (!date) return ''
  const d = String(date.getDate()).padStart(2, '0')
  const m = String(date.getMonth() + 1).padStart(2, '0')
  const y = date.getFullYear()
  return `${d}/${m}/${y}`
}

const dpStepMonth = (month, year, delta) => {
  let m = month + delta
  let y = year
  if (m < 0) { m = 11; y-- }
  else if (m > 11) { m = 0; y++ }
  return { month: m, year: y }
}

const dpInitDates = () => {
  dpInternalStart.value = dpParseDate(dateStart.value)
  dpInternalEnd.value = dpParseDate(dateEnd.value)
  dpTempStart.value = dpInternalStart.value
  dpTempEnd.value = dpInternalEnd.value
  if (dpInternalStart.value) {
    dpLeftMonth.value = dpInternalStart.value.getMonth()
    dpLeftYear.value = dpInternalStart.value.getFullYear()
    if (dpInternalEnd.value && dpMonthKey(dpInternalEnd.value.getFullYear(), dpInternalEnd.value.getMonth()) > dpMonthKey(dpLeftYear.value, dpLeftMonth.value)) {
      dpRightMonth.value = dpInternalEnd.value.getMonth()
      dpRightYear.value = dpInternalEnd.value.getFullYear()
    } else {
      const r = dpStepMonth(dpLeftMonth.value, dpLeftYear.value, 1)
      dpRightMonth.value = r.month
      dpRightYear.value = r.year
    }
  }
}
dpInitDates()

const dpDisplayValue = computed(() => {
  const isOpen = dpIsOpen.value
  const start = isOpen ? dpTempStart.value : dpInternalStart.value
  const end = isOpen ? dpTempEnd.value : dpInternalEnd.value
  const hasTime = isOpen ? dpShowTime.value : dpConfirmedShowTime.value
  const tStart = isOpen ? dpTimeStart.value : dpConfirmedTimeStart.value
  const tEnd = isOpen ? dpTimeEnd.value : dpConfirmedTimeEnd.value
  if (start && end) {
    if (dpDateToKey(start) === dpDateToKey(end)) {
      if (hasTime) return `${dpFormatDate(start)} ${tStart} - ${tEnd}`
      return dpFormatDate(start)
    }
    return `${dpFormatDate(start)} - ${dpFormatDate(end)}`
  }
  if (start) {
    if (hasTime) return `${dpFormatDate(start)} ${tStart} - ${tEnd}`
    return dpFormatDate(start)
  }
  return ''
})

const dpHasChanged = computed(() => {
  if (!dpIsOpen.value) return false
  const tempStartKey = dpTempStart.value ? dpDateToKey(dpTempStart.value) : null
  const tempEndKey = dpTempEnd.value ? dpDateToKey(dpTempEnd.value) : null
  const intStartKey = dpInternalStart.value ? dpDateToKey(dpInternalStart.value) : null
  const intEndKey = dpInternalEnd.value ? dpDateToKey(dpInternalEnd.value) : null
  if (tempStartKey !== intStartKey || tempEndKey !== intEndKey) return true
  if (dpShowTime.value !== dpConfirmedShowTime.value) return true
  if (dpShowTime.value && (dpTimeStart.value !== dpConfirmedTimeStart.value || dpTimeEnd.value !== dpConfirmedTimeEnd.value)) return true
  return false
})

const dpLeftPrev = () => {
  const { month, year } = dpStepMonth(dpLeftMonth.value, dpLeftYear.value, -1)
  dpLeftMonth.value = month
  dpLeftYear.value = year
}
const dpLeftNext = () => {
  const { month, year } = dpStepMonth(dpLeftMonth.value, dpLeftYear.value, 1)
  dpLeftMonth.value = month
  dpLeftYear.value = year
  if (dpMonthKey(dpLeftYear.value, dpLeftMonth.value) >= dpMonthKey(dpRightYear.value, dpRightMonth.value)) {
    const r = dpStepMonth(dpLeftMonth.value, dpLeftYear.value, 1)
    dpRightMonth.value = r.month
    dpRightYear.value = r.year
  }
}
const dpRightPrev = () => {
  const { month, year } = dpStepMonth(dpRightMonth.value, dpRightYear.value, -1)
  dpRightMonth.value = month
  dpRightYear.value = year
  if (dpMonthKey(dpRightYear.value, dpRightMonth.value) <= dpMonthKey(dpLeftYear.value, dpLeftMonth.value)) {
    const l = dpStepMonth(dpRightMonth.value, dpRightYear.value, -1)
    dpLeftMonth.value = l.month
    dpLeftYear.value = l.year
  }
}
const dpRightNext = () => {
  const { month, year } = dpStepMonth(dpRightMonth.value, dpRightYear.value, 1)
  dpRightMonth.value = month
  dpRightYear.value = year
}

const dpGetDaysInMonth = (year, month) => new Date(year, month + 1, 0).getDate()
const dpGetFirstDayOfMonth = (year, month) => {
  const day = new Date(year, month, 1).getDay()
  return day === 0 ? 6 : day - 1
}

const dpGenerateCalendarDays = (year, month) => {
  const days = []
  const daysInMonth = dpGetDaysInMonth(year, month)
  const firstDay = dpGetFirstDayOfMonth(year, month)
  const prevMonthVal = month === 0 ? 11 : month - 1
  const prevYearVal = month === 0 ? year - 1 : year
  const daysInPrevMonth = dpGetDaysInMonth(prevYearVal, prevMonthVal)
  for (let i = firstDay - 1; i >= 0; i--) {
    days.push({ day: daysInPrevMonth - i, month: prevMonthVal, year: prevYearVal, isCurrentMonth: false })
  }
  for (let i = 1; i <= daysInMonth; i++) {
    days.push({ day: i, month, year, isCurrentMonth: true })
  }
  const nextMonthVal = month === 11 ? 0 : month + 1
  const nextYearVal = month === 11 ? year + 1 : year
  const totalRows = 42
  const remaining = totalRows - days.length
  for (let i = 1; i <= remaining; i++) {
    days.push({ day: i, month: nextMonthVal, year: nextYearVal, isCurrentMonth: false })
  }
  return days
}

const dpLeftCalendarDays = computed(() => dpGenerateCalendarDays(dpLeftYear.value, dpLeftMonth.value))
const dpRightCalendarDays = computed(() => dpGenerateCalendarDays(dpRightYear.value, dpRightMonth.value))

const dpToDateKey = (year, month, day) => year * 10000 + month * 100 + day
const dpDateToKey = (date) => date ? dpToDateKey(date.getFullYear(), date.getMonth(), date.getDate()) : null

const dpIsToday = (dayObj) => {
  const today = new Date()
  return dayObj.day === today.getDate() && dayObj.month === today.getMonth() && dayObj.year === today.getFullYear()
}

const dpIsFuture = (dayObj) => {
  const today = new Date()
  const todayKey = dpToDateKey(today.getFullYear(), today.getMonth(), today.getDate())
  return dpToDateKey(dayObj.year, dayObj.month, dayObj.day) > todayKey
}

const dpIsSameDay = (dayObj, date) => {
  if (!date) return false
  return dayObj.day === date.getDate() && dayObj.month === date.getMonth() && dayObj.year === date.getFullYear()
}

const dpIsInRange = (dayObj) => {
  if (!dpTempStart.value) return false
  const key = dpToDateKey(dayObj.year, dayObj.month, dayObj.day)
  const startKey = dpDateToKey(dpTempStart.value)
  const endDate = dpSelecting.value ? dpHoverDate.value : dpTempEnd.value
  if (!endDate) return false
  const endKey = dpDateToKey(endDate)
  if (startKey < endKey) return key > startKey && key < endKey
  if (startKey > endKey) return key < startKey && key > endKey
  return false
}

const dpIsHoverEnd = (dayObj) => {
  if (!dpSelecting.value || !dpHoverDate.value) return false
  return dpIsSameDay(dayObj, dpHoverDate.value)
}

const dpGetEffectiveRange = (dayObj) => {
  const start = dpTempStart.value
  const end = dpSelecting.value ? dpHoverDate.value : dpTempEnd.value
  if (!start || !end) return { isStart: dpIsSameDay(dayObj, start), isEnd: false }
  const startKey = dpDateToKey(start)
  const endKey = dpDateToKey(end)
  if (startKey <= endKey) {
    return { isStart: dpIsSameDay(dayObj, start), isEnd: dpIsSameDay(dayObj, end) }
  } else {
    return { isStart: dpIsSameDay(dayObj, end), isEnd: dpIsSameDay(dayObj, start) }
  }
}

const dpSelectDay = (dayObj) => {
  if (!dayObj.isCurrentMonth || dpIsFuture(dayObj)) return
  const date = new Date(dayObj.year, dayObj.month, dayObj.day)
  dpActiveShortcut.value = null
  if (!dpSelecting.value) {
    dpTempStart.value = date
    dpTempEnd.value = null
    dpSelecting.value = true
  } else {
    const startKey = dpDateToKey(dpTempStart.value)
    const endKey = dpDateToKey(date)
    if (startKey <= endKey) {
      dpTempEnd.value = date
    } else {
      dpTempEnd.value = dpTempStart.value
      dpTempStart.value = date
    }
    dpSelecting.value = false
    dpHoverDate.value = null
    if (dpDateToKey(dpTempStart.value) !== dpDateToKey(dpTempEnd.value)) {
      dpShowTime.value = false
    }
  }
}

const dpOnDayHover = (dayObj) => {
  if (dpSelecting.value && dayObj.isCurrentMonth && !dpIsFuture(dayObj)) {
    dpHoverDate.value = new Date(dayObj.year, dayObj.month, dayObj.day)
  }
}

const dpDayClasses = (dayObj) => {
  const classes = ['datepicker-day']
  if (!dayObj.isCurrentMonth || dpIsFuture(dayObj)) {
    classes.push('other-month')
    return classes
  }
  if (dpIsToday(dayObj)) classes.push('today')
  const { isStart, isEnd } = dpGetEffectiveRange(dayObj)
  if (isStart && isEnd) classes.push('range-single')
  else if (isStart) classes.push('range-start')
  else if (isEnd || (dpSelecting.value && dpIsHoverEnd(dayObj))) classes.push('range-end')
  if (dpIsInRange(dayObj)) classes.push('in-range')
  return classes
}

const dpShortcutGroups = [
  { id: 'last-date', items: [
    { label: "Aujourd'hui", id: 'today', getValue: () => { const t = new Date(); return { start: t, end: t } } },
    { label: '1h', id: '1h', showTime: true, getValue: () => { const e = new Date(); const s = new Date(e.getTime() - 3600000); return { start: s, end: e } } },
    { label: '12h', id: '12h', showTime: true, getValue: () => { const e = new Date(); const s = new Date(e.getTime() - 43200000); return { start: s, end: e } } },
    { label: '24h', id: '24h', getValue: () => { const e = new Date(); const s = new Date(); s.setDate(s.getDate() - 1); return { start: s, end: e } } },
    { label: '7 jours', id: '7d', getValue: () => { const e = new Date(); const s = new Date(); s.setDate(s.getDate() - 6); return { start: s, end: e } } },
    { label: '14 jours', id: '14d', getValue: () => { const e = new Date(); const s = new Date(); s.setDate(s.getDate() - 13); return { start: s, end: e } } },
    { label: '30 jours', id: '30d', getValue: () => { const e = new Date(); const s = new Date(); s.setDate(s.getDate() - 29); return { start: s, end: e } } },
    { label: '365 jours', id: '365d', getValue: () => { const e = new Date(); const s = new Date(); s.setDate(s.getDate() - 364); return { start: s, end: e } } },
  ]},
  { id: 'current-date', items: [
    { label: 'Mois en cours', id: 'cur-month', getValue: () => { const n = new Date(); return { start: new Date(n.getFullYear(), n.getMonth(), 1), end: new Date(n.getFullYear(), n.getMonth() + 1, 0) } } },
    { label: 'Année en cours', id: 'cur-year', getValue: () => { const n = new Date(); return { start: new Date(n.getFullYear(), 0, 1), end: new Date(n.getFullYear(), 11, 31) } } },
  ]},
  { id: 'previous-date', items: [
    { label: 'Dernier mois', id: 'last-month', getValue: () => { const n = new Date(); return { start: new Date(n.getFullYear(), n.getMonth() - 1, 1), end: new Date(n.getFullYear(), n.getMonth(), 0) } } },
    { label: 'Dernier trimestre', id: 'last-quarter', getValue: () => { const n = new Date(); return { start: new Date(n.getFullYear(), n.getMonth() - 3, 1), end: new Date(n.getFullYear(), n.getMonth(), 0) } } },
    { label: 'Dernière année', id: 'last-year', getValue: () => { const n = new Date(); return { start: new Date(n.getFullYear() - 1, 0, 1), end: new Date(n.getFullYear() - 1, 11, 31) } } },
  ]}
]

const dpFormatTime = (date) => {
  const h = String(date.getHours()).padStart(2, '0')
  const m = String(date.getMinutes()).padStart(2, '0')
  return `${h}:${m}`
}

const dpApplyShortcut = (shortcut) => {
  const { start, end } = shortcut.getValue()
  dpTempStart.value = start
  dpTempEnd.value = end
  dpSelecting.value = false
  dpHoverDate.value = null
  dpActiveShortcut.value = shortcut.id
  if (shortcut.showTime) {
    dpShowTime.value = true
    dpTimeStart.value = dpFormatTime(start)
    dpTimeEnd.value = dpFormatTime(end)
  } else {
    dpShowTime.value = false
    dpTimeStart.value = '00:00'
    dpTimeEnd.value = '23:59'
  }
  dpLeftMonth.value = start.getMonth()
  dpLeftYear.value = start.getFullYear()
  if (dpMonthKey(end.getFullYear(), end.getMonth()) > dpMonthKey(start.getFullYear(), start.getMonth())) {
    dpRightMonth.value = end.getMonth()
    dpRightYear.value = end.getFullYear()
  } else {
    const r = dpStepMonth(dpLeftMonth.value, dpLeftYear.value, 1)
    dpRightMonth.value = r.month
    dpRightYear.value = r.year
  }
}

const dpApplySelection = () => {
  if (dpTempStart.value && !dpTempEnd.value) {
    dpTempEnd.value = dpTempStart.value
    dpSelecting.value = false
  }
  if (dpTempStart.value && dpTempEnd.value) {
    dpInternalStart.value = dpTempStart.value
    dpInternalEnd.value = dpTempEnd.value
    dpConfirmedShowTime.value = dpShowTime.value
    dpConfirmedTimeStart.value = dpTimeStart.value
    dpConfirmedTimeEnd.value = dpTimeEnd.value
    dateStart.value = dpFormatDate(dpTempStart.value)
    dateEnd.value = dpFormatDate(dpTempEnd.value)
    dpShowTimeStartDropdown.value = false
    dpShowTimeEndDropdown.value = false
    dpIsOpen.value = false
    dpSelecting.value = false
  }
}

const dpCancelSelection = () => {
  dpTempStart.value = dpInternalStart.value
  dpTempEnd.value = dpInternalEnd.value
  dpShowTime.value = dpConfirmedShowTime.value
  dpTimeStart.value = dpConfirmedTimeStart.value
  dpTimeEnd.value = dpConfirmedTimeEnd.value
  dpSelecting.value = false
  dpHoverDate.value = null
  dpActiveShortcut.value = null
  dpShowTimeStartDropdown.value = false
  dpShowTimeEndDropdown.value = false
  dpIsOpen.value = false
}

const dpResetSelection = () => {
  const today = new Date()
  dpTempStart.value = today
  dpTempEnd.value = today
  dpSelecting.value = false
  dpHoverDate.value = null
  dpActiveShortcut.value = 'today'
  dpLeftMonth.value = today.getMonth()
  dpLeftYear.value = today.getFullYear()
  const r = dpStepMonth(dpLeftMonth.value, dpLeftYear.value, 1)
  dpRightMonth.value = r.month
  dpRightYear.value = r.year
}

const dpToggleOpen = () => {
  if (dpIsOpen.value) {
    dpCancelSelection()
  } else {
    closeAllDropdowns('datepicker')
    dpTempStart.value = dpInternalStart.value
    dpTempEnd.value = dpInternalEnd.value
    dpShowTime.value = dpConfirmedShowTime.value
    dpTimeStart.value = dpConfirmedTimeStart.value
    dpTimeEnd.value = dpConfirmedTimeEnd.value
    dpSelecting.value = false
    dpActiveShortcut.value = null
    dpIsOpen.value = true
  }
}

const dpShowTime = ref(false)
const dpTimeStart = ref('00:00')
const dpTimeEnd = ref('23:59')
const dpConfirmedShowTime = ref(false)
const dpConfirmedTimeStart = ref('00:00')
const dpConfirmedTimeEnd = ref('23:59')
const dpShowTimeStartDropdown = ref(false)
const dpShowTimeEndDropdown = ref(false)

const dpIsSingleDay = computed(() => {
  if (!dpTempStart.value || !dpTempEnd.value) return !!dpTempStart.value && !dpTempEnd.value
  return dpDateToKey(dpTempStart.value) === dpDateToKey(dpTempEnd.value)
})

const dpTimeOptions = (() => {
  const opts = []
  for (let h = 0; h < 24; h++) {
    for (let m = 0; m < 60; m += 30) {
      opts.push(`${String(h).padStart(2, '0')}:${String(m).padStart(2, '0')}`)
    }
  }
  opts.push('23:59')
  return opts
})()

const dpToggleTimeStart = () => {
  dpShowTimeEndDropdown.value = false
  dpShowTimeStartDropdown.value = !dpShowTimeStartDropdown.value
}
const dpToggleTimeEnd = () => {
  dpShowTimeStartDropdown.value = false
  dpShowTimeEndDropdown.value = !dpShowTimeEndDropdown.value
}
const dpSelectTimeStart = (t) => {
  dpTimeStart.value = t
  dpShowTimeStartDropdown.value = false
  if (dpTimeEnd.value <= t) dpTimeEnd.value = '23:59'
}
const dpSelectTimeEnd = (t) => { if (t <= dpTimeStart.value) return; dpTimeEnd.value = t; dpShowTimeEndDropdown.value = false }

const dpToggleTime = () => {
  if (!dpIsSingleDay.value) return
  dpShowTime.value = !dpShowTime.value
  if (!dpShowTime.value) {
    dpTimeStart.value = '00:00'
    dpTimeEnd.value = '23:59'
  }
}

const datePickerRef = ref(null)

const handleClickOutside = (e) => {
  const topbar = document.querySelector('.topbar')
  if (topbar && !topbar.contains(e.target)) {
    closeAllDropdowns()
  }
  if (datePickerRef.value && !datePickerRef.value.contains(e.target) && dpIsOpen.value) {
    dpCancelSelection()
  }
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
})
</script>

<template>
  <div class="topbar">
    <!-- Corpus -->
    <div class="topbar-item topbar-dropdown" :class="{ active: showCorpusDropdown }">
      <span class="floating-label floating" :class="{ 'floating-active': showCorpusDropdown }">Corpus</span>
      <button class="topbar-dropdown-trigger" @click="toggleDropdown('corpus')">
        <span>{{ selectedCorpus }}</span>
        <svg class="icon-chevron" :class="{ rotated: showCorpusDropdown }" width="15" height="15" viewBox="0 0 25 25" fill="none"><path d="M13.3592 17.6611C12.8846 18.113 12.1173 18.113 11.6477 17.6611L3.5702 9.96875C3.09564 9.51683 3.09564 8.78606 3.5702 8.33894C4.04475 7.89183 4.81212 7.88702 5.28163 8.33894L12.5009 15.2139L19.7202 8.33894C20.1948 7.88702 20.9622 7.88702 21.4317 8.33894C21.9012 8.79087 21.9062 9.52164 21.4317 9.96875L13.3541 17.6611L13.3592 17.6611Z" fill="currentColor"/></svg>
      </button>
      <div v-if="showCorpusDropdown" class="menu">
        <div class="menu-search">
          <svg class="icon-search" width="12" height="12" viewBox="0 0 25 25" fill="none"><path d="M17.5296 10.3453C17.5296 8.43878 16.7725 6.6103 15.4247 5.26215C14.0769 3.914 12.2489 3.15662 10.3428 3.15662C8.43678 3.15662 6.60879 3.914 5.261 5.26215C3.91322 6.6103 3.15604 8.43878 3.15604 10.3453C3.15604 12.2519 3.91322 14.0804 5.261 15.4285C6.60879 16.7767 8.43678 17.5341 10.3428 17.5341C12.2489 17.5341 14.0769 16.7767 15.4247 15.4285C16.7725 14.0804 17.5296 12.2519 17.5296 10.3453ZM16.1417 17.6734C14.5516 18.9359 12.5348 19.6907 10.3428 19.6907C5.18182 19.6907 1 15.5078 1 10.3453C1 5.18294 5.18182 1 10.3428 1C15.5039 1 19.6857 5.18294 19.6857 10.3453C19.6857 12.5379 18.9311 14.5552 17.6689 16.1458L23.6833 22.1618C24.1056 22.5842 24.1056 23.2671 23.6833 23.6849C23.2611 24.1028 22.5784 24.1073 22.1606 23.6849L16.1417 17.6734Z" fill="currentColor"/></svg>
          <input type="text" v-model="corpusSearch" placeholder="" />
        </div>
        <div class="menu-options">
          <button
            v-for="option in filteredCorpusOptions()"
            :key="option"
            class="menu-option"
            :class="{ active: option === selectedCorpus }"
            @click="selectCorpus(option)"
            v-html="highlightMatch(option, corpusSearch)"
          />
        </div>
      </div>
    </div>

    <!-- Actors -->
    <div class="topbar-item topbar-dropdown" :class="{ active: showActorsDropdown }">
      <span
        v-if="showActorsDropdown || hasSelection()"
        class="floating-label floating"
        :class="{ 'floating-active': showActorsDropdown }"
      >Acteurs</span>
      <button class="topbar-dropdown-trigger" @click="toggleDropdown('actors')">
        <div v-if="hasSelection()" class="actors-tags">
          <span v-for="entity in checkedEntities" :key="entity" class="actors-tag--checkbox">
            {{ entity }}
            <svg class="icon-clear" @click.stop="removeEntity(entity)" width="10" height="10" viewBox="0 0 25 25" fill="none"><path d="M4.31219 5.81336C3.89594 5.39711 3.89594 4.72402 4.31219 4.31219C4.72844 3.90036 5.40154 3.89594 5.81336 4.31219L12.5 10.9988L19.1866 4.31219C19.6029 3.89594 20.276 3.89594 20.6878 4.31219C21.0996 4.72844 21.1041 5.40154 20.6878 5.81336L14.0012 12.5L20.6878 19.1866C21.1041 19.6029 21.1041 20.276 20.6878 20.6878C20.2716 21.0996 19.5985 21.1041 19.1866 20.6878L12.5 14.0012L5.81336 20.6878C5.39711 21.1041 4.72402 21.1041 4.31219 20.6878C3.90036 20.2716 3.89594 19.5985 4.31219 19.1866L10.9988 12.5L4.31219 5.81336Z" fill="currentColor"/></svg>
          </span>
          <span v-if="selectedActor" class="actors-tag">{{ selectedActor }}</span>
        </div>
        <span v-else-if="!showActorsDropdown">Acteurs</span>
        <span v-else></span>
        <div class="trigger-actions">
          <svg v-if="selectedActor" class="icon-clear" @click.stop="clearActor" width="13" height="13" viewBox="0 0 25 25" fill="none"><path d="M4.31219 5.81336C3.89594 5.39711 3.89594 4.72402 4.31219 4.31219C4.72844 3.90036 5.40154 3.89594 5.81336 4.31219L12.5 10.9988L19.1866 4.31219C19.6029 3.89594 20.276 3.89594 20.6878 4.31219C21.0996 4.72844 21.1041 5.40154 20.6878 5.81336L14.0012 12.5L20.6878 19.1866C21.1041 19.6029 21.1041 20.276 20.6878 20.6878C20.2716 21.0996 19.5985 21.1041 19.1866 20.6878L12.5 14.0012L5.81336 20.6878C5.39711 21.1041 4.72402 21.1041 4.31219 20.6878C3.90036 20.2716 3.89594 19.5985 4.31219 19.1866L10.9988 12.5L4.31219 5.81336Z" fill="currentColor"/></svg>
          <svg class="icon-chevron" :class="{ rotated: showActorsDropdown }" width="15" height="15" viewBox="0 0 25 25" fill="none"><path d="M13.3592 17.6611C12.8846 18.113 12.1173 18.113 11.6477 17.6611L3.5702 9.96875C3.09564 9.51683 3.09564 8.78606 3.5702 8.33894C4.04475 7.89183 4.81212 7.88702 5.28163 8.33894L12.5009 15.2139L19.7202 8.33894C20.1948 7.88702 20.9622 7.88702 21.4317 8.33894C21.9012 8.79087 21.9062 9.52164 21.4317 9.96875L13.3541 17.6611L13.3592 17.6611Z" fill="currentColor"/></svg>
        </div>
      </button>

      <div v-if="showActorsDropdown" class="menu" :class="{ 'menu--wide': actorsSearch || checkedEntities.length > 0 }">
        <div class="menu-search">
          <svg class="icon-search" width="12" height="12" viewBox="0 0 25 25" fill="none"><path d="M17.5296 10.3453C17.5296 8.43878 16.7725 6.6103 15.4247 5.26215C14.0769 3.914 12.2489 3.15662 10.3428 3.15662C8.43678 3.15662 6.60879 3.914 5.261 5.26215C3.91322 6.6103 3.15604 8.43878 3.15604 10.3453C3.15604 12.2519 3.91322 14.0804 5.261 15.4285C6.60879 16.7767 8.43678 17.5341 10.3428 17.5341C12.2489 17.5341 14.0769 16.7767 15.4247 15.4285C16.7725 14.0804 17.5296 12.2519 17.5296 10.3453ZM16.1417 17.6734C14.5516 18.9359 12.5348 19.6907 10.3428 19.6907C5.18182 19.6907 1 15.5078 1 10.3453C1 5.18294 5.18182 1 10.3428 1C15.5039 1 19.6857 5.18294 19.6857 10.3453C19.6857 12.5379 18.9311 14.5552 17.6689 16.1458L23.6833 22.1618C24.1056 22.5842 24.1056 23.2671 23.6833 23.6849C23.2611 24.1028 22.5784 24.1073 22.1606 23.6849L16.1417 17.6734Z" fill="currentColor"/></svg>
          <input type="text" v-model="actorsSearch" placeholder="" />
          <svg v-if="actorsSearch" class="icon-clear" @click="actorsSearch = ''" width="12" height="12" viewBox="0 0 25 25" fill="none"><path d="M4.31219 5.81336C3.89594 5.39711 3.89594 4.72402 4.31219 4.31219C4.72844 3.90036 5.40154 3.89594 5.81336 4.31219L12.5 10.9988L19.1866 4.31219C19.6029 3.89594 20.276 3.89594 20.6878 4.31219C21.0996 4.72844 21.1041 5.40154 20.6878 5.81336L14.0012 12.5L20.6878 19.1866C21.1041 19.6029 21.1041 20.276 20.6878 20.6878C20.2716 21.0996 19.5985 21.1041 19.1866 20.6878L12.5 14.0012L5.81336 20.6878C5.39711 21.1041 4.72402 21.1041 4.31219 20.6878C3.90036 20.2716 3.89594 19.5985 4.31219 19.1866L10.9988 12.5L4.31219 5.81336Z" fill="currentColor"/></svg>
        </div>

        <!-- Default view (no search, no selection) -->
        <div v-if="!actorsSearch && checkedEntities.length === 0" class="menu-options">
          <div v-for="category in actorsCategories" :key="category.id">
            <button class="menu-section-toggle" @click="toggleActorsCategory(category)">
              <span>{{ category.label }}</span>
              <svg class="icon-chevron icon-chevron--blue" :class="{ expanded: category.expanded }" width="15" height="15" viewBox="0 0 25 25" fill="none">
                <path d="M13.3592 17.6611C12.8846 18.113 12.1173 18.113 11.6477 17.6611L3.5702 9.96875C3.09564 9.51683 3.09564 8.78606 3.5702 8.33894C4.04475 7.89183 4.81212 7.88702 5.28163 8.33894L12.5009 15.2139L19.7202 8.33894C20.1948 7.88702 20.9622 7.88702 21.4317 8.33894C21.9012 8.79087 21.9062 9.52164 21.4317 9.96875L13.3541 17.6611L13.3592 17.6611Z" fill="currentColor"/>
              </svg>
            </button>
            <div v-if="isCategoryVisible(category)">
              <button
                v-for="item in filteredActorsItems(category.items)"
                :key="getItemName(item)"
                :class="['menu-option', { 'menu-option--flex': category.id === 'communities', active: getItemName(item) === selectedActor }]"
                @click="selectActor(item)"
              >
                <span v-if="category.id === 'communities'" class="actors-community-dot" :style="{ backgroundColor: item.color }"></span>
                <span v-html="highlightMatch(getItemName(item), actorsSearch)"></span>
              </button>
            </div>
          </div>
        </div>

        <!-- Search view (two columns) -->
        <div v-else class="actors-search-layout">
          <div class="actors-search-columns">
            <!-- Left: Entities -->
            <div class="actors-search-left">
              <div class="menu-section-header">
                <span class="menu-section">Entités</span>
                <svg v-if="checkedEntities.length > 0" class="icon-reset" @click="resetEntities" width="14" height="14" viewBox="0 0 25 25" fill="none">
                  <path d="M4.5 12.5C4.5 8.08 8.08 4.5 12.5 4.5C15.14 4.5 17.48 5.78 18.97 7.75L16.5 10.25H22.5V4.25L20.28 6.47C18.39 4.09 15.62 2.5 12.5 2.5C6.98 2.5 2.5 6.98 2.5 12.5C2.5 18.02 6.98 22.5 12.5 22.5C17.16 22.5 21.07 19.28 22.14 14.94H20.06C19.04 18.16 16.04 20.5 12.5 20.5C8.08 20.5 4.5 16.92 4.5 12.5Z" fill="currentColor"/>
                </svg>
              </div>
              <div class="menu-options">
                <button
                  v-for="entity in filteredEntities()"
                  :key="entity"
                  class="menu-option menu-option--flex"
                  :class="{ active: checkedEntities.includes(entity) }"
                  @click="toggleEntity(entity)"
                >
                  <svg v-if="!checkedEntities.includes(entity)" class="actors-checkbox" width="14" height="14" viewBox="0 0 12 12" fill="none">
                    <rect x="0.5" y="0.5" width="11" height="11" rx="1.5" stroke="#8D8D8D"/>
                  </svg>
                  <svg v-else class="actors-checkbox" width="14" height="14" viewBox="0 0 12 12" fill="none">
                    <rect width="12" height="12" rx="2" fill="#173EB7"/>
                    <path d="M9.2 3.6L5.8 8L3.8 6" stroke="white" stroke-width="0.72" stroke-linecap="round" stroke-linejoin="round"/>
                  </svg>
                  <span v-html="highlightMatch(entity, actorsSearch)"></span>
                </button>
              </div>
            </div>
            <!-- Right: Categories -->
            <div class="actors-search-right">
              <div class="menu-options">
                <div v-for="category in actorsCategories" :key="category.id">
                  <button class="menu-section-toggle" @click="toggleActorsCategory(category)">
                    <span>{{ category.label }}</span>
                    <svg class="icon-chevron icon-chevron--blue" :class="{ expanded: isCategoryVisible(category) }" width="15" height="15" viewBox="0 0 25 25" fill="none">
                      <path d="M13.3592 17.6611C12.8846 18.113 12.1173 18.113 11.6477 17.6611L3.5702 9.96875C3.09564 9.51683 3.09564 8.78606 3.5702 8.33894C4.04475 7.89183 4.81212 7.88702 5.28163 8.33894L12.5009 15.2139L19.7202 8.33894C20.1948 7.88702 20.9622 7.88702 21.4317 8.33894C21.9012 8.79087 21.9062 9.52164 21.4317 9.96875L13.3541 17.6611L13.3592 17.6611Z" fill="currentColor"/>
                    </svg>
                  </button>
                  <div v-if="isCategoryVisible(category)">
                    <button
                      v-for="item in filteredActorsItems(category.items)"
                      :key="getItemName(item)"
                      :class="['menu-option', { 'menu-option--flex': category.id === 'communities', active: getItemName(item) === selectedActor }]"
                      @click="selectActor(item)"
                    >
                      <span v-if="category.id === 'communities'" class="actors-community-dot" :style="{ backgroundColor: item.color }"></span>
                      <span v-html="highlightMatch(getItemName(item), actorsSearch)"></span>
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="actors-search-footer">
            <button v-if="checkedEntities.length >= 2" class="btn btn--outline btn--icon">
              <svg width="14" height="14" viewBox="0 0 25 25" fill="none"><path d="M5 4.25C4.5875 4.25 4.25 4.5875 4.25 5V20C4.25 20.4125 4.5875 20.75 5 20.75H20C20.4125 20.75 20.75 20.4125 20.75 20V8.62344C20.75 8.42656 20.6703 8.23438 20.5297 8.09375L17 4.55937V8.75C17 9.57969 16.3297 10.25 15.5 10.25H8C7.17031 10.25 6.5 9.57969 6.5 8.75V4.25H5ZM8.75 4.25V8H14.75V4.25H8.75ZM2 5C2 3.34531 3.34531 2 5 2H16.3766C17.1734 2 17.9375 2.31406 18.5 2.87656L22.1234 6.5C22.6859 7.0625 23 7.82656 23 8.62344V20C23 21.6547 21.6547 23 20 23H5C3.34531 23 2 21.6547 2 20V5ZM9.5 15.5C9.5 14.7044 9.81607 13.9413 10.3787 13.3787C10.9413 12.8161 11.7044 12.5 12.5 12.5C13.2956 12.5 14.0587 12.8161 14.6213 13.3787C15.1839 13.9413 15.5 14.7044 15.5 15.5C15.5 16.2956 15.1839 17.0587 14.6213 17.6213C14.0587 18.1839 13.2956 18.5 12.5 18.5C11.7044 18.5 10.9413 18.1839 10.3787 17.6213C9.81607 17.0587 9.5 16.2956 9.5 15.5Z" fill="currentColor"/></svg>
              Sauvegarder la liste
            </button>
            <button class="btn btn--primary" style="margin-left: auto" :class="{ 'btn--disabled': checkedEntities.length === 0 }" :disabled="checkedEntities.length === 0" @click="showActorsDropdown = false">Valider</button>
          </div>
        </div>
      </div>
    </div>

    <!-- Topic (Queries + Search + Language) -->
    <div class="topic-bar" :class="{ focused: topicSearchFocused }">
      <span v-if="topicSearchFocused || topicSearch" class="floating-label floating" :class="{ 'floating-active': topicSearchFocused }">Sujet</span>
      <!-- Query Dropdown -->
      <div class="topic-query" :class="{ active: showQueryDropdown }">
        <button class="topic-query-trigger" :class="{ open: showQueryDropdown, selected: !!selectedQuery }" @click="toggleQueryDropdown">
          <span v-if="!showQueryDropdown || selectedQuery" class="topic-query-text">{{ selectedQuery || 'Queries' }}</span>
          <div class="trigger-actions">
            <svg v-if="selectedQuery" class="icon-clear" @click.stop="clearQuery" width="13" height="13" viewBox="0 0 25 25" fill="none"><path d="M4.31219 5.81336C3.89594 5.39711 3.89594 4.72402 4.31219 4.31219C4.72844 3.90036 5.40154 3.89594 5.81336 4.31219L12.5 10.9988L19.1866 4.31219C19.6029 3.89594 20.276 3.89594 20.6878 4.31219C21.0996 4.72844 21.1041 5.40154 20.6878 5.81336L14.0012 12.5L20.6878 19.1866C21.1041 19.6029 21.1041 20.276 20.6878 20.6878C20.2716 21.0996 19.5985 21.1041 19.1866 20.6878L12.5 14.0012L5.81336 20.6878C5.39711 21.1041 4.72402 21.1041 4.31219 20.6878C3.90036 20.2716 3.89594 19.5985 4.31219 19.1866L10.9988 12.5L4.31219 5.81336Z" fill="currentColor"/></svg>
            <svg class="icon-chevron icon-chevron--blue" :class="{ rotated: showQueryDropdown }" width="15" height="15" viewBox="0 0 25 25" fill="none">
              <path d="M13.3592 17.6611C12.8846 18.113 12.1173 18.113 11.6477 17.6611L3.5702 9.96875C3.09564 9.51683 3.09564 8.78606 3.5702 8.33894C4.04475 7.89183 4.81212 7.88702 5.28163 8.33894L12.5009 15.2139L19.7202 8.33894C20.1948 7.88702 20.9622 7.88702 21.4317 8.33894C21.9012 8.79087 21.9062 9.52164 21.4317 9.96875L13.3541 17.6611L13.3592 17.6611Z" fill="currentColor"/>
            </svg>
          </div>
        </button>
        <div v-if="showQueryDropdown" class="menu topic-query-menu">
          <div class="menu-search">
            <svg class="icon-search" width="12" height="12" viewBox="0 0 25 25" fill="none"><path d="M17.5296 10.3453C17.5296 8.43878 16.7725 6.6103 15.4247 5.26215C14.0769 3.914 12.2489 3.15662 10.3428 3.15662C8.43678 3.15662 6.60879 3.914 5.261 5.26215C3.91322 6.6103 3.15604 8.43878 3.15604 10.3453C3.15604 12.2519 3.91322 14.0804 5.261 15.4285C6.60879 16.7767 8.43678 17.5341 10.3428 17.5341C12.2489 17.5341 14.0769 16.7767 15.4247 15.4285C16.7725 14.0804 17.5296 12.2519 17.5296 10.3453ZM16.1417 17.6734C14.5516 18.9359 12.5348 19.6907 10.3428 19.6907C5.18182 19.6907 1 15.5078 1 10.3453C1 5.18294 5.18182 1 10.3428 1C15.5039 1 19.6857 5.18294 19.6857 10.3453C19.6857 12.5379 18.9311 14.5552 17.6689 16.1458L23.6833 22.1618C24.1056 22.5842 24.1056 23.2671 23.6833 23.6849C23.2611 24.1028 22.5784 24.1073 22.1606 23.6849L16.1417 17.6734Z" fill="currentColor"/></svg>
            <input type="text" v-model="querySearch" placeholder="" />
            <svg v-if="querySearch" class="icon-clear" @click="querySearch = ''" width="12" height="12" viewBox="0 0 25 25" fill="none"><path d="M4.31219 5.81336C3.89594 5.39711 3.89594 4.72402 4.31219 4.31219C4.72844 3.90036 5.40154 3.89594 5.81336 4.31219L12.5 10.9988L19.1866 4.31219C19.6029 3.89594 20.276 3.89594 20.6878 4.31219C21.0996 4.72844 21.1041 5.40154 20.6878 5.81336L14.0012 12.5L20.6878 19.1866C21.1041 19.6029 21.1041 20.276 20.6878 20.6878C20.2716 21.0996 19.5985 21.1041 19.1866 20.6878L12.5 14.0012L5.81336 20.6878C5.39711 21.1041 4.72402 21.1041 4.31219 20.6878C3.90036 20.2716 3.89594 19.5985 4.31219 19.1866L10.9988 12.5L4.31219 5.81336Z" fill="currentColor"/></svg>
          </div>
          <span class="menu-section">Sujets</span>
          <div class="menu-options">
            <button
              v-for="option in filteredQueryOptions()"
              :key="option"
              class="menu-option"
              :class="{ active: option === selectedQuery }"
              @click="selectQuery(option)"
              v-html="highlightMatch(option, querySearch)"
            />
          </div>
        </div>
      </div>

      <!-- Search bar -->
      <svg class="topic-search-icon" width="15" height="15" viewBox="0 0 25 25" fill="none"><path d="M17.5296 10.3453C17.5296 8.43878 16.7725 6.6103 15.4247 5.26215C14.0769 3.914 12.2489 3.15662 10.3428 3.15662C8.43678 3.15662 6.60879 3.914 5.261 5.26215C3.91322 6.6103 3.15604 8.43878 3.15604 10.3453C3.15604 12.2519 3.91322 14.0804 5.261 15.4285C6.60879 16.7767 8.43678 17.5341 10.3428 17.5341C12.2489 17.5341 14.0769 16.7767 15.4247 15.4285C16.7725 14.0804 17.5296 12.2519 17.5296 10.3453ZM16.1417 17.6734C14.5516 18.9359 12.5348 19.6907 10.3428 19.6907C5.18182 19.6907 1 15.5078 1 10.3453C1 5.18294 5.18182 1 10.3428 1C15.5039 1 19.6857 5.18294 19.6857 10.3453C19.6857 12.5379 18.9311 14.5552 17.6689 16.1458L23.6833 22.1618C24.1056 22.5842 24.1056 23.2671 23.6833 23.6849C23.2611 24.1028 22.5784 24.1073 22.1606 23.6849L16.1417 17.6734Z" fill="currentColor"/></svg>
      <input type="text" v-model="topicSearch" class="topic-search-input" placeholder="Chercher un mot-clé, une thématique ..." @focus="topicSearchFocused = true; closeAllDropdowns()" @blur="topicSearchFocused = false" @keydown.enter="submitTopicSearch" />
      <svg v-if="topicSearch" class="icon-clear" @click="topicSearch = ''" width="13" height="13" viewBox="0 0 25 25" fill="none"><path d="M4.31219 5.81336C3.89594 5.39711 3.89594 4.72402 4.31219 4.31219C4.72844 3.90036 5.40154 3.89594 5.81336 4.31219L12.5 10.9988L19.1866 4.31219C19.6029 3.89594 20.276 3.89594 20.6878 4.31219C21.0996 4.72844 21.1041 5.40154 20.6878 5.81336L14.0012 12.5L20.6878 19.1866C21.1041 19.6029 21.1041 20.276 20.6878 20.6878C20.2716 21.0996 19.5985 21.1041 19.1866 20.6878L12.5 14.0012L5.81336 20.6878C5.39711 21.1041 4.72402 21.1041 4.31219 20.6878C3.90036 20.2716 3.89594 19.5985 4.31219 19.1866L10.9988 12.5L4.31219 5.81336Z" fill="currentColor"/></svg>

      <!-- Language Dropdown -->
      <div class="topic-lang" :class="{ active: showLanguageDropdown }">
        <button class="topic-lang-trigger" :class="{ open: showLanguageDropdown, selected: !!selectedLanguage }" @click="toggleLanguageDropdown">
          <img v-if="selectedLanguageOption" class="flag-circle flag-circle--sm" :src="selectedLanguageOption.flag" :alt="selectedLanguageOption.label" />
          <svg v-else width="14" height="13" viewBox="0 0 14 13" fill="none"><path d="M11.1538 5.2H9.84615L7 13H8.30769L9 11.0686H12L12.6923 13H14L11.1538 5.2ZM9.46154 9.88L10.5385 6.90857L11.6154 9.88H9.46154ZM5.76923 7.05714C7 5.72 8 4.38286 8.61538 2.82286H10V1.63429H5.53846V0H4.46154V1.63429H0V2.74857H7.38462C6.84615 3.93714 6 5.05143 5 6.16571C4.30769 5.34857 3.69231 4.45714 3.23077 3.71429H2C2.46154 4.75429 3.30769 5.86857 4.23077 6.98286L2.38462 8.76571C2.15385 9.06286 1.84615 9.36 1.53846 9.65714L2.30769 10.4L3.23077 9.50857C3.84615 8.91429 4.46154 8.39429 5 7.8C5.61538 8.46857 6.30769 9.06286 6.92308 9.65714L7.38462 8.54286C6.84615 8.09714 6.30769 7.57714 5.76923 7.05714Z" fill="currentColor"/></svg>
        </button>
        <div v-if="showLanguageDropdown" class="menu topic-lang-menu">
          <span class="menu-section">Langue des contenus</span>
          <div class="menu-options">
            <button
              v-for="lang in languageOptions"
              :key="lang.code"
              class="menu-option menu-option--flex"
              :class="{ active: lang.code === selectedLanguage }"
              @click="selectLanguage(lang)"
            >
              <img class="flag-circle" :src="lang.flag" :alt="lang.label" />
              <span class="topic-lang-label">{{ lang.label }}</span>
              <svg v-if="lang.code === selectedLanguage" class="icon-clear" @click.stop="clearLanguage" width="13" height="13" viewBox="0 0 25 25" fill="none"><path d="M4.31219 5.81336C3.89594 5.39711 3.89594 4.72402 4.31219 4.31219C4.72844 3.90036 5.40154 3.89594 5.81336 4.31219L12.5 10.9988L19.1866 4.31219C19.6029 3.89594 20.276 3.89594 20.6878 4.31219C21.0996 4.72844 21.1041 5.40154 20.6878 5.81336L14.0012 12.5L20.6878 19.1866C21.1041 19.6029 21.1041 20.276 20.6878 20.6878C20.2716 21.0996 19.5985 21.1041 19.1866 20.6878L12.5 14.0012L5.81336 20.6878C5.39711 21.1041 4.72402 21.1041 4.31219 20.6878C3.90036 20.2716 3.89594 19.5985 4.31219 19.1866L10.9988 12.5L4.31219 5.81336Z" fill="currentColor"/></svg>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Date Picker -->
    <div ref="datePickerRef" class="datepicker" :class="{ compact: sidebarExpanded }">
      <div class="datepicker-trigger" :class="{ active: dpIsOpen }" @click="dpToggleOpen">
        <span v-if="!sidebarExpanded" class="floating-label floating" :class="{ 'floating-active': dpIsOpen }">Période</span>
        <span v-if="!sidebarExpanded && dpDisplayValue" class="datepicker-value" :class="{ pending: dpHasChanged }">{{ dpDisplayValue }}</span>
        <span v-else-if="!sidebarExpanded" class="datepicker-placeholder">Sélectionner une période</span>
        <div class="trigger-actions">
          <svg class="datepicker-icon" width="15" height="15" viewBox="0 0 25 25" fill="none">
            <path d="M8.5 2V5.5M16.5 2V5.5M3.5 9.5H21.5M5.5 4H19.5C20.6046 4 21.5 4.89543 21.5 6V20C21.5 21.1046 20.6046 22 19.5 22H5.5C4.39543 22 3.5 21.1046 3.5 20V6C3.5 4.89543 4.39543 4 5.5 4Z" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
      </div>

      <Transition name="dropdown">
        <div v-if="dpIsOpen" class="datepicker-popup">
          <!-- Presets -->
          <div class="datepicker-presets">
            <template v-for="(group, gi) in dpShortcutGroups" :key="group.id">
              <div :class="group.id">
                <button
                  v-for="shortcut in group.items"
                  :key="shortcut.id"
                  class="menu-option"
                  :class="{ active: dpActiveShortcut === shortcut.id }"
                  @click="dpApplyShortcut(shortcut)"
                >
                  {{ shortcut.label }}
                </button>
              </div>
              <div v-if="gi < dpShortcutGroups.length - 1" class="datepicker-presets-divider"></div>
            </template>
          </div>

          <!-- Calendars -->
          <div class="datepicker-main">
            <div class="datepicker-calendars">
              <div class="datepicker-calendar">
                <div class="datepicker-calendar-nav">
                  <button class="datepicker-nav-arrow" @click="dpLeftPrev">
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none"><path d="M14 7L9 12L14 17" stroke="#595959" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
                  </button>
                  <span class="datepicker-month-title">{{ dpMonthNames[dpLeftMonth] }} {{ dpLeftYear }}</span>
                  <button class="datepicker-nav-arrow" @click="dpLeftNext">
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none"><path d="M10 7L15 12L10 17" stroke="#595959" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
                  </button>
                </div>
                <div class="datepicker-weekdays">
                  <span v-for="name in dpDayNames" :key="name" class="datepicker-weekday">{{ name }}</span>
                </div>
                <div class="datepicker-days">
                  <button
                    v-for="(dayObj, i) in dpLeftCalendarDays"
                    :key="'l' + i"
                    :class="dpDayClasses(dayObj)"
                    @click="dpSelectDay(dayObj)"
                    @mouseenter="dpOnDayHover(dayObj)"
                  >
                    {{ dayObj.day }}
                  </button>
                </div>
              </div>

              <div class="datepicker-calendar">
                <div class="datepicker-calendar-nav">
                  <button class="datepicker-nav-arrow" @click="dpRightPrev">
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none"><path d="M14 7L9 12L14 17" stroke="#595959" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
                  </button>
                  <span class="datepicker-month-title">{{ dpMonthNames[dpRightMonth] }} {{ dpRightYear }}</span>
                  <button class="datepicker-nav-arrow" @click="dpRightNext">
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none"><path d="M10 7L15 12L10 17" stroke="#595959" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
                  </button>
                </div>
                <div class="datepicker-weekdays">
                  <span v-for="name in dpDayNames" :key="name" class="datepicker-weekday">{{ name }}</span>
                </div>
                <div class="datepicker-days">
                  <button
                    v-for="(dayObj, i) in dpRightCalendarDays"
                    :key="'r' + i"
                    :class="dpDayClasses(dayObj)"
                    @click="dpSelectDay(dayObj)"
                    @mouseenter="dpOnDayHover(dayObj)"
                  >
                    {{ dayObj.day }}
                  </button>
                </div>
              </div>
            </div>

            <!-- Toggle row -->
            <div class="datepicker-toggle-row">
              <div class="datepicker-toggle-left">
                <span class="datepicker-toggle-label" :class="{ disabled: !dpIsSingleDay }">Sélectionner un horaire</span>
                <button class="datepicker-switch" :class="{ on: dpShowTime, disabled: !dpIsSingleDay }" @click="dpToggleTime">
                  <span class="datepicker-switch-knob"></span>
                </button>
              </div>
              <div v-if="dpShowTime && dpIsSingleDay" class="datepicker-time-range">
                <div class="datepicker-time-dropdown">
                  <button class="datepicker-time-pill" @click="dpToggleTimeStart">
                    <span>{{ dpTimeStart }}</span>
                    <svg class="icon-chevron" width="15" height="15" viewBox="0 0 25 25" fill="none"><path d="M13.3592 17.6611C12.8846 18.113 12.1173 18.113 11.6477 17.6611L3.5702 9.96875C3.09564 9.51683 3.09564 8.78606 3.5702 8.33894C4.04475 7.89183 4.81212 7.88702 5.28163 8.33894L12.5009 15.2139L19.7202 8.33894C20.1948 7.88702 20.9622 7.88702 21.4317 8.33894C21.9012 8.79087 21.9062 9.52164 21.4317 9.96875L13.3541 17.6611L13.3592 17.6611Z" fill="currentColor"/></svg>
                  </button>
                  <div v-if="dpShowTimeStartDropdown" class="menu datepicker-time-menu" @click.stop>
                    <button v-for="t in dpTimeOptions" :key="'ts'+t" class="menu-option menu-option--small" :class="{ active: t === dpTimeStart }" @click="dpSelectTimeStart(t)">{{ t }}</button>
                  </div>
                </div>
                <span class="datepicker-time-separator">-</span>
                <div class="datepicker-time-dropdown">
                  <button class="datepicker-time-pill" @click="dpToggleTimeEnd">
                    <span>{{ dpTimeEnd }}</span>
                    <svg class="icon-chevron" width="15" height="15" viewBox="0 0 25 25" fill="none"><path d="M13.3592 17.6611C12.8846 18.113 12.1173 18.113 11.6477 17.6611L3.5702 9.96875C3.09564 9.51683 3.09564 8.78606 3.5702 8.33894C4.04475 7.89183 4.81212 7.88702 5.28163 8.33894L12.5009 15.2139L19.7202 8.33894C20.1948 7.88702 20.9622 7.88702 21.4317 8.33894C21.9012 8.79087 21.9062 9.52164 21.4317 9.96875L13.3541 17.6611L13.3592 17.6611Z" fill="currentColor"/></svg>
                  </button>
                  <div v-if="dpShowTimeEndDropdown" class="menu datepicker-time-menu" @click.stop>
                    <button v-for="t in dpTimeOptions" :key="'te'+t" class="menu-option menu-option--small" :class="{ active: t === dpTimeEnd, disabled: t <= dpTimeStart }" :disabled="t <= dpTimeStart" @click="dpSelectTimeEnd(t)">{{ t }}</button>
                  </div>
                </div>
              </div>
            </div>

            <!-- Footer -->
            <div class="datepicker-footer">
              <button class="btn btn--outline" @click="dpResetSelection">Effacer</button>
              <div class="datepicker-footer-right">
                <button class="btn btn--outline" @click="dpCancelSelection">Annuler</button>
                <button class="btn btn--primary" :class="{ 'btn--disabled': !dpTempStart }" :disabled="!dpTempStart" @click="dpApplySelection">Valider</button>
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
/* ── Shared: Menu ── */
.menu {
  position: absolute;
  top: calc(100% + 5px);
  left: 0;
  right: 0;
  background: var(--color-white);
  border-radius: 8px;
  border: 1px solid var(--color-light-grey);
  z-index: 100;
  padding: 5px;
  display: flex;
  flex-direction: column;
  gap: 5px;
  max-height: 300px;
}
.menu--wide { min-width: 430px; max-height: 350px; }
.menu-search {
  display: flex;
  align-items: center;
  gap: 5px;
  height: 32px;
  padding: 8px;
  border: 1px solid var(--color-light-grey);
  border-radius: 6px;
}
.menu-search input {
  width: 100%;
  min-width: 0;
  border: none;
  outline: none;
  font-size: 14px;
  color: var(--color-black);
  background: transparent;
  font-family: 'Roboto', sans-serif;
}
.menu-options {
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 2px;
}
.menu-option {
  display: block;
  width: 100%;
  padding: 8px;
  border: none;
  background: var(--color-white);
  text-align: left;
  cursor: pointer;
  font-size: 16px;
  color: var(--color-black);
  font-family: 'Roboto', sans-serif;
  border-radius: 4px;
  transition: background-color 0.15s ease;
}
.menu-option:hover { background-color: var(--color-background); }
.menu-option.active {
  background-color: var(--color-active);
  color: var(--color-blue);
  font-weight: 500;
}
.menu-option.disabled { cursor: default; pointer-events: none; opacity: 0.5; }
.menu-option--flex { display: flex; align-items: center; gap: 8px; }
.menu-option--small { padding: 6px 10px; font-size: 14px; background: transparent; }
.menu-section {
  font-size: 14px;
  font-weight: 500;
  color: var(--color-blue);
  font-family: 'Roboto', sans-serif;
  padding: 8px;
}
.menu-section-toggle {
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
  border-radius: 4px;
  transition: background-color 0.15s ease;
}
.menu-section-toggle:hover { background-color: var(--color-background); }
.menu-section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
:deep(.highlight-match) { font-weight: 600; }

/* ── Shared: Icons ── */
.icon-chevron {
  width: 15px;
  height: 15px;
  transition: transform 0.2s ease;
  color: var(--color-black);
  flex-shrink: 0;
}
.icon-chevron.rotated,
.icon-chevron.expanded { transform: rotate(180deg); }
.icon-chevron--blue { color: var(--color-blue); }
.icon-clear { color: var(--color-black); cursor: pointer; flex-shrink: 0; }
.icon-search { flex-shrink: 0; color: var(--color-medium-grey); }
.icon-reset { color: var(--color-blue); cursor: pointer; }

/* ── Shared: Buttons ── */
.btn {
  padding: 10px 15px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  font-family: 'Roboto', sans-serif;
  cursor: pointer;
}
.btn--primary {
  background: var(--color-blue);
  color: var(--color-white);
  border: none;
  transition: opacity 0.15s ease;
}
.btn--primary:hover:not(.btn--disabled) { opacity: 0.9; }
.btn--outline {
  background: transparent;
  color: var(--color-blue);
  border: 1px solid var(--color-blue);
  transition: background-color 0.15s ease;
}
.btn--outline:hover:not(.btn--disabled) { background-color: #EFF6FF; }
.btn--icon { display: flex; align-items: center; gap: 5px; }
.btn--disabled { opacity: 0.4; cursor: not-allowed; }

/* ── Shared: Floating Label ── */
.floating-label {
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
  border-left: 1px solid var(--color-medium-grey);
  background: transparent;
  transition: all 0.2s ease;
  z-index: 102;
}
.floating-label.floating {
  right: auto;
  left: 10px;
  top: 0;
  padding: 0 1px;
  height: auto;
  font-size: 12px;
  font-weight: 500;
  color: var(--color-medium-grey);
  background: var(--color-white);
  border-left: none;
}
.floating-label.floating-active { color: var(--color-blue); }

/* ── Shared: Flag ── */
.flag-circle {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  flex-shrink: 0;
  object-fit: cover;
  overflow: hidden;
}
.flag-circle--sm { width: 15px; height: 15px; }

/* ── Layout ── */
.topbar {
  display: flex;
  align-items: center;
  padding: 20px 30px;
  gap: 10px;
}
.topbar-item {
  background: var(--color-white);
  border-radius: 8px;
  height: 40px;
  display: flex;
  align-items: center;
}
.topbar-dropdown {
  position: relative;
  width: 240px;
  border: 1px solid var(--color-light-grey);
  transition: border-color 0.2s ease;
}
.topbar-dropdown.active { border-color: var(--color-blue); }
.topbar-dropdown-trigger {
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
  gap: 5px;
}
.trigger-actions { display: flex; align-items: center; gap: 6px; }

/* ── Actors ── */
.actors-tags {
  display: flex;
  align-items: center;
  gap: 2px;
  overflow-x: auto;
  overflow-y: hidden;
  flex: 1;
  min-width: 0;
  scrollbar-width: none;
}
.actors-tags::-webkit-scrollbar { display: none; }
.actors-tag--checkbox {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 14px;
  white-space: nowrap;
  background: var(--color-active);
  padding: 0 5px;
  border-radius: 4px;
  height: 26px;
  flex-shrink: 0;
}
.actors-tag {
  display: flex;
  align-items: center;
  gap: 4px;
  font-weight: 500;
  white-space: nowrap;
  flex-shrink: 0;
}
.actors-community-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  flex-shrink: 0;
}
.actors-checkbox { flex-shrink: 0; }
.actors-search-layout {
  display: flex;
  flex-direction: column;
  gap: 10px;
  overflow: hidden;
  flex: 1;
}
.actors-search-columns {
  display: flex;
  gap: 10px;
  overflow: hidden;
  flex: 1;
}
.actors-search-left {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 5px;
  overflow-y: auto;
  border-right: 1px solid var(--color-light-grey);
  padding-right: 10px;
}
.actors-search-right {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}
.actors-search-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 5px;
}

/* ── Topic ── */
.topic-bar {
  position: relative;
  flex: 1;
  min-width: 0;
  height: 40px;
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 5px;
  background: var(--color-white);
  border: 1px solid var(--color-light-grey);
  border-radius: 8px;
}
.topic-bar.focused { border-color: var(--color-blue); }
.topic-query { position: relative; flex-shrink: 0; }
.topic-query-trigger {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 200px;
  height: 29px;
  padding: 5px 10px;
  background: var(--color-background);
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
  font-family: 'Roboto', sans-serif;
  color: var(--color-blue);
}
.topic-query-trigger.selected { color: var(--color-black); }
.topic-query-trigger.selected .icon-chevron { color: var(--color-black); }
.topic-query-trigger.selected .topic-query-text {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  min-width: 0;
  flex: 1;
  text-align: left;
}
.topic-query-trigger.open { border: 1px solid var(--color-blue); justify-content: flex-end; }
.topic-query-trigger.open.selected { justify-content: space-between; }
.topic-query-menu { left: 0; right: auto; width: 200px; }
.topic-search-icon { flex-shrink: 0; color: var(--color-dark-grey); }
.topic-search-input {
  flex: 1;
  min-width: 0;
  border: none;
  outline: none;
  font-size: 16px;
  font-weight: 400;
  color: var(--color-black);
  background: transparent;
  font-family: 'Roboto', sans-serif;
}
.topic-search-input::placeholder { color: var(--color-medium-grey); font-style: italic; }
.topic-lang { position: relative; flex-shrink: 0; }
.topic-lang-trigger {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 29px;
  height: 29px;
  background: var(--color-background);
  border: 1px solid transparent;
  border-radius: 8px;
  cursor: pointer;
  color: var(--color-blue);
}
.topic-lang-trigger.open { border-color: var(--color-blue); }
.topic-lang-menu { left: auto; right: 0; width: 176px; max-height: 200px; gap: 2px; }
.topic-lang-menu .menu-option.active .icon-clear { color: var(--color-blue); }
.topic-lang-label { flex: 1; }

/* ── DatePicker ── */
.datepicker { position: relative; flex-shrink: 0; }
.datepicker-trigger {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  height: 40px;
  padding: 0 12px;
  background: var(--color-white);
  border: 1px solid var(--color-light-grey);
  border-radius: 8px;
  cursor: pointer;
  transition: border-color 0.2s ease;
}
.datepicker-trigger.active { border-color: var(--color-blue); }
.datepicker-value {
  font-size: 16px;
  font-weight: 500;
  color: var(--color-black);
  font-family: 'Roboto', sans-serif;
  white-space: nowrap;
}
.datepicker-value.pending { color: var(--color-dark-grey); }
.datepicker-placeholder {
  font-size: 16px;
  color: var(--color-medium-grey);
  font-family: 'Roboto', sans-serif;
  white-space: nowrap;
}
.datepicker-icon { color: var(--color-black); flex-shrink: 0; }
.datepicker.compact .datepicker-trigger { width: auto; padding: 0 12px; gap: 0; }
.datepicker-popup {
  position: absolute;
  top: calc(100% + 5px);
  right: 0;
  display: flex;
  height: 472px;
  background: var(--color-white);
  border-radius: 16px;
  box-shadow: 0 3px 5px rgba(0, 0, 0, 0.08);
  z-index: 200;
}
.datepicker-presets {
  display: flex;
  flex-direction: column;
  width: 170px;
  padding: 12px 10px;
  gap: 10px;
  border-right: 1px solid var(--color-light-grey);
  flex-shrink: 0;
  overflow-x: scroll;
}
.datepicker-presets-divider {
  width: 100%;
  height: 1px;
  background: var(--color-light-grey);
  flex-shrink: 0;
}
.datepicker-main {
  display: flex;
  flex-direction: column;
  padding: 20px 30px;
  gap: 20px;
}
.datepicker-calendars { display: flex; gap: 30px; }
.datepicker-calendar {
  display: flex;
  flex-direction: column;
  width: 281px;
  gap: 20px;
}
.datepicker-calendar-nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 40px;
}
.datepicker-nav-arrow {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border: none;
  background: transparent;
  border-radius: 10px;
  cursor: pointer;
  padding: 0 8px;
  transition: background-color 0.15s ease;
}
.datepicker-nav-arrow:hover { background-color: var(--color-background); }
.datepicker-month-title {
  font-size: 16px;
  font-weight: 500;
  line-height: 18.75px;
  color: var(--color-dark-grey);
  font-family: 'Roboto', sans-serif;
}
.datepicker-weekdays { display: flex; gap: 6px; }
.datepicker-weekday {
  width: 35px;
  text-align: center;
  font-size: 16px;
  font-weight: 500;
  line-height: 18.75px;
  color: var(--color-dark-grey);
  font-family: 'Roboto', sans-serif;
}
.datepicker-days {
  display: grid;
  grid-template-columns: repeat(7, 35px);
  gap: 6px;
  row-gap: 2px;
}
.datepicker-day {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 35px;
  height: 35px;
  border: none;
  background: transparent;
  cursor: pointer;
  font-size: 16px;
  font-weight: 400;
  color: var(--color-dark-grey);
  font-family: 'Roboto', sans-serif;
  border-radius: 99px;
  transition: background-color 0.15s ease;
}
.datepicker-day:hover:not(.other-month):not(.range-start):not(.range-end):not(.range-single):not(.today) {
  background-color: var(--color-active);
}
.datepicker-day.other-month { opacity: 0.5; cursor: default; }
.datepicker-day.range-start,
.datepicker-day.range-end,
.datepicker-day.range-single {
  background-color: var(--color-blue);
  color: var(--color-white);
  font-weight: 600;
}
.datepicker-day.in-range {
  background-color: var(--color-active);
  color: var(--color-blue);
}
.datepicker-toggle-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 35px;
}
.datepicker-toggle-left { display: flex; align-items: center; gap: 10px; }
.datepicker-toggle-label {
  font-size: 14px;
  font-weight: 500;
  line-height: 16.41px;
  color: var(--color-black);
  font-family: 'Roboto', sans-serif;
}
.datepicker-toggle-label.disabled { color: var(--color-medium-grey); }
.datepicker-switch {
  position: relative;
  width: 30px;
  height: 16px;
  border-radius: 10px;
  border: none;
  background: rgba(23, 62, 183, 0.5);
  cursor: pointer;
  padding: 2px;
  transition: background-color 0.2s ease;
}
.datepicker-switch.on { background: var(--color-blue); }
.datepicker-switch.disabled { background: var(--color-light-grey); cursor: not-allowed; }
.datepicker-switch-knob {
  display: block;
  width: 12px;
  height: 12px;
  border-radius: 999px;
  background: var(--color-white);
  transition: transform 0.2s ease;
}
.datepicker-switch.on .datepicker-switch-knob { transform: translateX(14px); }
.datepicker-time-range { display: flex; align-items: center; gap: 10px; }
.datepicker-time-separator {
  font-size: 16px;
  color: var(--color-black);
  font-family: 'Roboto', sans-serif;
}
.datepicker-time-dropdown { position: relative; }
.datepicker-time-pill {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 95px;
  height: 35px;
  padding: 8px 15px;
  border-radius: 99px;
  background: #EFF6FF;
  border: 1px solid #EFF6FF;
  cursor: pointer;
  font-size: 16px;
  color: var(--color-black);
  font-family: 'Roboto', sans-serif;
}
.datepicker-time-menu {
  position: absolute;
  bottom: calc(100% + 5px);
  left: 0;
  right: auto;
  top: auto;
  width: 95px;
  max-height: 200px;
  overflow-y: auto;
  z-index: 300;
}
.datepicker-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.datepicker-footer-right { display: flex; align-items: center; gap: 10px; }

/* ── Transition ── */
.dropdown-enter-active { transition: all 0.2s ease-out; }
.dropdown-leave-active { transition: all 0.15s ease-in; }
.dropdown-enter-from,
.dropdown-leave-to { opacity: 0; transform: translateY(-4px); }
</style>
