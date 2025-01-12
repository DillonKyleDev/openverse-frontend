<template>
  <div
    :class="{ ['filters']: true, ['single']: isSingleFilter }"
    @click="hideLicenseExplanationVisibility()"
    @keyup.enter="hideLicenseExplanationVisibility()"
  >
    <div
      v-if="title"
      class="filters-title"
      @click.prevent="toggleFilterVisibility"
      @keyup.enter="toggleFilterVisibility"
    >
      <h4>{{ title }}</h4>
      <button
        v-if="!filtersExpandedByDefault"
        :aria-label="$t('filter-list.category-aria', { categoryName: title })"
        class="filter-visibility-toggle is-white py-2"
      >
        <i
          v-if="areFiltersExpanded"
          class="icon angle-up rotImg text-lg text-light-gray"
          title="toggle filters visibility"
        />
        <i
          v-else
          class="icon angle-down text-lg text-light-gray"
          title="toggle filters visibility"
        />
      </button>
    </div>
    <template v-if="areFiltersExpanded">
      <div
        v-for="(item, index) in options"
        :key="index"
        class="filter-checkbox-wrapper"
      >
        <!--eslint-disable vuejs-accessibility/label-has-for -->
        <label class="checkbox" :disabled="isDisabled(item)">
          <input
            :key="index"
            :name="item.code"
            type="checkbox"
            class="filter-checkbox mr-2"
            :checked="item.checked"
            :disabled="isDisabled(item)"
            @change="onValueChange"
          />
          <LicenseIcons v-if="filterType === 'licenses'" :license="item.code" />
          {{ itemLabel(item) }}
        </label>
        <img
          v-if="filterType === 'licenses'"
          :aria-label="$t('browse-page.aria.license-explanation')"
          tabindex="0"
          src="@/assets/help_icon.svg"
          alt="help"
          class="license-help pr-1"
          @click.stop="toggleLicenseExplanationVisibility(item.code)"
          @keyup.enter="toggleLicenseExplanationVisibility(item.code)"
        />

        <LicenseExplanationTooltip
          v-if="
            shouldRenderLicenseExplanationTooltip(item.code) &&
            !isDisabled(item)
          "
          :license="licenseExplanationCode"
        />
      </div>
    </template>
  </div>
</template>

<script>
import LicenseIcons from '~/components/LicenseIcons'
import LicenseExplanationTooltip from '~/components/Filters/LicenseExplanationTooltip'

export default {
  name: 'FilterCheckList',
  components: {
    LicenseIcons,
    LicenseExplanationTooltip,
  },
  props: ['options', 'title', 'filterType'],
  data() {
    return {
      filtersVisible: false,
      licenseExplanationVisible: false,
      licenseExplanationCode: '',
    }
  },
  computed: {
    /**
     * Show filters expanded by default
     * @todo: The A/B test is over and we're going with the expanded view. Can remove a lot of this old test logic
     */
    filtersExpandedByDefault() {
      return true
    },
    areFiltersExpanded() {
      return this.filtersExpandedByDefault || this.filtersVisible
    },
    isSingleFilter() {
      return this.$props.filterType === 'searchBy'
    },
  },
  methods: {
    itemLabel(item) {
      return ['audioProviders', 'imageProviders'].includes(
        this.$props.filterType
      )
        ? item.name
        : this.$t(item.name)
    },
    onValueChange(e) {
      this.$emit('filterChanged', {
        code: e.target.name,
        filterType: this.$props.filterType,
      })
    },
    /**
     * This function is used to hide sub items in filters
     * It was decided not to use them after a/b tests
     * TODO: either remove or use if necessary when implementing new designs
     */
    toggleFilterVisibility() {
      this.filtersVisible = !this.filtersVisible
    },
    toggleLicenseExplanationVisibility(licenseCode) {
      this.licenseExplanationVisible = !this.licenseExplanationVisible
      this.licenseExplanationCode = licenseCode
    },
    hideLicenseExplanationVisibility() {
      this.licenseExplanationVisible = false
    },
    isDisabled(e) {
      if (this.$props.filterType === 'licenseTypes') {
        const nc = this.$store.state.filters.licenses.filter((item) =>
          item.code.includes('nc')
        )
        const nd = this.$store.state.filters.licenses.filter((item) =>
          item.code.includes('nd')
        )
        return (
          (e.code === 'commercial' && nc.some((li) => li.checked)) ||
          (e.code === 'modification' && nd.some((li) => li.checked))
        )
      }

      if (this.$props.filterType === 'licenses') {
        const commercial = this.$store.state.filters.licenseTypes.find(
          (item) => item.code === 'commercial'
        )
        const modification = this.$store.state.filters.licenseTypes.find(
          (item) => item.code === 'modification'
        )
        return (
          (commercial.checked && e.code.includes('nc')) ||
          (modification.checked && e.code.includes('nd'))
        )
      }
      return this.$props.disabled
    },
    shouldRenderLicenseExplanationTooltip(licenseCode) {
      return (
        this.licenseExplanationVisible &&
        this.licenseExplanationCode === licenseCode
      )
    },
  },
}
</script>

<style lang="scss" scoped>
.filters {
  border-bottom: 2px solid rgb(245, 245, 245);
  padding: 1.5rem 1rem 1.5rem 1.5rem;
}

.filters-title {
  font-size: 1.25em;
  font-weight: 600;
  font-stretch: normal;
  font-style: normal;
  line-height: 1.5;
  letter-spacing: normal;
  cursor: pointer;
  margin-bottom: 0.5rem;
}

.filter-visibility-toggle {
  float: right;
  cursor: pointer;
  background: none;
  border: none;
}

label {
  color: #333333;
}

.license-help {
  cursor: pointer;
}

.filter-checkbox-wrapper {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 0.5rem;
  padding-bottom: 0.75rem;
}

.checkbox {
  display: flex;
  align-items: center;
  font-size: 0.875rem;
}
.single .checkbox {
  font-size: 1rem;
}
</style>
