<script setup lang="ts">
import type { ThemeHomeHero } from 'vuepress-theme-plume'
import {  VPHomeHero } from 'vuepress-theme-plume/client'
import { useData } from 'vuepress-theme-plume/client';
import { computed, ref, watch, onMounted } from 'vue'
import { isLinkHttp } from 'vuepress/shared';


const props = defineProps<ThemeHomeHero>()
const { frontmatter, isDark } = useData<'home'>()

const heroTemp = computed(() => props.hero ?? frontmatter.value.hero ?? {})
const hero = ref({name: heroTemp.value.name ?? '', tagline: heroTemp.value.tagline ?? '', text:'', actions: heroTemp.value.actions ?? []})




const fetchDailyQuote = async () => {
  if (!heroTemp.value.text || !isLinkHttp(heroTemp.value.text)) {
    hero.value.text = heroTemp.value.text ?? ''
    return
  }
  
  try {
    const response = await fetch(heroTemp.value.text)
    if (response.ok) {
      console.log('获取每日一言:', response)
      const data = await response.text();
      const jsonData = JSON.parse(data);
      hero.value.text = `${jsonData.hitokoto} —— ${jsonData.from || '佚名'}`;
      console.log("重新赋值:", hero.value.text)
    } 
  } catch (error) {
  }
}



onMounted(() => {
  fetchDailyQuote()
})
</script>

<template>
  <div class="dhero-container">
    <VPHomeHero
      type="hero"
      :index="0"
      :hero="hero"
      :full="props.full"
      :effect="props.effect"
      :effectConfig="props.effectConfig"
      :filter="props.filter"
      :forceDark="props.forceDark"
      :background-attachment="props.backgroundAttachment"
      :background-image="props.backgroundImage"
    />
  </div>
</template>

<style scoped>
.dhero-container {
  position: relative;
}

.dhero-container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(rgba(0, 0, 0, 0.4), rgba(0, 0, 0, 0.6));
  z-index: 1;
}

.dhero-container > * {
  position: relative;
  z-index: 2;
}

.dhero-container :deep(.hero-text) {
  font-size: 20px;
  font-weight: 600;
  color: #ffffff !important;
  white-space: pre-wrap;
  pointer-events: auto;
  transition: all var(--vp-t-color);
  text-shadow: 
    0 2px 4px rgba(0, 0, 0, 0.8),
    0 4px 8px rgba(0, 0, 0, 0.6),
    0 8px 16px rgba(0, 0, 0, 0.4);
  padding: 15px 25px;
  background-color: rgba(0, 0, 0, 0.3);
  border-radius: 12px;
  display: inline-block;
  margin: 0 auto;
  backdrop-filter: blur(2px);
}


</style>