<script setup lang="ts">
import { computed } from 'vue';
import { getFieldPrecisionAndScale, useLocaleSettings } from '../shared/utils.js';

//
// Setup and types
//

/**
 * Interface representing the properties for a specific configuration or data structure.
 *
 * @interface Props
 *
 * @property {string} collection - The name of the collection that is being referenced or operated on.
 * @property {string} field - The specific field within the collection to be targeted or utilized.
 * @property {string | null} value -  The value associated with the field. Can be a string or null.
 * @property {{ currency: string } | null} interfaceOptions - Contains optional interface-specific options such as currency settings. Can be null if there are no options provided.
 */
interface Props {
	collection: string;
	field: string;
	value: string | null;
	interfaceOptions: {
		currency: string;
	} | null;
}

//
// Core state and configuration
//

const props = defineProps<Props>();

// Get the configured currency from the interface options.
const configuredCurrency = props.interfaceOptions?.currency ?? 'USD';

// Get the configured scale for proper decimal output.
const [, numericScale] = getFieldPrecisionAndScale(
	props.collection,
	props.field,
);
const scale = numericScale ?? 2;

const { locale } = useLocaleSettings();



/**
 * @computed
 * Formats the input value as a localized currency string.
 *
 * @returns {string} Formatted currency string
 * @description
 * Formatted currency string based on:
 * - Current locale settings
 * - Configured currency (defaults to 'USD')
 * - Numeric scale from field configuration
 * - Returns empty string for null/undefined values
 *
 * @example
 * // With value = 1234.5, USD currency, en-US locale, scale = 2
 * // Returns "$1,234.50"
 */
const formattedValue = computed(() => {
	if (props.value === null || props.value === undefined) return '';

	if (!configuredCurrency) {
		return new Intl.NumberFormat(locale.value, {
			style: 'decimal',
			useGrouping: true,
			minimumFractionDigits: scale,
			maximumFractionDigits: scale,
		}).format(Number(props.value));
	}

	return new Intl.NumberFormat(locale.value, {
		style: 'currency',
		currency: configuredCurrency,
		useGrouping: true,
		minimumFractionDigits: scale,
		maximumFractionDigits: scale,
	}).format(Number(props.value));
});
</script>

<template>
	{{ formattedValue }}
</template>
