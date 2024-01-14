<script lang="ts" setup>

import { useViewColumnsOrThrow } from '#imports'
import type { UploadChangeParam, UploadFile } from 'ant-design-vue'
import { Upload } from 'ant-design-vue'
import { toRaw, unref } from '@vue/runtime-core'
import type { ImportWorkerPayload, importFileList, streamImportFileList } from '#imports'
import {
    BASE_FALLBACK_URL,
    CSVTemplateAdapter,
    ExcelTemplateAdapter,
    ExcelUrlTemplateAdapter,
    Form,
    ImportSource,
    ImportType,
    ImportWorkerOperations,
    ImportWorkerResponse,
    JSONTemplateAdapter,
    JSONUrlTemplateAdapter,
    computed,
    extractSdkResponseErrorMsg,
    fieldRequiredValidator,
    iconMap,
    importCsvUrlValidator,
    importExcelUrlValidator,
    importUrlValidator,
    initWorker,
    message,
    reactive,
    ref,
    storeToRefs,
    useBase,
    useGlobal,
    useI18n,
    useNuxtApp,
    useVModel,
} from '#imports'
import importWorkerUrl from '~/workers/importWorker?worker&url'

import axios from 'axios';
interface Props {
    modelValue: boolean
    importType: 'csv' | 'json' | 'excel'
    sourceId: string
    importDataOnly?: boolean
}
const templateData = ref();

const importData = ref();

const importColumns = ref([]);

const isWorkerSupport = typeof Worker !== 'undefined';

let importWorker: Worker | null;

const config = useRuntimeConfig();

const { token } = useGlobal();

const { appInfo } = useGlobal();

const { tables } = storeToRefs(useBase());

const templateEditorModal = ref(false);

const { importType, importDataOnly = false, sourceId, ...rest } = defineProps<Props>();

const progressMsg = ref('Parsing Data ...');


const { showEnrichModal, showInputModal, showEditModal } = storeToRefs(useShare())

const { filteredFieldList } = useViewColumnsOrThrow();

const FieldTitle = filteredFieldList.value;


const emit = defineEmits(['update:modelValue']);

const dialogShow = useVModel(rest, 'modelValue', emit);

const { t } = useI18n();

const importLoading = ref(false);

const templateEditorRef = ref();

const selectedOptionList = ref<string[]>();

const selectedOption = ref('')

const defaultImportState = {
    fileList: [] as importFileList | streamImportFileList,
    url: '',
    jsonEditor: {},
    parserConfig: {
        maxRowsToParse: 500,
        normalizeNested: true,
        autoSelectFieldTypes: true,
        firstRowAsHeaders: true,
        shouldImportData: true,
    },
}
const importState = reactive(defaultImportState)

let templateGenerator: CSVTemplateAdapter | JSONTemplateAdapter | ExcelTemplateAdapter | null

/** This is a function which scraps profile from LinkedIn.
 * Scrapping has been processing by third-party- workflow automatically.
 * */

const getScrapedData = () => {
    axios.get('https://workflows.saleshub.ai/api/workflow/11505/prospects', {
        timeout: 10000,
        headers: {
            'Authorization': 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwczpcL1wvYXBwLnVzZWRlbWFuZC5jb21cL2FwaVwvYWNjb3VudFwvbG9naW4iLCJpYXQiOjE3MDQyOTg3MzYsImV4cCI6MTcwNTUwODMzNiwibmJmIjoxNzA0Mjk4NzM2LCJqdGkiOiI3Z2xHdEYzWTZxdnNTV0xQIiwic3ViIjo1LCJwcnYiOiIyM2JkNWM4OTQ5ZjYwMGFkYjM5ZTcwMWM0MDA4NzJkYjdhNTk3NmY3Iiwic2NvcGUiOlsiZnVsbF9hY2Nlc3MiXX0.HRZAmRstvJ6uWTEstlsa5qqMezUBur0DhEeyuqiJXsY',
            'Content-Type': 'application/json'
        },
        data: {
            "name": "Nocodb5",
            "tag": "",
            "id": null,
            "userId": null,
            "flowchart": [
                {
                    "content": {
                        "value": "",
                        "searchURL": "https://www.linkedin.com/sales/search/people?query=(spellCorrectionEnabled%3Atrue%2CrecentSearchParam%3A(id%3A2646723569%2CdoLogHistory%3Atrue)%2Cfilters%3AList((type%3ALEAD_LIST%2Cvalues%3AList((id%3AALL%2Ctext%3AAll%2520lead%2520lists%2CselectionType%3AINCLUDED))))%2Ckeywords%3AQvist)&sessionId=XggVPe6%2FS9exZRibHQrsbg%3D%3D&viewAllFilters=true",
                        "profileNumber": "1"
                    },
                    "choosenFunction": "linkedin",
                    "id": "29e3035d-de3f-4dd3-930f-2d8fb3dcd643",
                    "prevNode": null
                }
            ],
            "status": "created"
        }
    }).then( async response => {
        const Scrap_Response = response.data;

        const scrapedData = Scrap_Response.data;
        const prospectArray = [{}]       

        
        scrapedData.forEach((element:any) => {
           // const displayData = filterFunc(element.single_prospect);
            prospectArray.push(JSON.parse(element.single_prospect));
             //console.log("displayData ===>>>", displayData)
            
        });

         prospectArray.shift();
         displayDataToTable(JSON.stringify(prospectArray));

    })
        .catch(error => {
            console.log(error)
        })

}

if (isWorkerSupport) {
    importWorker = await initWorker(importWorkerUrl)
}

/**
 * This function display the table to edit the scrapped data.
 * @param val The type of this value can be several type, but here its type is string.
 */
async function displayDataToTable(val: UploadFile[] | ArrayBuffer | string) {

    showEditModal.value = true
    templateData.value = null
    importData.value = null
    importColumns.value = []

    try {
        if (isWorkerSupport && importWorker) {
            importWorker.postMessage([
                ImportWorkerOperations.INIT_SDK,
                {
                    baseURL: config.public.ncBackendUrl || appInfo.value.ncSiteUrl || BASE_FALLBACK_URL,
                    token: token.value,
                },
            ])

            let value = toRaw(val)

            if (Array.isArray(value)) value = value.map((v) => toRaw(v))

            const payload = extractImportWorkerPayload(value)

            importWorker.postMessage([
                ImportWorkerOperations.SET_TABLES,
                unref(tables).map((t) => ({
                    table_name: t.table_name,
                    title: t.title,
                })),
            ])
            importWorker.postMessage([
                ImportWorkerOperations.SET_CONFIG,
                {
                    importDataOnly,
                    importColumns: !!importColumns.value,
                    importData: !!importData.value,
                },
            ])

            const response: {
                templateData: any
                importColumns: any
                importData: any
            } = await new Promise((resolve, reject) => {
                const handler = (e: MessageEvent) => {
                    const [type, payload] = e.data
                    switch (type) {
                        case ImportWorkerResponse.PROCESSED_DATA:
                            resolve(payload)
                            importWorker?.removeEventListener('message', handler, false)
                            break
                        case ImportWorkerResponse.PROGRESS:
                            progressMsg.value = payload
                            break
                        case ImportWorkerResponse.ERROR:
                            reject(payload)
                            importWorker?.removeEventListener('message', handler, false)
                            break
                    }
                }
                importWorker?.addEventListener('message', handler, false)

                importWorker?.postMessage([ImportWorkerOperations.PROCESS, payload])
            })

            templateData.value = response.templateData
            importColumns.value = response.importColumns
            importData.value = response.importData
        }

        templateEditorModal.value = true

    } catch (e: any) {
        console.log(e)
        message.error(await extractSdkResponseErrorMsg(e))
    }
}
/**
 * This function returns the final result.
 */
async function handleImport() {
    try {
        if (!templateGenerator && !importWorker) {
            message.error(t('msg.error.templateGeneratorNotFound'))
            return
        }
        importLoading.value = true
        await templateEditorRef.value.importTemplate()
    } catch (e: any) {
        return message.error(await extractSdkResponseErrorMsg(e))
    } finally {

        importLoading.value = false
        templateEditorModal.value = true
        Object.assign(importState, defaultImportState)
    }
    dialogShow.value = false
    showEditModal.value = false
}

function extractImportWorkerPayload(value: UploadFile[] | ArrayBuffer | string) {
    let payload: ImportWorkerPayload
    payload = {
        config: toRaw(importState.parserConfig),
        value,
        importType: ImportType.JSON,
        importSource: ImportSource.STRING,
    }
    return payload
}

/**
 * Select the table field that display finally.
 * @param event whenever select the fields, this event occurs.
 */
const onChange = (event: Event) => {
    selectedOption.value = (event.target as HTMLSelectElement).value;

    if (selectedOptionList.value === undefined) selectedOptionList.value = new Array();

    selectedOptionList.value?.push(selectedOption.value);
    console.log(selectedOptionList.value.at(0));
    console.log(typeof selectedOptionList.value.at(0));
};

function filterFunc (data_array:string){
    console.log("arr value and type=====>>>", data_array.at(0)," : ", typeof data_array.at(0))
    selectedOptionList.value?.forEach(val => {
        
    });
    
    return data_array
}

</script>
<template>
    <div class="flex flex-row items-center w-full gap-x-1 px-4 pt-4 pb-2 text-lg">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
            stroke="var(--colors--primaryText, #1d2026)" stroke-width="0.125rem" stroke-linecap="round"
            stroke-linejoin="round" style="flex-shrink: 0; width: 1rem; height: 1rem;">
            <path d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-2-2 2 2 0 0 0-2 2v7h-4v-7a6 6 0 0 1 6-6z"></path>
            <rect x="2" y="9" width="4" height="12"></rect>
            <circle cx="4" cy="4" r="2"></circle>
        </svg>
        <span>
            Company Profile
        </span>
    </div>
    <hr />
    <div class="flex flex-row items-center w-full gap-x-1 p-4 justify-between">
        <div>
            <h1><b>Use a single provider</b></h1>
            <span>Use a individual integration to find the data you're looking for</span>
        </div>
        <div class="flex flex-col items-left w-full gap-y-1 cursor-pointer hover:bg-gray-200 hover:rounded-lg p-2"
            @click="showEnrichModal = false, showInputModal = true">
            <div class="flex flex-row items-center w-full gap-x-1 self-center">
                <img src="https://clay-base-prod-static.s3.amazonaws.com/icons/new/linkedin.png"
                    class="AttributeExplorerItem__StyledActionIcon-doeSCW eQKPcj" style="width: 20px;">
                <h1 class="mb-0">Enrich Company</h1>
            </div>
            <div>
                <span>
                    Enrich a comapny with data from LinkedIn starting with a profile or validateDomains.
                    Note: This action may be...
                </span>
            </div>
        </div>
        <div class="bg-gray-200 px-2 rounded-lg">
            <span>free</span>
        </div>
    </div>
    <a-modal v-model:visible="showInputModal" class="!top-[1%] enrich-input-modal" :class="{ active: showInputModal }"
        size="small" :footer="null" width="min(100vw,500px)" style="float: right; margin-top:3%; margin-right:3%;"
        wrap-class-name="nc-modal-share-view">
        <div>
            <div class="flex flex-row items-center w-full gap-x-1 pb-3">
                <button v-if="showInputModal" aria-disabled="false" type="button"
                    class="Button__enabled-jFcQwC hIbQxO hover:bg-gray-200"
                    style="--flexGrow: 0; --flexShrink: 0; --color: var(--colors--accent, #038ff7);"
                    @click="showInputModal = false, showEnrichModal = true">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none"
                        stroke="var(--colors--accent, #038ff7)" stroke-width="0.1rem" stroke-linecap="round"
                        stroke-linejoin="round" style="flex-shrink: 0; width: 1.25rem; height: 1.25rem;">
                        <line x1="19" y1="12" x2="5" y2="12"></line>
                        <polyline points="12 19 5 12 12 5"></polyline>
                    </svg>
                </button>
                <span><b>Enrich Company</b></span>
            </div>
            <hr />
            <div class="pt-3">
                <span>Learn how to use: Enrich a Company with Lusha with Lusha starting with a company name or
                    Domain.</span>
                <!-- <div>
                    <span>CHOOSE LUSHA API KEY ACCOUNT</span>
                    <div>

                    </div>
                </div> -->
                <span>SETUP INPUTS</span>
                <div class="flex flex-col gap-y-4">
                    <a-collapse style="border:none">
                        <a-collapse-panel class="collapse-enrich" header="Company Name - Optional">
                            <select id="combobox" ref="comapny_name" class="combo-select" v-model="selectedOption"
                                v-on:change="onChange">
                                <option class="combo-option" v-for="option in FieldTitle" :key="option">{{ option.title }}
                                </option>
                            </select>
                        </a-collapse-panel>
                    </a-collapse>
                    <a-collapse style="border:none">
                        <a-collapse-panel class="collapse-enrich" header="Name - Optional">
                            <select ref="name" id="combobox" class="combo-select" v-model="selectedOption"
                                v-on:change="onChange">
                                <option v-for="option in FieldTitle" :key="option">{{ option.title }}</option>
                            </select>
                        </a-collapse-panel>
                    </a-collapse>
                    <a-collapse style="border:none">
                        <a-collapse-panel class="collapse-enrich" header="RUN SETTINGS">

                        </a-collapse-panel>
                    </a-collapse>
                </div>
            </div>
            <hr />
            <div>
                <NcButton v-e="['c:share:open']" :size="isMobileMode ? 'medium' : 'small'" class="z-10 !rounded-lg mt-3"
                    :class="{
                        '!px-2': !isMobileMode,
                        '!px-0 !max-w-8.5 !min-w-8.5': isMobileMode,
                    }" type="primary" :disabled="disabled" @click="showInputModal = false; getScrapedData();">
                    <div v-if="!isMobileMode" class="flex flex-row items-center w-full gap-x-1">
                        <MdiPlus class="h-3.5" />
                        <div class="flex">Continue to Add Fields</div>
                    </div>
                    <GeneralIcon v-else icon="mobileShare" />
                </NcButton>
            </div>
        </div>
    </a-modal>
    <a-modal v-model:visible="showEditModal" class="!top-[1%] enrich-input-modal" :class="{ active: showInputModal }"
        size="small" :footer="null" width="min(100vw,800px)" style=" margin-top:3%; margin-right:30%;"
        wrap-class-name="nc-modal-share-view">
        <div>
            <div class="mt-5">
                <LazyTemplateEditor v-if="templateEditorModal" ref="templateEditorRef" :base-template="templateData"
                    :import-data="importData" :import-columns="importColumns" :import-data-only="importDataOnly"
                    :quick-import-type="importType" :max-rows-to-parse="importState.parserConfig.maxRowsToParse"
                    :source-id="sourceId" :import-worker="importWorker" class="nc-quick-import-template-editor"
                    @import="handleImport" />
                <div class="mt-5" style="margin-left:80%">
                    <a-button key="import" type="primary" class="!rounded-md" :loading="importLoading"
                        @click="handleImport">
                        {{ $t('activity.import') }}
                    </a-button>
                </div>
            </div>
        </div>
    </a-modal>
</template>
<style scoped lang="scss">
// .enrich-input-modal {
//     float: right;
// }
.collapse-enrich {
    background-color: white;
    border: none;
}

// .ant-collapse-item:last-child>.ant-collapse-content {
//     border: none !important;;
// }
.ant-collapse-content.ant-collapse-content-active {
    border: none;
}

.combo-select {
    border-radius: 4px;
    border: 1px solid #d9d9d9;
    padding: 6px 12px;
    font-size: 14px;
    color: #333;
    width: 80%;
}

.combo-select:hover {
    border-color: #73a35c;
}

.combo-option {
    background-color: white;
}
</style>
