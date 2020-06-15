<script>
import MasterHeader from '@layouts/partials/header'
import MasterHeaderMenuProject from '@layouts/partials/header-menu-project'
import { isMobile } from 'mobile-device-detect';

export default {
  components: {
    MasterHeader,
    MasterHeaderMenuProject
  },

  data() {
    return {}
  },
  created() {},
  methods: {},
}
</script>

<template>
  <div class="">
      <MasterHeader>
        <MasterHeaderMenuProject
          v-if="$route.params.projectSlug"
          slot="submenu"
          :background="false"
        ></MasterHeaderMenuProject>
      </MasterHeader>
    
      <div v-if="!isMobile" id="header-project-area" class="tpl-main-project-area subheader">
        <div class="container">
          <div class="d-flex justify-content-between align-items-center ">
            <div>
              <slot name="header-left"></slot>
            </div>
            <div>
              <slot name="header-right"></slot>
            </div>
          </div>
        </div>
      </div>

      <div id="page-content" class="container">
        <slot name="content"></slot>
      </div>

  </div>
</template>
