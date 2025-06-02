<script setup lang="ts">
import { isChatNode } from '@/components/CanvasChat/utils';
import KeyboardShortcutTooltip from '@/components/KeyboardShortcutTooltip.vue';
import { useWorkflowsStore } from '@/stores/workflows.store';
import { type ActionDropdownItem, N8nActionDropdown, N8nButton } from '@n8n/design-system';
import { useI18n } from '@n8n/i18n';
import { ref, computed } from 'vue';

defineEmits<{
	mouseenter: [event: MouseEvent];
	mouseleave: [event: MouseEvent];
	click: [event: MouseEvent];
}>();

const props = defineProps<{
	waitingForWebhook?: boolean;
	executing?: boolean;
	disabled?: boolean;
}>();

const i18n = useI18n();

const workflowsStore = useWorkflowsStore();

const triggerNodes = computed(() =>
	workflowsStore.workflowTriggerNodes.filter((node) => !isChatNode(node)),
);

const manuallySelectedTriggerNodeName = ref<string>();

const selectedTriggerNode = computed(
	() => manuallySelectedTriggerNodeName.value ?? triggerNodes.value[0]?.name,
);

const label = computed(() => {
	if (props.executing) {
		return i18n.baseText('nodeView.runButtonText.executingWorkflow');
	}

	if (props.waitingForWebhook) {
		return i18n.baseText('nodeView.runButtonText.waitingForTriggerEvent');
	}

	return i18n.baseText('nodeView.runButtonText.executeWorkflow');
});

const actions = computed(() =>
	triggerNodes.value.map<ActionDropdownItem>((node) => ({
		label: node.name,
		disabled: !!node.disabled,
		id: node.name,
		checked: selectedTriggerNode.value === node.name,
	})),
);

function handleSelectTrigger(nodeName: string) {
	manuallySelectedTriggerNodeName.value = nodeName;
}
</script>

<template>
	<div :class="[$style.component, triggerNodes.length > 1 ? $style.split : '']">
		<KeyboardShortcutTooltip
			:label="label"
			:shortcut="{ metaKey: true, keys: ['↵'] }"
			:disabled="executing"
		>
			<N8nButton
				:class="$style.button"
				:loading="executing"
				:disabled="disabled"
				size="large"
				icon="flask"
				type="primary"
			>
				<span :class="$style.buttonContent">
					{{ label }}
					<span :class="$style.subText">
						<I18nT keypath="nodeView.runButtonText.from">
							<template #nodeName>
								<b>{{ selectedTriggerNode }}</b>
							</template>
						</I18nT>
					</span>
				</span>
			</N8nButton>
		</KeyboardShortcutTooltip>
		<template v-if="triggerNodes.length > 1">
			<div role="presentation" :class="$style.divider" />
			<N8nActionDropdown
				:class="$style.menu"
				:items="actions"
				placement="top"
				@select="handleSelectTrigger"
			>
				<template #activator>
					<N8nButton size="large" :class="$style.chevron" icon="angle-down" />
				</template>
			</N8nActionDropdown>
		</template>
	</div>
</template>

<style lang="scss" module>
.component {
	display: flex;
	align-items: stretch;
}

.split {
	.divider {
		width: var(--border-width-base);
		background-color: var(--color-background-light);
	}

	.button {
		border-top-right-radius: 0;
		border-bottom-right-radius: 0;
		padding-block: var(--spacing-2xs);
	}

	.chevron {
		border-top-left-radius: 0;
		border-bottom-left-radius: 0;
		padding-inline: var(--spacing-2xs);
	}

	.menu :global(.el-dropdown) {
		height: 100%;
	}
}

.buttonContent {
	display: flex;
	flex-direction: column;
	align-items: flex-start !important;
	gap: var(--spacing-3xs);
}

.subText {
	font-size: var(--font-size-3xs);
}
</style>
