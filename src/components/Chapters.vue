<template>
  <section style="margin-bottom: 10px">
    <Toolbar
      :title="$t('chapters')"
      :menu-items="toolbarMenuItems"
      @title-clicked="toggleExpand"
    />
    <v-divider />
    <Container v-if="expanded">
      <v-list>
        <v-list-item
          v-for="chapter in itemDetails?.metadata?.chapters"
          :key="chapter.position"
          :disabled="!itemIsAvailable(itemDetails)"
          @click="chapterClicked(chapter)"
        >
          <template #prepend>
            <v-chip>
              {{ chapter.position }}
            </v-chip>
          </template>
          <template #title>
            <div style="margin-left: 15px">{{ chapter.name }}</div>
          </template>
          <template #append>
            <span v-if="chapter.end"
              >{{ formatDuration(chapter.end - chapter.start) }}
            </span>
          </template>
        </v-list-item>
      </v-list>
    </Container>
  </section>
</template>

<script setup lang="ts">
import { itemIsAvailable } from "@/plugins/api/helpers";
import { formatDuration } from "@/helpers/utils";
import { MediaItemChapter, type MediaItemType } from "@/plugins/api/interfaces";
import { api } from "@/plugins/api";
import Container from "@/components/mods/Container.vue";
import Toolbar from "@/components/Toolbar.vue";
import { computed, reactive, ref } from "vue";

export interface Props {
  itemDetails: MediaItemType;
}
const props = defineProps<Props>();

const expanded = ref(false);

const toggleExpand = function () {
  expanded.value = !expanded.value;
};

const toolbarMenuItems = computed(() => {
  return [
    // toggle expand
    {
      label: "tooltip.collapse_expand",
      icon: expanded.value ? "mdi-chevron-up" : "mdi-chevron-down",
      action: toggleExpand,
      overflowAllowed: false,
    },
  ];
});

const chapterClicked = function (chapter: MediaItemChapter) {
  if (!props.itemDetails || !itemIsAvailable(props.itemDetails)) return;
  api.playMedia(
    props.itemDetails.uri,
    undefined,
    undefined,
    chapter.position.toString(),
  );
};
</script>
