<script>
import Axios from '@utils/axios'
import ListUsers from '@components/utils/list-users'

export default {
  components: { ListUsers },
  props: {
    task: {
      type: Object,
      required: true,
    },
    context: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      activities: [],
      loading: true,
      restricted: false,
    }
  },
  methods: {
    getActivities() {
      let endpoint = '?company_slug=' + this.task.company.slug

      if (this.task.project.slug !== '') {
        endpoint += '&project_slug=' + this.task.project.slug
      }

      if (this.task.slug !== '') {
        endpoint += '&slug=' + this.task.slug
      }

      endpoint += '&from_context=' + this.context

      Axios()
        .get('activities/' + endpoint)
        .then((response) => {
          this.loading = false
          this.activities = response.data.data
        })
        .catch((e) => {
          this.loading = false
          this.restricted = true
        })
    },
  },
}
</script>

<template>
  <div>
    <button 
      v-if="authorize('tasks', 'create')" 
      v-b-toggle.activities-icon 
      class="btn btn-secondary btn-block">
      {{ $t('Activities') }}
    </button>
    <b-collapse id="activities-icon" @shown="getActivities">
      <b-card>
        <div v-if="!restricted">
          <div
            v-for="item in activities"
            :key="item.id"
            class="d-flex align-items-center ml-7-px mt-10-px tx-12-px txt-1E1E2F"
          >
            <ListUsers :user="item.user" :link="true" size="30"></ListUsers>
            <div class="ml-10-px">
              <router-link
                :to="{
                  name: 'profile.user',
                  params: { username: item.user.username },
                }"
                class="fw-600 txt-1E1E2F"
                v-text="item.user.name"
              >
              </router-link>
              <span class="ml-5-px fw-500" v-text="item.message"></span>
              <div class="tx-11-px lh-13-px txt-A7AFB7" :title="item.created_at.timezone">
                {{ item.created_at.date_for_humans }}
              </div>
            </div>
          </div>
        </div>
      </b-card>
    </b-collapse>
  </div>
</template>
