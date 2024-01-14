<script lang="ts" setup>

import type { NcButton } from '#build/components';
import { ref } from '#imports'

import CompanyEnrich from "./Enrich/CompanyEnrich.vue"


interface Props {
  disabled?: boolean
  isViewToolbar?: boolean
}

const viewComponentList = [
  { item: "Company Profile", component: CompanyEnrich },
  { item: "Job Openings", component: "Job Openings comp" }
]

const { showEnrichModal, showInputModal } = storeToRefs(useShare())
// let { isModalFirstPage } = storeToRefs(useShare())
let isModalFirstPage = ref(true);
const { isMobileMode } = useGlobal()
const emit = defineEmits(['submit', 'cancel', 'mounted', 'add', 'update'])

const { predictColumnType: _predictColumnType } = useNocoEe()
const { disabled } = defineProps<Props>()

const fieldsListWrapperDomRef = ref<HTMLElement>()

const getComponentByItem = (item: String) => {
  let component = null;
  viewComponentList.forEach(element => {
    if (element.item === item) {
      component = element.component;
    }
  });
  return component;
}

const showDetailItem = (item: String) => {
  watch(isModalFirstPage, () => {
    
  })
}
</script>

<template>
  <a-modal v-model:visible="showEnrichModal" class="!top-[1%]" :class="{ active: showEnrichModal }" size="small"
    :footer="null" width="min(100vw,720px)" wrap-class-name="nc-modal-share-view">
    <div class="flex flex-row items-center w-full gap-x-1">
      <button v-if="!isModalFirstPage" aria-disabled="false" type="button" class="Button__enabled-jFcQwC hIbQxO hover:bg-gray-200"
        style="--flexGrow: 0; --flexShrink: 0; --color: var(--colors--accent, #038ff7);" @click="showEnrichModal=true, isModalFirstPage=true">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
          stroke="var(--colors--accent, #038ff7)" stroke-width="0.1rem" stroke-linecap="round" stroke-linejoin="round"
          style="flex-shrink: 0; width: 1.25rem; height: 1.25rem;">
          <line x1="19" y1="12" x2="5" y2="12"></line>
          <polyline points="12 19 5 12 12 5"></polyline>
        </svg>
      </button>
      <div>
        <h1 class="mb-0">Add Enrichment Column</h1>
      </div>
    </div>
    <div class="flex flex-row rounded-lg border-1 border-gray-200">
      <div v-if="isModalFirstPage" ref="fieldsListWrapperDomRef"
        class="nc-scrollbar-md !overflow-auto flex-1 flex-grow-2 nc-fields-height px-4 pb-4 pt-4">
        <div>
          <div class="pb-4">Company Data</div>
          <div class="grid inline-grid grid-cols-4 gap-x-2 gap-y-2">
            <NcButton v-e="['c:share:open']" :size="isMobileMode ? 'medium' : 'small'" class="z-10 !rounded-lg" :class="{
              '!px-2': !isMobileMode,
              '!px-0 !max-w-8.5 !min-w-8.5': isMobileMode,
            }" type="secondary" :disabled="disabled" @click="isModalFirstPage = false">
              <div v-if="!isMobileMode" class="flex flex-row items-center w-full gap-x-1">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="var(--colors--primaryText, #1d2026)" stroke-width="0.125rem" stroke-linecap="round"
                  stroke-linejoin="round" style="flex-shrink: 0; width: 1rem; height: 1rem;">
                  <path d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-2-2 2 2 0 0 0-2 2v7h-4v-7a6 6 0 0 1 6-6z"></path>
                  <rect x="2" y="9" width="4" height="12"></rect>
                  <circle cx="4" cy="4" r="2"></circle>
                </svg>
                <div class="flex">Company Profile</div>
              </div>
              <GeneralIcon v-else icon="mobileShare" />
            </NcButton>
            <NcButton v-e="['c:share:open']" :size="isMobileMode ? 'medium' : 'small'" class="z-10 !rounded-lg" :class="{
              '!px-2': !isMobileMode,
              '!px-0 !max-w-8.5 !min-w-8.5': isMobileMode,
            }" type="secondary" :disabled="disabled" @click="showDetailItem('Job Openings')">
              <div class="flex flex-row items-center w-full gap-x-1">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="var(--colors--primaryText, #1d2026)" stroke-width="0.125rem" stroke-linecap="round"
                  stroke-linejoin="round" style="flex-shrink: 0; width: 1rem; height: 1rem;">
                  <path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2z"></path>
                  <path d="M22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z"></path>
                </svg>
                <div class="flex">Job Openings</div>
              </div>
            </NcButton>
            <NcButton v-e="['c:share:open']" :size="isMobileMode ? 'medium' : 'small'" class="z-10 !rounded-lg" :class="{
              '!px-2': !isMobileMode,
              '!px-0 !max-w-8.5 !min-w-8.5': isMobileMode,
            }" type="secondary" :disabled="disabled">
              <div class="flex flex-row items-center w-full gap-x-1">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="var(--colors--primaryText, #1d2026)" stroke-width="0.125rem" stroke-linecap="round"
                  stroke-linejoin="round" style="flex-shrink: 0; width: 1rem; height: 1rem;">
                  <path
                    d="M18 3a3 3 0 0 0-3 3v12a3 3 0 0 0 3 3 3 3 0 0 0 3-3 3 3 0 0 0-3-3H6a3 3 0 0 0-3 3 3 3 0 0 0 3 3 3 3 0 0 0 3-3V6a3 3 0 0 0-3-3 3 3 0 0 0-3 3 3 3 0 0 0 3 3h12a3 3 0 0 0 3-3 3 3 0 0 0-3-3z">
                  </path>
                </svg>
                <div class="flex">Tech Stack</div>
              </div>
            </NcButton>
            <NcButton v-e="['c:share:open']" :size="isMobileMode ? 'medium' : 'small'" class="z-10 !rounded-lg" :class="{
              '!px-2': !isMobileMode,
              '!px-0 !max-w-8.5 !min-w-8.5': isMobileMode,
            }" type="secondary" :disabled="disabled">
              <div class="flex flex-row items-center w-full gap-x-1">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="var(--colors--primaryText, #1d2026)" stroke-width="0.125rem" stroke-linecap="round"
                  stroke-linejoin="round" style="flex-shrink: 0; width: 1rem; height: 1rem;">
                  <path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path>
                  <path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path>
                </svg>
                <div class="flex">Domains</div>
              </div>
            </NcButton>
          </div>
        </div>

        <div>
          <div class="pt-4 pb-4">People</div>
          <div class="grid inline-grid grid-cols-4 gap-x-2 gap-y-2">
            <NcButton v-e="['c:share:open']" :size="isMobileMode ? 'medium' : 'small'" class="z-10 !rounded-lg" :class="{
              '!px-2': !isMobileMode,
              '!px-0 !max-w-8.5 !min-w-8.5': isMobileMode,
            }" type="secondary" :disabled="disabled">
              <div v-if="!isMobileMode" class="flex flex-row items-center w-full gap-x-1">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="var(--colors--primaryText, #1d2026)" stroke-width="0.125rem" stroke-linecap="round"
                  stroke-linejoin="round" style="flex-shrink: 0; width: 1rem; height: 1rem;">
                  <path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"></path>
                  <polyline points="22,6 12,13 2,6"></polyline>
                </svg>

                <!-- <MdiStar v-else-if="visibility === 'private'" class="h-3.5" /> -->
                <div class="flex">Personal Email</div>
              </div>
              <GeneralIcon v-else icon="mobileShare" />
            </NcButton>
            <NcButton v-e="['c:share:open']" :size="isMobileMode ? 'medium' : 'small'" class="z-10 !rounded-lg" :class="{
              '!px-2': !isMobileMode,
              '!px-0 !max-w-8.5 !min-w-8.5': isMobileMode,
            }" type="secondary" :disabled="disabled">
              <div class="flex flex-row items-center w-full gap-x-1">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="var(--colors--primaryText, #1d2026)" stroke-width="0.125rem" stroke-linecap="round"
                  stroke-linejoin="round" style="flex-shrink: 0; width: 1rem; height: 1rem;">
                  <path
                    d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z">
                  </path>
                </svg>
                <div class="flex">Phone Number</div>
              </div>
            </NcButton>
            <NcButton v-e="['c:share:open']" :size="isMobileMode ? 'medium' : 'small'" class="z-10 !rounded-lg" :class="{
              '!px-2': !isMobileMode,
              '!px-0 !max-w-8.5 !min-w-8.5': isMobileMode,
            }" type="secondary" :disabled="disabled">
              <div class="flex flex-row items-center w-full gap-x-1">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                  stroke="var(--colors--primaryText, #1d2026)" stroke-width="0.125rem" stroke-linecap="round"
                  stroke-linejoin="round" style="flex-shrink: 0; width: 1rem; height: 1rem;">
                  <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path>
                  <circle cx="12" cy="7" r="4"></circle>
                </svg>
                <div class="flex">Gender</div>
              </div>
            </NcButton>
          </div>
          <!-- <template>
            <div v-if="true"
              class="flex flex-col justify-center h-full" data-testid="share-base-button" >
              <NcButton>
                <div v-if="true" class="flex flex-row items-center w-full gap-x-1">
                  <MdiStar class="h-3.5" />
                  <div class="flex">Enrich Data</div>
                </div>
                <GeneralIcon v-else icon="mobileShare" />
              </NcButton>
            </div>
            
          </template> -->
          <!-- <div class="flex px-2 hover:bg-gray-100 first:rounded-t-lg last:rounded-b-none border-gray-200 pl-5 group">
            <div class="flex items-center flex-1 py-2.5 gap-1 w-2/6">
              Domains
            </div>
          </div> -->
          <!-- <div class="flex px-2 hover:bg-gray-100 first:rounded-t-lg  last:rounded-b-none border-gray-200 pl-5 group">
            <div class="flex items-center flex-1 py-2.5 gap-1 w-2/6">
              Job Listenings
            </div>
          </div>
          <div class="flex px-2 hover:bg-gray-100 first:rounded-t-lg  last:rounded-b-none border-gray-200 pl-5 group">
            <div class="flex items-center flex-1 py-2.5 gap-1 w-2/6">
              Tech Stack
            </div>
          </div> -->
        </div>
      </div>
      <div v-else="!isModalFirstPage" class="px-2 pb-2">
        <CompanyEnrich />
        <!-- {{ viewComponentList[0].component }} -->
      </div>
      <div>

      </div>
      <!-- <Transition v-if="!changingField" name="slide-fade">
        <div class="border-gray-200 border-l-1 rounded-r-xl h-[calc(100vh-(var(--topbar-height)*3.85))]">
          <SmartsheetColumnEditOrAddProvider v-if="activeField" class="p-4 w-[25rem]" :column="activeField"
            :preload="fieldState(activeField)" :table-explorer-columns="fields" embed-mode from-table-explorer
            @update="onFieldUpdate" @add="onFieldAdd" />
          <div v-else class="w-[25rem] flex flex-col justify-center p-4 items-center">
            <img src="~assets/img/fieldPlaceholder.svg" class="!w-[18rem]" />
            <div class="text-2xl text-gray-600 font-bold text-center pt-6">Select a field</div>
            <div class="text-center text-sm px-2 text-gray-500 pt-6">
              Make changes to field properties by selecting a field from the list
            </div>
          </div>
        </div>
      </Transition> -->
    </div>
  </a-modal>

</template>

<style lang="scss" scoped>
.share-collapse-item {
  @apply !rounded-lg !mb-2 !mt-4 !border-0;
}

.ant-collapse {
  @apply !bg-white !border-0;
}
</style>

<style lang="scss">
.nc-modal-share-collaborate {
  .ant-modal {
    top: 10vh !important;
  }

  .share-view,
  .share-base {
    @apply !border-1 border-gray-200 mx-3 rounded-lg mt-3 px-1 py-1;
  }

  .ant-collapse-item {
    @apply !border-1 border-gray-100;
  }

  .ant-collapse-content {
    @apply !border-t-0;
  }

  .ant-collapse-content-box {
    @apply !p-0;
  }

  .ant-modal-content {
    @apply !rounded-lg !px-1 !py-2;
  }

  .ant-select-selector {
    @apply !rounded-md !border-gray-200 !border-1;
  }

  .ant-form-item {
    @apply !my-0;
  }

  .ant-form-item-explain {
    @apply !ml-3;
  }

  .ant-select {
    @apply !p-0.5;
  }

  .ant-select-selector {
    @apply !bg-white;
  }

  ul {
    list-style-type: none;
    margin: 5vw, 5vw, 5vw, 5vw;
    padding-left: 10px;
    padding-right: 10px;
    width: 10vw;
    border-right: solid 1px;
    border-color: grey;
    background-color: white;
  }

  li div {
    display: block;
    color: #000;
    padding: 8px 16px;
    font-size: 18px;
    text-decoration: none;
  }

  .ul-container {
    list-style-type: none;
    margin: 5vw, 5vw, 5vw, 5vw;
    padding-left: 10px;
    padding-right: 10px;
    width: 30vw;
    background-color: white;
  }

  .div-container {
    display: block;
    color: #000;
    padding: 8px 16px;
    font-size: 18px;
    text-decoration: none;
  }

  .div-container:hover {
    background-color: lightgrey;
    color: white;
  }

  /* Change the link color on hover */
  li div:hover {
    background-color: grey;
    color: white;
  }
}
</style>
