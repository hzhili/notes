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

const textColor = ref('var(--vp-c-text-3)')



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

const analyzeImageBrightness = async (imageUrl: string): Promise<boolean> => {
  return new Promise((resolve) => {
    const img = new Image()
    img.crossOrigin = 'Anonymous'
    img.onload = () => {
      console.log('Image loaded successfully')
      const canvas = document.createElement('canvas')
      const ctx = canvas.getContext('2d')
      if (!ctx) {
        resolve(false)
        return
      }
      
      canvas.width = img.width
      canvas.height = img.height
      
      try {
        ctx.drawImage(img, 0, 0)
        
        try {
          const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
          const data = imageData.data
          
          let totalBrightness = 0
          for (let i = 0; i < data.length; i += 4) {
            const r = data[i]
            const g = data[i + 1]
            const b = data[i + 2]
            const brightness = (r * 299 + g * 587 + b * 114) / 1000
            totalBrightness += brightness
          }
          
          const avgBrightness = totalBrightness / (data.length / 4)
          const isDarkBackground = avgBrightness < 128
          
          resolve(isDarkBackground)
        } catch (corsError) {
          console.warn('CORS error when analyzing image brightness:', corsError)
          resolve(false)
        }
      } catch (error) {
        console.warn('Error drawing image:', error)
        resolve(false)
      }
    }
    
    img.onerror = () => {
      resolve(false)
    }
    
    img.src = imageUrl
  })
}

const updateTextColor = async () => {
  const bgImage = props.backgroundImage
  
  if (!bgImage || typeof bgImage !== 'string' || !isLinkHttp(bgImage)) {
    textColor.value = 'var(--vp-c-text-3)'
    return
  }
  
  try {
    const isDarkBackground = await analyzeImageBrightness(bgImage)
    console.log('isDarkBackground:', isDarkBackground)
    if (isDarkBackground) {
      textColor.value = '#ffffff'
    } else {
      textColor.value = '#1a1a1a'
    }
  } catch (error) {
    console.warn('Error updating text color:', error)
    textColor.value = 'var(--vp-c-text-3)'
  }
}


onMounted(() => {
  fetchDailyQuote()
  updateTextColor()
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

.dhero-container :deep(.hero-name) {
  font-size: 48px !important;
  font-weight: 800 !important;
  color: #ffffff !important;
  text-shadow: 
    0 2px 4px rgba(0, 0, 0, 0.8),
    0 4px 8px rgba(0, 0, 0, 0.6),
    0 8px 16px rgba(0, 0, 0, 0.4);
  margin-bottom: 16px !important;
}

.dhero-container :deep(.hero-tagline) {
  font-size: 24px !important;
  font-weight: 600 !important;
  color: #ffffff !important;
  text-shadow: 
    0 2px 4px rgba(0, 0, 0, 0.8),
    0 4px 8px rgba(0, 0, 0, 0.6);
  margin-bottom: 32px !important;
}
</style>