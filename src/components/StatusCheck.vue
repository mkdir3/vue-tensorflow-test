<script lang="ts" setup>
import {watch} from "vue";
import {storeToRefs} from "pinia";
import {useObjectStore} from "@/stores/object";

const objectStore = useObjectStore();
const {isModelLoading, isModelLoaded} = storeToRefs(objectStore);

const emit = defineEmits(["model-loaded"]);
watch(isModelLoaded, () => {
  if (isModelLoaded.value) emit("model-loaded");
});
</script>
<template>
  <v-card
    class="my-4"
    elevation="3"
    outlined
    rounded="lg"
  >
    <v-card-title class="text-h6 font-weight-bold">
      Configuration du modèle
    </v-card-title>
    <v-divider />
    <v-card-text>
      <div class="d-flex justify-space-between align-center">
        <div class="d-flex align-center">
          <v-icon
            class="mr-2"
            color="primary"
            size="24"
          >
            mdi-brain
          </v-icon>
          <span class="font-weight-medium text-subtitle-1">Coco SSD</span>
        </div>
        <v-fade-transition>
          <div
            v-if="isModelLoading"
            class="d-flex align-center"
          >
            <span class="mr-2 text-body-2">Chargement du modèle en cours...</span>
            <v-progress-circular
              :size="20"
              color="primary"
              indeterminate
            />
          </div>
        </v-fade-transition>
      </div>
      <v-slide-y-transition>
        <v-list-item-subtitle
          v-if="isModelLoaded"
          class="pt-4 text-success font-weight-medium text-body-2"
        >
          <v-icon
            class="mr-1"
            color="success"
            icon="mdi-check-circle"
            size="20"
          />
          Modèle chargé, vous pouvez dès à présent l'utiliser!
        </v-list-item-subtitle>
      </v-slide-y-transition>
    </v-card-text>
  </v-card>
</template>
