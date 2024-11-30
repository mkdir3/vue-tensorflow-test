<script lang="ts" setup>
import {Ref, ref} from "vue";
import {storeToRefs} from "pinia";
import {useObjectStore} from "@/stores/object";
import TextToSpeech from "@/components/TextToSpeech.vue";
import StatusCheckSimple from "@/components/StatusCheck.vue";

const image: Ref<File | any | undefined> = ref(undefined);
const imageToDetect: Ref<HTMLImageElement | undefined> = ref(undefined);
const url: Ref<string | undefined> = ref(undefined);
const modelLoaded: Ref<boolean> = ref(false);

const objectStore = useObjectStore();
const {detected} = storeToRefs(objectStore);
const {detect} = objectStore;

const detectObject = async (): Promise<void> => {
  await detect(imageToDetect.value);
};
const inputFromFile = (event: Event): void => {
  const target = event.target as HTMLInputElement;
  if (target.files && target.files[0]) {
    const file = target.files[0];
    image.value = [file];
    imageToDetect.value = dataToImageData(file);
  }
};

const dataToImageData = (dataBlob: Blob | MediaSource): HTMLImageElement => {
  const objUrl = URL.createObjectURL(dataBlob);
  const img = new Image();
  img.onload = () => URL.revokeObjectURL(img.src);
  img.src = objUrl;
  url.value = objUrl;
  return img;
};

const roundNumber = (num: number, decimal: number): number => {
  return Math.round(num * Math.pow(10, decimal)) / Math.pow(10, decimal);
};

const uniqueObjects = computed(() => {
  return detected.value
    .filter((v, i, a) => a.indexOf(v) === i)
    .map((v) => v.class);
});
const speech = computed(() => {
  const elements = uniqueObjects.value;
  return elements.length > 0
    ? `J'ai reconnu les éléments suivants sur l'image : ${new Intl.ListFormat('en').format(elements)}.`
    : "Je n'ai pas reconnu d'élément sur l'image, désolé.";
});
</script>
<template>
  <v-container class="py-5">
    <StatusCheckSimple @model-loaded="modelLoaded = true"/>
    <v-fade-transition>
      <v-card
        v-if="modelLoaded"
        class="pa-4"
        elevation="3"
        outlined
        rounded="lg"
      >
        <v-card-title class="text-h5 font-weight-bold mb-4">
          Détecter un objet dans une image
        </v-card-title>
        <v-divider/>
        <v-file-input
          v-model="image"
          :disabled="!modelLoaded"
          accept="image/png, image/jpeg"
          class="my-4"
          label="Choisissez une image png ou jpeg"
          outlined
          prepend-icon="mdi-image-outline"
          @change="inputFromFile"
        />
        <v-img
          v-if="url"
          :src="url"
          class="mb-4 elevation-2"
          contain
          height="300"
        />
        <v-btn
          :disabled="!image || !modelLoaded"
          append-icon="mdi-search-web"
          color="primary"
          @click="detectObject"
        >
          {{ detected.length ? 'Redétecter' : 'Détecter' }}
        </v-btn>
        <v-slide-y-transition>
          <v-card
            v-if="detected.length"
            class="mt-4"
            elevation="2"
            outlined
            rounded="lg"
          >
            <v-card-title class="text-h6 font-weight-medium">
              Résultats de Détection
            </v-card-title>
            <v-divider/>
            <v-table v-if="detected.length" class="mb-3" theme="dark">
              <thead>
              <tr>
                <th class="text-left">Nom</th>
                <th class="text-left">Score de précision</th>
              </tr>
              </thead>
              <tbody>
              <tr
                v-for="(item, index) in detected"
                :key="index"
              >
                <td>{{ item.class }}</td>
                <td>{{ roundNumber(item.score * 100, 2) }}%</td>
              </tr>
              </tbody>
            </v-table>
          </v-card>
        </v-slide-y-transition>
        <TextToSpeech
          :class="detected.length ? '' : 'ml-4'"
          :disabled="!detected.length || !speech"
          :message="speech"
        />
      </v-card>
    </v-fade-transition>
  </v-container>

</template>

