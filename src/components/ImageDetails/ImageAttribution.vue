<template>
  <section class="sidebar_section">
    <MediaLicense
      :full-license-name="fullLicenseName"
      :license="image.license"
      :license-u-r-l="licenseURL"
    />
    <CopyLicense
      :media="image"
      :full-license-name="fullLicenseName"
      :license-u-r-l="licenseURL"
      :attribution-html="attributionHtml"
      class="my-6"
    />
  </section>
</template>

<script>
export default {
  name: 'ImageAttribution',
  props: {
    id: {},
    image: {},
    ccLicenseURL: String,
    fullLicenseName: String,
    attributionHtml: String,
  },
  computed: {
    licenseURL() {
      return `${this.ccLicenseURL}&atype=rich`
    },
  },
}
</script>
