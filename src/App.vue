<script setup>
import { computed, nextTick, onMounted, onUnmounted, ref, watch } from 'vue'
import source from './whitepaper.ko.json'
import { chapterMeta, groups } from './whitepaperMeta'

const asset = (name) => import.meta.env.BASE_URL + name
const lang = ref(localStorage.getItem('surfx-language') === 'ko' ? 'ko' : 'en')
const pageId = ref(readPageId())
const menuOpen = ref(false)
const sidebar = ref(null)
const searchOpen = ref(false)
const searchQuery = ref('')
const searchInput = ref(null)
const activeAnchor = ref('')

const englishIntro = [
  'SurfX proposes to connect mobile games, utility apps, and tools for user-directed stock and crypto system trading through one account and a shared token. The whitepaper sets out common payment and settlement rules for users, developers, and creators.',
  'SURFX is proposed as a BEP-20 utility token on BNB Smart Chain. App-specific scores and points remain tied to their services; only approved uses and conversions would follow published terms.',
  'This is a planning document, version 0.2. Service launches, token issuance, contract deployment, funding, audits, and Telegram or WhatsApp channels require separate confirmation and disclosure.',
]

function readPageId() {
  const id = window.location.hash.replace(/^#\/?/, '')
  return source.chapters.some((chapter) => chapter.id === id) ? id : 'intro'
}
function cleanTitle(chapter) { return chapter.title.replace(/^\d+\s*/, '') }
const currentIndex = computed(() => source.chapters.findIndex((chapter) => chapter.id === pageId.value))
const currentChapter = computed(() => source.chapters[currentIndex.value] || null)
const currentMeta = computed(() => currentIndex.value < 0 ? null : chapterMeta[currentIndex.value])
const currentGroup = computed(() => groups.find((group) => group.chapters.includes(currentIndex.value + 1)) || null)
const pageTitle = computed(() => currentChapter.value ? (lang.value === 'ko' ? cleanTitle(currentChapter.value) : currentMeta.value.en) : (lang.value === 'ko' ? '소개' : 'Introduction'))
const toc = computed(() => currentChapter.value ? currentChapter.value.blocks.map((block, index) => ({ ...block, index })).filter((block) => block.type === 'heading') : [])
const previousPage = computed(() => currentIndex.value > 0 ? source.chapters[currentIndex.value - 1] : null)
const nextPage = computed(() => currentIndex.value < source.chapters.length - 1 ? source.chapters[currentIndex.value + 1] : null)
const searchResults = computed(() => {
  const query = searchQuery.value.trim().toLocaleLowerCase()
  if (!query) return []
  return source.chapters.map((chapter, index) => ({
    ...chapter,
    englishTitle: chapterMeta[index].en,
    summary: chapterMeta[index].summary,
    searchable: [chapter.title, chapterMeta[index].en, chapterMeta[index].summary, ...chapter.blocks.flatMap((block) => block.type === 'table' ? block.rows.flat() : [block.text])].join(' ').toLocaleLowerCase(),
  })).filter((chapter) => chapter.searchable.includes(query)).slice(0, 10)
})

function navigate(id) {
  window.location.hash = '#/' + id
  pageId.value = id
  menuOpen.value = false
  searchOpen.value = false
  searchQuery.value = ''
  activeAnchor.value = ''
  window.scrollTo({ top: 0, behavior: 'instant' })
  revealSelected()
}
function syncRoute() {
  pageId.value = readPageId()
  menuOpen.value = false
  activeAnchor.value = ''
  window.scrollTo({ top: 0, behavior: 'instant' })
  revealSelected()
}
function revealSelected() {
  nextTick(() => {
    const selected = sidebar.value?.querySelector('button.selected')
    if (selected && sidebar.value) sidebar.value.scrollTop = Math.max(0, selected.offsetTop - sidebar.value.clientHeight * 0.42)
  })
}
function changeLanguage(value) {
  lang.value = value
  localStorage.setItem('surfx-language', value)
}
function openSearch() {
  searchOpen.value = true
  nextTick(() => searchInput.value?.focus())
}
function jumpTo(index) {
  activeAnchor.value = 'section-' + index
  window.document.getElementById(activeAnchor.value)?.scrollIntoView({ behavior: 'smooth', block: 'start' })
}
function jumpToSelector(selector) { window.document.querySelector(selector)?.scrollIntoView({ behavior: 'smooth' }) }
function onKeydown(event) {
  if ((event.ctrlKey || event.metaKey) && event.key.toLowerCase() === 'k') {
    event.preventDefault()
    openSearch()
  }
  if (event.key === 'Escape') searchOpen.value = false
}
function isExternalLink(value) { return /^https:\/\/\S+$/.test(value) }

watch([lang, pageTitle], () => {
  window.document.documentElement.lang = lang.value
  window.document.title = pageTitle.value + ' | SurfX Whitepaper'
}, { immediate: true })
onMounted(() => {
  window.addEventListener('hashchange', syncRoute)
  window.addEventListener('keydown', onKeydown)
  revealSelected()
})
onUnmounted(() => {
  window.removeEventListener('hashchange', syncRoute)
  window.removeEventListener('keydown', onKeydown)
})
</script>

<template>
  <div class="docs-shell">
    <header class="docs-header">
      <button class="menu-button" type="button" :aria-label="lang === 'ko' ? '목차 열기' : 'Open navigation'" @click="menuOpen = !menuOpen">☰</button>
      <a class="docs-brand" href="#/intro" @click.prevent="navigate('intro')"><span class="brand-wave">∿</span><span>Surf<span class="brand-accent">X</span></span><span class="brand-divider"></span><span class="brand-document">Whitepaper</span></a>
      <div class="header-tools">
        <button class="search-trigger" type="button" @click="openSearch"><span class="search-icon">⌕</span><span>{{ lang === 'ko' ? '백서 검색' : 'Search documentation' }}</span><kbd>Ctrl K</kbd></button>
        <div class="language-switch" :aria-label="lang === 'ko' ? '언어 선택' : 'Choose language'"><button type="button" :class="{ active: lang === 'ko' }" :aria-pressed="lang === 'ko'" @click="changeLanguage('ko')">KO</button><span>/</span><button type="button" :class="{ active: lang === 'en' }" :aria-pressed="lang === 'en'" @click="changeLanguage('en')">EN</button></div>
        <a class="main-site-link" href="https://www.surfx-protocol.com/">{{ lang === 'ko' ? '메인 사이트' : 'Main site' }} ↗</a>
      </div>
    </header>

    <div v-if="menuOpen" class="mobile-shade" @click="menuOpen = false"></div>
    <aside ref="sidebar" class="docs-sidebar" :class="{ open: menuOpen }" :aria-label="lang === 'ko' ? '백서 목차' : 'Whitepaper navigation'">
      <div class="sidebar-inner">
        <p class="sidebar-caption">SURFX DOCUMENTATION</p>
        <button class="nav-intro" type="button" :class="{ selected: pageId === 'intro' }" @click="navigate('intro')"><span class="nav-icon">⌂</span>{{ lang === 'ko' ? '소개' : 'Introduction' }}</button>
        <section v-for="group in groups" :key="group.id" class="nav-group">
          <h2><span class="nav-icon">{{ group.icon }}</span>{{ lang === 'ko' ? group.ko : group.en }}</h2>
          <button v-for="number in group.chapters" :key="number" type="button" :class="{ selected: pageId === source.chapters[number - 1].id }" @click="navigate(source.chapters[number - 1].id)"><span class="chapter-number">{{ String(number).padStart(2, '0') }}</span><span>{{ lang === 'ko' ? cleanTitle(source.chapters[number - 1]) : chapterMeta[number - 1].en }}</span></button>
        </section>
      </div>
    </aside>

    <div class="docs-content-grid">
      <main class="docs-main" id="main-content">
        <div class="breadcrumb"><span>{{ currentGroup ? (lang === 'ko' ? currentGroup.ko : currentGroup.en) : (lang === 'ko' ? '개요' : 'OVERVIEW') }}</span><span class="breadcrumb-slash">/</span><span>{{ pageTitle }}</span></div>

        <article v-if="!currentChapter" class="article">
          <div class="article-heading"><p class="article-kicker">SURFX WHITEPAPER · V0.2</p><h1>{{ pageTitle }}</h1><p class="article-lede">{{ lang === 'ko' ? 'SurfX 생태계와 공용 토큰의 설계, 운영 원칙과 단계별 계획' : 'The ecosystem, shared token design, operating principles, and phased plan for SurfX.' }}</p></div>
          <div class="cover-art"><div><span>THE NEXT WAVE OF CONNECTED EXPERIENCES</span><strong>SurfX<br>Whitepaper</strong><small>October 2026 · Version 0.2</small></div><img :src="asset('wave.webp')" alt="" /></div>
          <div class="document-notice"><strong>{{ lang === 'ko' ? '발행 준비 기준안' : 'Planning baseline' }}</strong><p>{{ lang === 'ko' ? '이 문서는 토큰 발행이나 서비스 출시가 완료되었다는 뜻이 아닙니다. 실제 배포와 운영 정보는 별도로 확인해야 합니다.' : 'This document does not state that token issuance or service launches are complete. Deployment and operating details require separate confirmation.' }}</p></div>
          <section class="intro-copy"><h2>{{ lang === 'ko' ? '프로젝트 개요' : 'Project overview' }}</h2><p v-for="paragraph in (lang === 'ko' ? source.introduction : englishIntro)" :key="paragraph">{{ paragraph }}</p></section>
          <div class="intro-highlights"><div><span>25</span><small>{{ lang === 'ko' ? '개 장' : 'Chapters' }}</small></div><div><span>14</span><small>{{ lang === 'ko' ? '개 표' : 'Source tables' }}</small></div><div><span>v0.2</span><small>{{ lang === 'ko' ? '운영 정책 반영본' : 'Planning draft' }}</small></div></div>
          <button class="next-card" type="button" @click="navigate('chapter-01')"><small>{{ lang === 'ko' ? '다음 장' : 'NEXT' }}</small><strong>{{ lang === 'ko' ? '프로젝트 개요' : 'Project Overview' }} →</strong></button>
        </article>

        <article v-else class="article chapter-article">
          <div class="article-heading"><p class="article-kicker">CHAPTER {{ String(currentIndex + 1).padStart(2, '0') }} / 25</p><h1>{{ pageTitle }}</h1><p v-if="lang === 'en'" class="original-title">{{ currentChapter.title }}</p></div>
          <div v-if="lang === 'en'" class="english-summary"><span>CHAPTER GUIDE</span><p>{{ currentMeta.summary }}</p></div>
          <div v-if="lang === 'en'" class="source-label"><span>ORIGINAL KOREAN TEXT</span><p>The full source follows in Korean. The English guide above summarizes the chapter and does not replace the original wording.</p></div>
          <div class="chapter-blocks">
            <template v-for="(block, index) in currentChapter.blocks" :key="index">
              <h2 v-if="block.type === 'heading'" :id="'section-' + index">{{ block.text }}</h2>
              <div v-else-if="block.type === 'table'" class="table-scroll"><table><thead><tr><th v-for="(cell, cellIndex) in block.rows[0]" :key="cellIndex">{{ cell }}</th></tr></thead><tbody><tr v-for="(row, rowIndex) in block.rows.slice(1)" :key="rowIndex"><td v-for="(cell, cellIndex) in row" :key="cellIndex">{{ cell }}</td></tr></tbody></table></div>
              <p v-else-if="isExternalLink(block.text)" class="reference-link"><a :href="block.text" target="_blank" rel="noopener noreferrer">{{ block.text }} ↗</a></p>
              <p v-else>{{ block.text }}</p>
            </template>
          </div>
          <div class="chapter-pagination"><button v-if="previousPage" type="button" @click="navigate(previousPage.id)"><small>← {{ lang === 'ko' ? '이전 장' : 'PREVIOUS' }}</small><strong>{{ lang === 'ko' ? cleanTitle(previousPage) : chapterMeta[currentIndex - 1].en }}</strong></button><button v-else type="button" @click="navigate('intro')"><small>← {{ lang === 'ko' ? '이전' : 'PREVIOUS' }}</small><strong>{{ lang === 'ko' ? '소개' : 'Introduction' }}</strong></button><button v-if="nextPage" type="button" @click="navigate(nextPage.id)"><small>{{ lang === 'ko' ? '다음 장' : 'NEXT' }} →</small><strong>{{ lang === 'ko' ? cleanTitle(nextPage) : chapterMeta[currentIndex + 1].en }}</strong></button></div>
        </article>

        <footer class="article-footer">
          <img :src="asset('surfx-neocode-collab.png')" alt="SurfX × NEOCODE" />
          <div class="footer-details"><strong>Neocode</strong><span>Business Type: Company · Representative: Eunseong Park · Business Registration No. 422-29-01059</span><span>E-Commerce Registration No. 2025-Seoul Guro-0315 · D-U-N-S No. 963437563</span><span>Suite 702, 36, Gyeongin-ro 20na-gil, Guro-gu, Seoul, Republic of Korea</span><span>Email: <a href="mailto:neocode.kr@gmail.com">neocode.kr@gmail.com</a> · Customer Support: <a href="tel:+821086741663">010-8674-1663</a></span></div>
          <div class="footer-bottom"><span>© 2026 SurfX × NEOCODE</span><a href="https://www.surfx-protocol.com/">{{ lang === 'ko' ? 'SurfX 메인 사이트' : 'SurfX main site' }} ↗</a></div>
        </footer>
      </main>

      <aside class="on-this-page" :aria-label="lang === 'ko' ? '이 페이지에서' : 'On this page'"><div><h2>{{ lang === 'ko' ? '이 페이지에서' : 'ON THIS PAGE' }}</h2><template v-if="currentChapter"><button v-for="item in toc" :key="item.index" type="button" :class="{ selected: activeAnchor === 'section-' + item.index }" @click="jumpTo(item.index)">{{ item.text }}</button><span v-if="!toc.length" class="toc-empty">{{ lang === 'ko' ? '본문과 표' : 'Source text and tables' }}</span></template><template v-else><button type="button" @click="jumpToSelector('.intro-copy')">{{ lang === 'ko' ? '프로젝트 개요' : 'Project overview' }}</button><button type="button" @click="jumpToSelector('.intro-highlights')">{{ lang === 'ko' ? '문서 구성' : 'Document at a glance' }}</button></template></div></aside>
    </div>

    <div v-if="searchOpen" class="search-overlay" @click.self="searchOpen = false"><div class="search-dialog" role="dialog" aria-modal="true" :aria-label="lang === 'ko' ? '백서 검색' : 'Search whitepaper'"><div class="search-input-row"><span>⌕</span><input ref="searchInput" v-model="searchQuery" :placeholder="lang === 'ko' ? '제목과 본문 검색…' : 'Search titles and content…'" @keydown.enter="searchResults[0] && navigate(searchResults[0].id)" /><button type="button" @click="searchOpen = false">Esc</button></div><div class="search-results"><p v-if="!searchQuery" class="search-hint">{{ lang === 'ko' ? '장 제목과 백서 본문을 검색할 수 있습니다.' : 'Search all chapters and the original source text.' }}</p><p v-else-if="!searchResults.length" class="search-hint">{{ lang === 'ko' ? '검색 결과가 없습니다.' : 'No results found.' }}</p><button v-for="result in searchResults" :key="result.id" type="button" @click="navigate(result.id)"><span>{{ lang === 'ko' ? result.title : result.englishTitle }}</span><small>{{ lang === 'ko' ? result.blocks.find((block) => block.type === 'paragraph')?.text.slice(0, 110) : result.summary.slice(0, 110) }}…</small></button></div></div></div>
  </div>
</template>
