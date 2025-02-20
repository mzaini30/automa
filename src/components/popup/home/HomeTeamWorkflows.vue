<template>
  <div class="px-5 pb-5 space-y-2">
    <ui-card
      v-for="workflow in workflows"
      :key="workflow.id"
      class="w-full flex items-center relative space-x-2 hover:ring-2 hover:ring-gray-900"
    >
      <div
        class="flex-1 text-overflow cursor-pointer mr-4"
        @click="openWorkflowPage(workflow)"
      >
        <p class="leading-tight text-overflow">{{ workflow.name }}</p>
        <div class="text-gray-500 flex items-center">
          <span>{{ dayjs(workflow.createdAt).fromNow() }}</span>
          <div class="flex-grow" />
          <span
            :class="tagColors[workflow.tag]"
            class="text-overflow px-2 text-sm ml-2 text-gray-600 py-1 rounded-md"
            style="max-width: 120px"
          >
            {{ workflow.tag }}
          </span>
        </div>
      </div>
      <p v-if="workflow.isDisabled" class="text-sm text-gray-600">Disabled</p>
      <button v-else title="Execute" @click="executeWorkflow(workflow)">
        <v-remixicon name="riPlayLine" />
      </button>
    </ui-card>
  </div>
</template>
<script setup>
import { computed, onMounted, shallowRef } from 'vue';
import { useUserStore } from '@/stores/user';
import { sendMessage } from '@/utils/message';
import { useTeamWorkflowStore } from '@/stores/teamWorkflow';
import { tagColors } from '@/utils/shared';
import { executeWorkflow } from '@/newtab/workflowEngine';
import dayjs from '@/lib/dayjs';

const props = defineProps({
  search: {
    type: String,
    default: '',
  },
});

const userStore = useUserStore();
const teamWorkflowStore = useTeamWorkflowStore();

const teamWorkflows = shallowRef([]);

const workflows = computed(() =>
  teamWorkflows.value.filter((workflow) =>
    workflow.name.toLocaleLowerCase().includes(props.search.toLocaleLowerCase())
  )
);

function openWorkflowPage({ teamId, id }) {
  const url = `/teams/${teamId}/workflows/${id}`;
  sendMessage('open:dashboard', url, 'background');
}

onMounted(() => {
  if (!userStore.user?.teams) return;

  teamWorkflows.value = userStore.user.teams
    .reduce((acc, team) => {
      const currentWorkflows = teamWorkflowStore
        .getByTeam(team.id)
        .map((workflow) => {
          workflow.teamId = team.id;
          workflow.teamName = team.name;

          return workflow;
        });
      acc.push(...currentWorkflows);

      return acc;
    }, [])
    .sort((a, b) => (a.createdAt > b.createdAt ? 1 : -1));
});
</script>
