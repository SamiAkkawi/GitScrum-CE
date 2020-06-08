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
    }
  },
}
</script>

<template>
  <div>
    <b-button 
    v-if="authorize('tasks', 'create')" 
    v-b-toggle.activities-icon 
    class="btn btn-secondary btn-block"
    :style="(task.type) ? 'color: ' + 
    invertColor(task.type.color, true) + 
    ';background: ' + 
    opacityColor(task.type.color, '0.6') : ''"
    v-text="$t('Activities')"></b-button>
    <b-collapse id="activities-icon" @shown="getActivities">
      <b-card>
        <div v-if="!restricted">
          <b-spinner 
            v-if="loading" 
            :label="$t('Loading')" 
            tag="div" small class="ml-1 mb-1"></b-spinner>
          <div
            v-for="item in activities"
            :key="item.id">
            <div class="box-useravatar">
              <ListUsers :user="item.user" :link="true" size="14"></ListUsers>
              <router-link
              :to="{
                name: 'profile.user',
                params: { username: item.user.username } }" 
                class="txt-primary" v-text="item.user.name" />
            </div>
            <div class="ml-4">
              <span class="small text-dark" v-text="item.message"></span>
              <div class="small text-muted" :title="item.created_at.timezone">
                {{ item.created_at.date_for_humans }}
              </div>
            </div>
          </div>
        </div>
      </b-card>
    </b-collapse>
  </div>
</template>
