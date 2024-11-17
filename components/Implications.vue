<template>
  <div class="w-full bg-sky-50">
    <h2 class="text-2xl font-bold p-2 bg-sky-100 text-black text-center">{{ currentItem.title }}</h2>
    <div class="p-4">
      <div class="bg-white shadow-md rounded-lg p-4 w-10/12 mx-auto">
        <div class="flex flex-col gap-6">
          <div v-for="level in maxLevel" :key="`level-${level}`" class="flex flex-col">
            <div class="grid grid-cols-3 gap-4">
              <!-- Left Section -->
              <template v-if="hasContent(level, 'left')">
                <ContentBlockList :content="getContent(level, 'left')" />
              </template>
              <template v-else-if="hasContent(level, 'left-center')">
                <ContentBlockList :content="getContent(level, 'left-center')" class="col-span-2" />
              </template>

              <!-- Center Section -->
              <template v-if="!hasContent(level, 'left-center') && !hasContent(level, 'right-center')">
                <ContentBlockList :content="getContent(level, 'center')" />
              </template>

              <!-- Right Section -->
              <template v-if="hasContent(level, 'right')">
                <ContentBlockList :content="getContent(level, 'right')" />
              </template>
              <template v-else-if="hasContent(level, 'right-center')">
                <ContentBlockList :content="getContent(level, 'right-center')" class="col-span-2" />
              </template>

              <!-- Full Width Section -->
              <template v-if="hasContent(level, 'all')">
                <ContentBlockList :content="getContent(level, 'all')" class="col-span-3" />
              </template>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const ContentBlockList = defineComponent({
  name: 'ContentBlockList',
  props: {
    content: {
      type: Array,
      required: true
    }
  },
  render() {
    return h('div', { class: 'flex flex-col space-y-4' },
      this.content.map(item => h(ContentBlock, {
        key: item.id,
        content: item
      }))
    );
  }
});

const ContentBlock = defineComponent({
  name: 'ContentBlock',
  props: {
    content: {
      type: Object,
      required: true
    }
  },
  render() {
    const components = {
      subtitle: () => h('h3', {
        class: 'text-lg font-semibold text-sky-900 mb-2'
      }, this.content.content),
      
      image: () => h('div', { class: 'w-full' }, [
        h('img', {
          src: processImageSource(this.content.content),
          alt: this.content.alt || 'Image',
          class: 'w-full rounded-lg shadow-md',
          loading: 'lazy'
        }),
        this.content.caption && h('p', {
          class: 'text-xs text-black mt-1 italic'
        }, this.content.caption)
      ]),
      
      text: () => h('p', {
        class: 'text-sm text-black text-justify'
      }, this.content.content)
    };

    return components[this.content.type]?.() || null;
  }
});

const props = defineProps({
  currentItem: {
    type: Object,
    required: true
  }
});

const VALID_LAYOUTS = ['left', 'center', 'right', 'left-center', 'right-center', 'all'];

const parseContent = (segment, index) => {
  const patterns = {
    subtitle: /^##\s*(.*?)(?:\[(.*?)\])?(?:\[(\d+)\])?$/,
    text: /^{(.*?)}(?:\[(.*?)\])?(?:\[(\d+)\])?$/s,
    image: /!\[(.*?)\]\((.*?)\)(?:{(.*?)})?(?:\[(.*?)\])?(?:\[(\d+)\])?/
  };

  const getValidLayout = (layout, defaultLayout) =>
    VALID_LAYOUTS.includes(layout?.toLowerCase()) ? layout.toLowerCase() : defaultLayout;

  if (patterns.subtitle.test(segment)) {
    const [, content, layout, level] = segment.match(patterns.subtitle);
    return {
      id: `subtitle-${index}`,
      type: 'subtitle',
      content: content.trim(),
      layout: getValidLayout(layout, 'all'),
      level: parseInt(level) || 1
    };
  }

  if (patterns.text.test(segment)) {
    const [, content, layout, level] = segment.match(patterns.text);
    return {
      id: `text-${index}`,
      type: 'text',
      content: content.trim(),
      layout: getValidLayout(layout, 'right'),
      level: parseInt(level) || 1
    };
  }

  if (patterns.image.test(segment)) {
    const [, alt, src, caption, layout, level] = segment.match(patterns.image);
    return {
      id: `image-${index}`,
      type: 'image',
      content: src,
      alt,
      caption: caption || '',
      layout: getValidLayout(layout, 'left'),
      level: parseInt(level) || 1
    };
  }

  return {
    id: `default-${index}`,
    type: 'text',
    content: segment,
    layout: 'right',
    level: 1
  };
};

const processedContent = computed(() => 
  props.currentItem.description
    .split('\n\n')
    .filter(segment => segment.trim())
    .map(parseContent)
);

const maxLevel = computed(() => 
  Math.max(...processedContent.value.map(item => item.level))
);

const processImageSource = (src) => {
  if (src.startsWith('content:/')) return src.replace('content:/', '/');
  return src.startsWith('/') ? src : src;
};

const hasContent = (level, position) => 
  processedContent.value.some(item => item.level === level && item.layout === position);

const getContent = (level, position) => 
  processedContent.value.filter(item => item.level === level && item.layout === position);
</script>

<style scoped>
.min-h-0 {
  min-height: 0;
}
</style>