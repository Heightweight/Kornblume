<script setup lang="ts" name="PlannerView">
import { ref, computed, watchEffect } from 'vue'
import { onClickOutside } from '@vueuse/core'
import { useGlobalStore } from '@/stores/global'
import { usePlannerStore } from '@/stores/plannerStore'
import { useActivityStore } from '@/stores/activityStore'
import { useWildernessStore } from '@/stores/wildernessStore'
import { usePlannerSettingsStore } from '@/stores/plannerSettingsStore'
import { useDataStore } from '@/stores/dataStore'
import { IArcanist, ISelectedArcanist } from '@/types'
import { formatArcanists, sortSelectedArcanists } from '@/composables/arcanists'
import { GApiSvc, syncDrive } from '@/composables/gApi'

import PlannerSelector from '@/components/planner/PlannerSelector.vue'
import PlannerEdit from '@/components/planner/PlannerEdit.vue'
import PlannerActivity from '@/components/planner/PlannerActivity.vue'
import PlannerWilderness from '@/components/planner/PlannerWilderness.vue'
import PlannerWarehouse from '@/components/planner/PlannerWarehouse.vue'
import PlannerSettings from '@/components/planner/PlannerSettings.vue'
import PlannerTotal from '@/components/planner/PlannerTotal.vue'
import PlannerResult from '@/components/planner/PlannerResult.vue'
import ArcanistAddArcanistList from '@/components/arcanist/ArcanistAddArcanistList.vue'

const plannerStore = usePlannerStore()
const activityStore = useActivityStore()
const wildernessStore = useWildernessStore()
const arcanistStore = useDataStore().arcanists
const settingsStore = usePlannerSettingsStore()
const listArcanists = ref<IArcanist[]>([])

const selectedArcanistIds = computed(() =>
    plannerStore.selectedArcanists.map((arcanist) => arcanist.Id)
)

// TODO: need to add the type definition for these refs
const selectedArcanist = ref<ISelectedArcanist | null>(null) // Current working arcanist
const totalActivityAndDays = ref([]) // Total activity and days

const isAddingArcanist = ref(false)
const isEditingArcanist = ref(false)
const isActivity = ref(false)
const isWilderness = ref(false)
const isWarehouse = ref(false)
const isSettings = ref(false)

const activityRef = ref(null) // Ref to close modal
const arcanistListRef = ref(null)
const plannerEditRef = ref(null)
const wildernessRef = ref(null)
const warehouseRef = ref(null)
const settingsRef = ref(null)

const openAddOverlay = () => {
    isAddingArcanist.value = true
    useGlobalStore().setIsEditingPlanner(true);
}

const closeAddOverlay = () => {
    isAddingArcanist.value = false
    useGlobalStore().setIsEditingPlanner(false);
}

const openEditOverlay = () => {
    isEditingArcanist.value = true
    useGlobalStore().setIsEditingPlanner(true);
}

const editEditOverlay = (arcanist) => {
    // console.log(arcanist);
    selectedArcanist.value = arcanist
    openEditOverlay()
}

const closeEditOverlay = () => {
    isEditingArcanist.value = false
    useGlobalStore().setIsEditingPlanner(false);
}

const openActivity = () => {
    isActivity.value = true
    useGlobalStore().setIsEditingPlanner(true);
}

const closeActivity = () => {
    isActivity.value = false
    useGlobalStore().setIsEditingPlanner(false);
}

const openWilderness = () => {
    isWilderness.value = true
    useGlobalStore().setIsEditingPlanner(true);
}

const closeWilderness = () => {
    isWilderness.value = false
    useGlobalStore().setIsEditingPlanner(false);
}

const openWarehouse = () => {
    isWarehouse.value = true
    useGlobalStore().setIsEditingPlanner(true);
}

const closeWarehouse = () => {
    isWarehouse.value = false
    useGlobalStore().setIsEditingPlanner(false);
}

const openSettings = () => {
    isSettings.value = true
    useGlobalStore().setIsEditingPlanner(true);
}

const closeSettings = () => {
    isSettings.value = false
    useGlobalStore().setIsEditingPlanner(false);
}

const handleSelectArcanist = (arcanist: IArcanist) => {
    selectedArcanist.value = {
        Id: arcanist.Id,
        isVisible: true,
        currentInsight: 0,
        currentLevel: 1,
        currentResonance: 0,
        goalInsight: 0,
        goalLevel: 1,
        goalResonance: 0
    }
    // console.log(selectedArcanist.value);
    openEditOverlay()
}

const handleUpdateSelectedArcanists = (updateSelectedArcanists) => {
    sortSelectedArcanists(updateSelectedArcanists, arcanistStore)
    plannerStore.selectedArcanists = updateSelectedArcanists
    // console.log(selectedArcanists.value);
    closeEditOverlay()
    syncDrive();
}

const handleUpdateListArcanists = (updateListArcanists) => {
    listArcanists.value = updateListArcanists
}

const handleUpdateTotalActivityAndDays = (result) => {
    totalActivityAndDays.value = result
}

const handleSaveActivitySettings = (result) => {
    activityStore.settings = result
    // console.log(activitySettings.value);
}

const handleSaveWildernessSettings = (result) => {
    wildernessStore.settings = result
    // console.log(wildernessSettings.value);
}

const handleSaveSettings = (result) => {
    settingsStore.settings = result
    // console.log(result);
}

watchEffect(() => {
    listArcanists.value = arcanistStore.filter((arcanist: IArcanist) =>
        !selectedArcanistIds.value.includes(arcanist.Id)
    )
    listArcanists.value = formatArcanists(listArcanists.value);
})

onClickOutside(activityRef, closeActivity)
onClickOutside(arcanistListRef, closeAddOverlay)
onClickOutside(plannerEditRef, closeEditOverlay)
onClickOutside(wildernessRef, closeWilderness)
onClickOutside(warehouseRef, closeWarehouse)
onClickOutside(settingsRef, closeSettings)

GApiSvc.init().then(async () => {
    syncDrive();
});
</script>

<template>
    <div class="responsive-spacer">
        <!-- Selector -->
        <div class="flex pb-4">
            <h2 class="text-2xl text-white font-bold">{{ $t('planner') }}</h2>
        </div>

        <!-- Notification -->
        <div class="pb-4">
            <div role="alert" class="alert alert-info custom-gradient-gray-blue text-white">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
                    class="stroke-current shrink-0 w-6 h-6">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                        d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                </svg>
                <div>
                    <!-- <p class="text-sm lg:text-base"> Fixed 1.4 items not being added to Warehouse. </p>
                    <p class="text-sm lg:text-base"> If you see low opacity materials in Crafting section, ignore it. The issue
                        will be fixed once the data is fully gathered. </p> -->
                    <!-- <p class="text-sm lg:text-base"> Planner has been modified and upgraded. Enjoy almost lag free planning experience. </p> -->
                    <p class="text-sm lg:text-base"> Google Drive Sync has been added. Head to Profile to try it out.</p>
                </div>
            </div>
        </div>

        <PlannerSelector :selectedArcanists="plannerStore.selectedArcanists" @open-edit-overlay="editEditOverlay" />

        <div class="flex flex-wrap justify-between items-center mb-2 mt-2 gap-y-3">
            <button @click="openAddOverlay" class="btn btn-ghost btn-sm custom-gradient-button"><i
                    class="fa-solid fa-wand-magic-sparkles"></i> {{ $t('add-arcanist') }}</button>
            <div class="flex space-x-2">
                <div class="tooltip" :data-tip="$t('activity-settings')">
                    <button @click="openActivity" class="btn btn-ghost btn-sm custom-gradient-button rounded"><i
                            class="fa-solid fa-bolt"></i></button>
                </div>
                <div class="tooltip" :data-tip="$t('wilderness-settings')">
                    <button @click="openWilderness" class="btn btn-ghost btn-sm custom-gradient-button"><i
                            class="fa-solid fa-tree"></i></button>
                </div>
                <div class="tooltip tooltip-left" :data-tip="$t('manage-warehouse')">
                    <button @click="openWarehouse" class="btn btn-ghost btn-sm custom-gradient-button"><i
                            class="fa-solid fa-box-archive"></i></button>
                </div>
                <!-- Hide this settings button till we actually have some settings that allow them to make changes -->
                <div v-if="false" class="tooltip tooltip-left" :data-tip="$t('settings')">
                    <button @click="openSettings" class="btn btn-ghost btn-sm custom-gradient-button"><i
                            class="fa-solid fa-gear"></i></button>
                </div>
            </div>
        </div>

        <div class="custom-line"></div>

        <PlannerTotal :totalActivityAndDays="totalActivityAndDays" :wildernessSettings="wildernessStore.settings" />

        <div class="custom-line"></div>

        <!-- Add Arcanist Overlay -->
        <div v-if="isAddingArcanist" class="overlay">
            <ArcanistAddArcanistList ref="arcanistListRef" :arcanists="listArcanists" @closeOverlay="closeAddOverlay"
                @selectArcanist="handleSelectArcanist" />
        </div>

        <!-- Edit Arcanist Overlay -->
        <div v-if="isEditingArcanist" class="overlay">
            <PlannerEdit ref="plannerEditRef" v-if="selectedArcanist" :selectedArcanist="selectedArcanist"
                :selectedArcanists="plannerStore.selectedArcanists" :listArcanists="listArcanists"
                @closeOverlay="closeEditOverlay" @updateSelectedArcanists="handleUpdateSelectedArcanists"
                @updateListArcanists="handleUpdateListArcanists" />
        </div>

        <!-- Activity Overlay -->
        <div v-if="isActivity" class="overlay">
            <PlannerActivity ref="activityRef" :settings="activityStore.settings" @closeOverlay="closeActivity"
                @saveActivitySettings="handleSaveActivitySettings" />
        </div>

        <!-- Wilderness Overlay -->
        <div v-if="isWilderness" class="overlay">
            <PlannerWilderness ref="wildernessRef" :settings="wildernessStore.settings" @closeOverlay="closeWilderness"
                @saveWildernessSettings="handleSaveWildernessSettings" />
        </div>

        <!-- Warehouse Overlay -->
        <div v-if="isWarehouse" class="overlay">
            <PlannerWarehouse ref="warehouseRef" @closeOverlay="closeWarehouse" />
        </div>

        <!-- Settings Overlay -->
        <div v-if="isSettings" class="overlay">
            <PlannerSettings ref="settingsRef" :settings="settingsStore.settings" @closeOverlay="closeSettings"
                @saveSettings="handleSaveSettings" />
        </div>

        <!-- Result -->
        <PlannerResult @update:totalActivityAndDays="handleUpdateTotalActivityAndDays" />
    </div>
</template>

<style scoped>
/* * {
  border: 1px solid red !important;
} */
</style>
